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
Shopping cart (or host billing) ── collects SKUs / plan choice ──► invoice ───┘
        │
        │  invoice: amount_due, currency, ids, opaque line-item + optional subscription metadata
        ▼
Payux (billing) ── payment rail ──► receipt (ledger-backed), per successful settlement
        │            (OTP: one checkout; subscription: initial + each renewal webhook)
        ▼
Integrator receives receipt(s) ──► application grants/extends entitlements (each backrefs receipt / billing txn; gifts use grant provenance)
```

| Payux **does** | Payux **does not** |
|----------------|---------------------|
| Validate invoice payload shape (amount, currency, required ids) | Resolve SKU → price |
| Collect payment via **pluggable processor adapter** (M1: Stripe implementation behind interface) for **OTP** and **subscription** (initial session + recurring webhooks) | Expose processor-specific concepts on **public** Payux APIs |
| Mint ledger + issue **receipt** id **per successful charge** (including **each** subscription renewal settlement) | Grant entitlements or interpret product/service domain |
| Store **metadata** blob verbatim (audit, handoff) | Use metadata for business rules |
| Surface **payment** outcomes for renewals (via webhooks / optional APIs) | Own subscription **plan catalog**, proration policy, or “what canceled means” for product access |

**Receipt:** The artifact returned to the integrator after **each** successful payment. For **one-time (OTP)** checkouts this is usually paired with the integrator’s **invoice id**. For **subscriptions**, **each** renewal settlement produces a **new** receipt id; logical grouping uses **opaque** subscription correlation in **metadata** / ledger snapshot (not SKU interpretation). Internally backed by a **mint** ledger entry (and optional `tickets` / `receipts` row). The host application uses it to **grant entitlements**; each stored entitlement **SHOULD reference** this receipt (or underlying billing transaction id). **Gifts:** same rule using the gift/grant transaction as provenance instead of a purchase receipt.

---

## 1. Goals (M1)

| Goal | Acceptance |
|------|------------|
| Collect money in production | Live payment path completes; ledger mint + receipt issued. |
| Invoice → receipt (OTP) | Cart (or test client) submits **invoice**; successful payment yields **receipt** returned to caller. |
| Subscription initial → receipt | Integrator submits **invoice** for first payable period (or setup); successful payment yields **receipt**; processor may hold recurring mandate (adapter blackbox). |
| Subscription renewals → receipts | Processor webhooks for successful renewal charges produce **additional** receipts (idempotent per processor event / dedupe key). |
| Opaque metadata | Invoice may include `metadata` (e.g. line items with `sku_id`, quantities, **subscription** handle); Payux persists it on the transaction **without validation beyond schema/size**. |
| White-label | Branding, success/cancel URLs, display strings from config. |
| Standalone | REST + client; no OpenErgo. |
| Deployable | Docker, `/health`, `/version`, structured logging. |
| Pluggable processor | Core depends on **adapter interface**; **Stripe** is first blackbox implementation; cart APIs unchanged when adding processors. |

**Non-goals**

- OpenErgo, full entitlements engine, physical multi-tenant isolation (unless required); **SKU manager and cart implementation** (`../SKU-and-Cart/SKU-MANAGEMENT.md`, `../SKU-and-Cart/SHOPPING-CART.md`).
- **Subscription management product** inside Payux (plan merchandising, proration engine, dunning *policy* as business rules)—Payux records **settlements** and issues **receipts**; commercial policy stays upstream unless expressed as **opaque** metadata for audit/display.

---

## 2. Domain model (Payux-only)

### 2.1 Invoice (input contract)

Submitted by cart (or authorized client). Payux treats as **payment intent**, not as commerce logic. Applies to **OTP** and to the **initial** subscription checkout; **renewals** are normally **not** preceded by a new invoice POST (see §2.4).

Suggested fields:

- `invoice_id` (client-supplied UUID, idempotency key for “same checkout” / initial subscription session)
- `billing_mode` (optional enum): `one_time` | `subscription_initial` — informs the **adapter** whether to create a processor session that establishes recurring billing; **not** exposed as processor vendor names on public HTTP docs
- `amount_due` (minor units integer **or** decimal + `currency` ISO 4217)
- `currency`
- `principal_id` or `customer_reference` (opaque)
- `tenant_id` (optional)
- `success_url`, `cancel_url` (or templates resolved server-side)
- `metadata` (JSONB, **opaque**): e.g. `{ "lines": [ … ], "subscription": { "external_id": "…" } }` — **not validated** by Payux beyond max size / JSON parse

Payux **never** recomputes total from `metadata`; **amount_due` is authoritative for the **integrator-initiated** session. **Renewal** charges use **settlement amounts** from verified processor events (see §2.4).

### 2.2 Ledger entry

Append-only. Same shape as before (mint/refund/…).

- `metadata` or dedicated `invoice_snapshot` stores full invoice + processor ids.
- `processor_kind` (string): e.g. `stripe` — which adapter handled the payment (audit, support).
- `source_reference`: **Processor-native** id (session, payment intent, charge—opaque to Payux domain logic).

### 2.3 Receipt

External name for successful **mint** outcome (**one receipt per successful settlement**).

- `receipt_id` (UUID), links to `ledger_entry_id`, optional `invoice_id` (set for OTP and **initial** subscription charge; may be null or synthetic for **renewal-only** settlements if no integrator invoice exists—policy choice: either require integrator-issued **renewal invoice** ids via adapter config, or stamp `invoice_id` from first checkout and use `billing_period` / processor refs in metadata)
- `principal_id`, `created_at`, `amount_paid`, `currency`, `metadata` echo (optional), processor references for support
- Returned to integrator in API response, outbound integrator webhook, and/or polling—**including** subscription renewals

### 2.4 Subscription renewals and settlement (Payux view)

- **Who schedules the charge:** The **processor** (subscription object, saved payment method) after the initial Payux-orchestrated session—**inside the adapter blackbox**.
- **How Payux learns:** Inbound **webhooks** (same ingress as OTP) verified by the adapter → normalized **PaymentSettled** (or equivalent) with **settlement amount**, **currency**, **success/failure**, and **dedupe** key.
- **Receipt rule:** **Each** successful renewal **mint** gets a **new** `receipt_id`; correlate periods via **opaque** integrator subscription id echoed from the **initial** invoice metadata snapshot and/or **processor** references stored on `LedgerEntry`.
- **Amount authority:** For renewals, **authoritative amount** is the **verified settlement** from the processor, not metadata from the host. Optional future **guard:** compare settlement to integrator-registered **expected** amounts (out of scope for M1 unless explicitly added).
- **Failures:** Failed renewal webhooks **do not** mint receipts; host decides grace/downgrade using application rules (Payux may expose **notifications** only).

### 2.5 Payment processor abstraction (pluggable blackbox)

Payux **never** leaks a specific processor in its **integrator-facing** contract. All HTTP resources that the **cart or host** calls are **processor-agnostic**. A **first-party adapter** implements Payux-defined interfaces; **Stripe** is the **initial** blackbox implementation (M1). Additional processors ship as **new classes** (or packages) that implement the same interfaces—**no change** to invoice/receipt semantics.

**Interface boundary (conceptual — names illustrative):**

| Responsibility | Method / behavior |
|----------------|-------------------|
| Start payment | `create_checkout_for_invoice(invoice)` → checkout handoff (e.g. redirect URL, client token, expiry) plus **internal** processor correlation id(s). For `billing_mode=subscription_initial`, implementation **may** create processor-native recurring resources; **still** no vendor types on public Payux APIs. |
| Inbound completion | Accept raw webhook HTTP → **verify** signature/Headers per processor rules → emit a **normalized** `PaymentSettled` (or failure) **internal** event: settlement `amount`, `currency`, success flag, processor refs, optional `invoice_id` correlation when present, **dedupe** key. Must handle **OTP** completion events **and** **subscription renewal** (and failure) events. |
| Idempotency | Adapter cooperates with Payux dedupe (same processor event must not double-mint; same `invoice_id` must not double-charge the **same** OTP intent). |

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
| GET | `/api/v1/receipts/:receipt_id` | Integrator fetches receipt details after redirect or async notification. |
| GET | `/api/v1/receipts` | Query by `invoice_id`, `principal_id`, and/or **opaque subscription key** in metadata (auth scoped). Exact filters evolve with persistence projections. |

**OTP vs subscription:** Same **session** and **webhook** surfaces; `billing_mode` (or equivalent) on **invoice** steers the adapter. **Renewal** traffic is **webhook-first**; optional future `GET` for “subscription payment state” stays **processor-agnostic** (no Stripe types in JSON).

**Deprecated for Payux boundary:** Direct `product_code` → checkout without invoice (catalog belongs to SKU stack).

Errors: canonical envelope + `correlation_id` (`core-platform-spec/contracts/`).

---

## 4. Persistence

- Centralized Postgres; **Invoice** (optional staging), **Receipt**, **LedgerEntry**, **WebhookDedupe** accessors—**no** `ProductCode` table owned by Payux (catalog is external). Receipt and ledger rows for subscriptions may duplicate **metadata** snapshots for **correlation** across renewals (still **opaque** to Payux logic).
- **No ORM** as abstraction; explicit accessors; numbered migrations.

---

## 5. Observability

Structured JSON logs; `correlation_id` on every request; propagate `invoice_id` and **opaque subscription** correlation (when present) in log context—**no** processor secret payloads.

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

Verify webhooks; HTTPS redirects; rate-limit session creation; never trust `metadata` for **amount**—only `amount_due` on integrator-initiated invoices and **verified** settlement on renewals.

---

## 9. Implementation sequencing (suggested)

1. Define **processor adapter interface** + **fake** adapter for tests.
2. Schema: ledger, receipts, webhook dedupe, `processor_kind` / `source_reference`; optional `invoices` staging table.
3. Implement **Stripe adapter** (blackbox) behind interface; wire via `PAYMENT_PROCESSOR_DRIVER`.
4. `POST` payment session from **invoice** DTO via interface only (ignore SKU meaning in code paths); support **`billing_mode`** for subscription **initial** path in adapter.
5. **`POST /api/v1/webhooks/payments`** dispatcher → adapter verify → mint + receipt; persist metadata blob; handle **renewal** events with **dedupe** and **new** receipt per successful charge.
6. `GET` receipt by id / invoice id / subscription-scoped queries (as projections allow).
7. White-label URLs and branding hooks.
8. Harden for production; document how to add **Processor B** (new adapter + registration only).

---

## 10. Reference index

| Topic | Document |
|-------|----------|
| **Functional spec (UX, flows, isolation from cart/SKU)** | `FUNCSPEC.md` (this folder) |
| **Workflow (text sequence, participants)** | `WORKFLOW.md` (this folder) |
| Billing ecosystem | `empirestate/BILLING.md` |
| Cart / SKU companion | `../SKU-and-Cart/SKU-and-Cart.md` |
| SKU management design | `../SKU-and-Cart/SKU-MANAGEMENT.md` |
| Shopping cart design | `../SKU-and-Cart/SHOPPING-CART.md` |
| Applications index | `../README.md` |
| M1 | `empirestate/ROADMAP.md` |
| Billing primitives (platform) | `core-platform-spec/04_SECTION04.md` §4.1 |
| Logging / errors | `core-platform-spec/contracts/logging-schema.md`, `error-codes.md` |
| ORM policy | `core-platform-spec/policies/NO_ORM_POLICY.md` |
| Agent constraints | `GUIDE.md` |
