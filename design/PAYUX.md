# Detailed Design — Payux (Billing Gateway)

**Scope:** **Payux (billing) only**—the service that turns an **invoice** into **payment** and a **receipt**. **Not** SKU catalog, pricing rules, or cart state (separate white-labeled **SKU management + shopping cart** application). **No OpenErgo** in this phase.

**Handoff:** Implement per this document and `GUIDE.md`. Product truth: `empirestate/BILLING.md`, `empirestate/ROADMAP.md` (M1). Engineering primitives: `core-platform-spec/04_SECTION04.md` (§4.1 Billing Product)—interpret **invoice/receipt** as the Payux boundary; catalog **Product Code** primitives in the spec align with the **SKU manager** domain, not with Payux’s business logic.

**Ledger topology:** **Centralized** billing ledger (e.g. Postgres). Ledger pattern: append-only, immutable entries, compensating transactions. Optional **tenant/partition** silos per policy/regulation. See `empirestate/LEDGER.md` for distributed vs centralized by application.

---

## 0. Ecosystem and boundaries

```text
Application — products & services; access via entitlements (not SKUs)
        │
        ▼
SKU management — maps products / services / purchasable offers ──► SKUs + prices   } white-labeled
        │                                                                      } separate app(s)
Shopping cart ── collects SKUs ──► invoice ───────────────────────────────────┘
        │
        │  invoice: amount_due, currency, ids, opaque line-item metadata
        ▼
Payux (billing) ── payment rail ──► receipt (ledger-backed)
        │
        ▼
Cart receives receipt ──► application grants entitlements (each backrefs receipt / billing txn; gifts use grant provenance)
```

| Payux **does** | Payux **does not** |
|----------------|---------------------|
| Validate invoice payload shape (amount, currency, required ids) | Resolve SKU → price |
| Collect payment via **pluggable processor adapter** (M1: Stripe implementation behind interface) | Expose processor-specific concepts on **public** Payux APIs |
| Mint ledger + issue **receipt** id | Grant entitlements or interpret product/service domain |
| Store **metadata** blob verbatim (audit, handoff) | Use metadata for business rules |

**Receipt:** The artifact returned to the cart after successful payment. Internally backed by a **mint** ledger entry (and optional `tickets` / `receipts` row). The host application uses it to **grant entitlements**; each stored entitlement **SHOULD reference** this receipt (or underlying billing transaction id). **Gifts:** same rule using the gift/grant transaction as provenance instead of a purchase receipt.

---

## 1. Goals (M1)

| Goal | Acceptance |
|------|------------|
| Collect money in production | Live payment path completes; ledger mint + receipt issued. |
| Invoice → receipt | Cart (or test client) submits **invoice**; successful payment yields **receipt** returned to caller. |
| Opaque metadata | Invoice may include `metadata` (e.g. line items with `sku_id`, quantities); Payux persists it on the transaction **without validation beyond schema/size**. |
| White-label | Branding, success/cancel URLs, display strings from config. |
| Standalone | REST + client; no OpenErgo. |
| Deployable | Docker, `/health`, `/version`, structured logging. |
| Pluggable processor | Core depends on **adapter interface**; **Stripe** is first blackbox implementation; cart APIs unchanged when adding processors. |

**Non-goals**

- OpenErgo, full entitlements engine, physical multi-tenant isolation (unless required); **SKU manager and cart implementation** (`design/SKU-MANAGEMENT.md`, `design/SHOPPING-CART.md`).

---

## 2. Domain model (Payux-only)

### 2.1 Invoice (input contract)

Submitted by cart (or authorized client). Payux treats as **payment intent**, not as commerce logic.

Suggested fields:

- `invoice_id` (client-supplied UUID, idempotency key for “same checkout”)
- `amount_due` (minor units integer **or** decimal + `currency` ISO 4217)
- `currency`
- `principal_id` or `customer_reference` (opaque)
- `tenant_id` (optional)
- `success_url`, `cancel_url` (or templates resolved server-side)
- `metadata` (JSONB, **opaque**): e.g. `{ "lines": [ { "sku": "…", "qty": 1, "unit_price_minor": … } ] }` — **not validated** by Payux beyond max size / JSON parse

Payux **never** recomputes total from `metadata`; **amount_due** is authoritative for payment.

### 2.2 Ledger entry

Append-only. Same shape as before (mint/refund/…).

- `metadata` or dedicated `invoice_snapshot` stores full invoice + processor ids.
- `processor_kind` (string): e.g. `stripe` — which adapter handled the payment (audit, support).
- `source_reference`: **Processor-native** id (session, payment intent, charge—opaque to Payux domain logic).

### 2.3 Receipt

External name for successful **mint** outcome.

- `receipt_id` (UUID), links to `ledger_entry_id`, `invoice_id`, `principal_id`, `created_at`, `amount_paid`, `currency`, `metadata` echo (optional).
- Returned to cart in API response and/or webhook callback pattern cart registers.

### 2.4 Payment processor abstraction (pluggable blackbox)

Payux **never** leaks a specific processor in its **integrator-facing** contract. All HTTP resources that the **cart or host** calls are **processor-agnostic**. A **first-party adapter** implements Payux-defined interfaces; **Stripe** is the **initial** blackbox implementation (M1). Additional processors ship as **new classes** (or packages) that implement the same interfaces—**no change** to invoice/receipt semantics.

**Interface boundary (conceptual — names illustrative):**

| Responsibility | Method / behavior |
|----------------|-------------------|
| Start payment | `create_checkout_for_invoice(invoice)` → checkout handoff (e.g. redirect URL, client token, expiry) plus **internal** processor correlation id(s). |
| Inbound completion | Accept raw webhook HTTP → **verify** signature/Headers per processor rules → emit a **normalized** `PaymentSettled` (or failure) **internal** event: `invoice_id`, `amount_paid`, `currency`, success flag, processor refs. |
| Idempotency | Adapter cooperates with Payux dedupe (same processor event / same `invoice_id` must not double-mint). |

**Composition:**

- At runtime, Payux loads **one active processor implementation** (config: `PAYMENT_PROCESSOR_DRIVER=stripe` or similar). Core services depend only on the **interface**, not on Stripe types.
- **Stripe** (or any vendor) SDK and secrets live **inside** the adapter implementation—**blackbox** behind the interface, consistent with `NO_ORM_POLICY` “implementation behind the contract.”
- **Tests:** fake / in-memory adapter implementing the same interface for unit tests without network.

**Webhooks:**

- Expose a **single** ingress such as `POST /api/v1/webhooks/payments` that **dispatches** to the registered adapter(s) (e.g. detect provider from headers and delegate). Processors require different verification rules; the dispatcher is thin routing, not business logic.
- M1 may ship with only the Stripe adapter registered; adding **Processor B** = new adapter + register + env for secrets—**no** change to cart-facing APIs.

---

## 3. API surface (REST)

Version prefix: `/api/v1`.

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health` | Liveness/readiness. |
| GET | `/version` | Service name, version, build. |
| POST | `/api/v1/invoices/:invoice_id/pay` | Body: full invoice DTO if not already stored, or reference; creates payment session for **amount_due**. |
| POST | `/api/v1/payments/sessions` | Alternative: single POST with invoice in body → returns checkout session (simpler for MVP). |
| POST | `/api/v1/webhooks/payments` | **Processor webhooks** (not called by integrators). Dispatches to the active adapter(s); verifies signatures; idempotent mint + receipt. Stripe registers this URL in M1; other adapters use the same path with their own verification branch. |
| GET | `/api/v1/receipts/:receipt_id` | Cart fetches receipt status/details after redirect. |
| GET | `/api/v1/receipts` | Query by `invoice_id` or `principal_id` (auth scoped). |

**Deprecated for Payux boundary:** Direct `product_code` → checkout without invoice (catalog belongs to SKU stack).

Errors: canonical envelope + `correlation_id` (`core-platform-spec/contracts/`).

---

## 4. Persistence

- Centralized Postgres; **Invoice** (optional staging), **Receipt**, **LedgerEntry**, **WebhookDedupe** accessors—**no** `ProductCode` table owned by Payux (catalog is external).
- **No ORM** as abstraction; explicit accessors; numbered migrations.

---

## 5. Observability

Structured JSON logs; `correlation_id` on every request; propagate `invoice_id` in log context.

---

## 6. Configuration

- **`PAYMENT_PROCESSOR_DRIVER`** (or equivalent): selects the **registered** adapter implementation (M1: `stripe`).
- **Processor-specific secrets** (e.g. Stripe API key, webhook signing secret): loaded only by that adapter’s factory; **not** referenced from Payux core code paths.
- **Shared:** `PUBLIC_BASE_URL`, `SERVICE_*`, DB URL. No secrets in logs.

---

## 7. Deployment

Docker, non-debug, healthcheck on `/health`. Matches `empirestate/DEPLOYMENT.md`.

---

## 8. Security

Verify webhooks; HTTPS redirects; rate-limit session creation; never trust `metadata` for **amount**—only `amount_due`.

---

## 9. Implementation sequencing (suggested)

1. Define **processor adapter interface** + **fake** adapter for tests.
2. Schema: ledger, receipts, webhook dedupe, `processor_kind` / `source_reference`; optional `invoices` staging table.
3. Implement **Stripe adapter** (blackbox) behind interface; wire via `PAYMENT_PROCESSOR_DRIVER`.
4. `POST` payment session from **invoice** DTO via interface only (ignore SKU meaning in code paths).
5. **`POST /api/v1/webhooks/payments`** dispatcher → adapter verify → mint + receipt; persist metadata blob.
6. `GET` receipt by id / invoice id.
7. White-label URLs and branding hooks.
8. Harden for production; document how to add **Processor B** (new adapter + registration only).

---

## 10. Reference index

| Topic | Document |
|-------|----------|
| **Functional spec (UX, flows, isolation from cart/SKU)** | `funcspec/PAYUX.md` |
| Billing ecosystem | `empirestate/BILLING.md` |
| Cart / SKU companion | `empirestate/app-documentation/SKU-and-Cart.md` |
| SKU management design | `design/SKU-MANAGEMENT.md` |
| Shopping cart design | `design/SHOPPING-CART.md` |
| Design folder index | `design/README.md` |
| M1 | `empirestate/ROADMAP.md` |
| Billing primitives (platform) | `core-platform-spec/04_SECTION04.md` §4.1 |
| Logging / errors | `core-platform-spec/contracts/logging-schema.md`, `error-codes.md` |
| ORM policy | `core-platform-spec/policies/NO_ORM_POLICY.md` |
| Agent constraints | `GUIDE.md` |
