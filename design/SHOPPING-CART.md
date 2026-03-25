# Detailed Design — Shopping Cart

**Scope:** **Shopping cart and checkout orchestration**—persists **basket** state, computes **balance**, builds the **invoice** for Payux, handles **return from payment**, and **hands off** to the host application to **grant entitlements** with mandatory **purchase provenance**. **Not** payment capture or ledger mint (Payux). **Not** authoritative product definitions (host) or long-term price book admin (SKU management)—but cart **calls** SKU management for **resolved prices**.

**Handoff:** Implement per this document and `GUIDE.md`. Product truth: `empirestate/BILLING.md`, `empirestate/app-documentation/SKU-and-Cart.md`. **Invoice/receipt contracts:** `design/PAYUX.md`. **Prices:** `design/SKU-MANAGEMENT.md`.

**Deployment note:** Often combined with SKU management in one **white-labeled** commerce service; this doc describes the **cart** bounded context only.

---

## 0. Boundaries

| Cart **does** | Cart **does not** |
|---------------|-------------------|
| Store **cart session**, **lines** (`sku_id`, qty, optional frozen unit price) | Mint **receipts** or touch the **billing ledger** |
| Compute **balance** and build **invoice** (`amount_due` + metadata) | Interpret payment processor webhooks (Payux) |
| Call Payux to **start payment**; **poll or subscribe** for completion + **receipt id** | Store **granted entitlements** as system of record (host app does) |
| Invoke host **post-checkout hook** (or enqueue job) with **receipt** + line snapshot | Decide entitlement **business rules** (host) |

---

## 1. End-to-end flow

1. User adds lines; cart resolves **SKU + price** via **SKU management** (or uses server-frozen snapshot on line).
2. Cart maintains **balance** = sum of line totals (+ tax/shipping if you add them—then include in `amount_due` and metadata consistently).
3. Cart creates **`invoice_id`** (UUID) as idempotency key for checkout.
4. Cart calls **Payux** with invoice DTO (`amount_due` **must** equal server-computed checkout total).
5. User pays at processor; Payux issues **receipt**.
6. Cart obtains **receipt** (redirect + fetch, or async notification from Payux if implemented).
7. Cart **completes** checkout transaction and calls **host application** to **grant entitlements**; each grant records **provenance** = receipt / billing transaction (see §5).

---

## 2. Domain model (cart)

### 2.1 Cart session

- `cart_id`, `principal_id` (or anonymous token with later merge), `tenant_id` optional.
- Status: `open` → `checkout_pending` → `completed` | `abandoned` | `expired`.
- TTL / cleanup policy: implementation-specific.

### 2.2 Cart line

- `sku_id`, `quantity`, `currency`
- `unit_price_minor` **snapshot** at line add or at checkout (policy); must align with **SKU management** resolution.
- Optional: display labels copied from SKU service for UX (not sent to Payux as authority).

### 2.3 Checkout session (optional table)

- Links `cart_id` ↔ `invoice_id` ↔ Payux session id ↔ eventual `receipt_id`.
- Supports idempotent “retry checkout” for same `invoice_id` per Payux contract.

---

## 3. Invoice assembly (Payux input)

Align field names and types with **`design/PAYUX.md` §2.1**.

**Invariant:** `amount_due` = **server-side** sum of priced lines (plus tax/shipping if modeled). **Never** trust browser-only totals.

**Metadata (opaque to Payux, useful for host):** e.g. `{ "lines": [ { "sku_id", "qty", "unit_price_minor", "line_total_minor", "catalog_ref": "..." } ], "cart_id" }`. Payux persists verbatim; host uses for entitlement mapping.

---

## 4. Payux integration

- **Create payment session:** e.g. `POST /api/v1/payments/sessions` (see Payux doc) with full invoice body or stored invoice reference.
- **Success / cancel URLs:** cart or Payux resolves; include `invoice_id` in query/state for correlation.
- **Receipt retrieval:** `GET /api/v1/receipts/:receipt_id` or by `invoice_id` as Payux exposes—cart verifies **paid** before calling host.

**Failure / timeout:** Cart marks checkout failed; inventory or holds (if any) released per your policy.

---

## 5. Host handoff — entitlements and provenance

After **confirmed receipt**:

1. Cart (or **checkout worker**) sends to host: `receipt_id`, Payux `invoice_id`, `principal_id`, **metadata** echo or line snapshot.
2. Host **grants entitlements** for each purchased offer; each record stores:
   - **provenance_type:** e.g. `purchase` | `gift` | `admin` | `promo` (only `purchase` uses billing receipt in the happy path),
   - **billing_receipt_id** / **billing_transaction_id** (for `purchase`),
   - optional **gift_transaction_id** for gifts,
   - SKU/line snapshot as needed for support.

**Rule:** Paid access from this flow **must** trace to the **billing receipt** (or explicit non-purchase provenance for free/admin paths). See `empirestate/BILLING.md`.

---

## 6. API surface (illustrative REST)

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health`, `/version` | Ops. |
| POST | `/api/v1/carts` | Create cart. |
| POST | `/api/v1/carts/:id/lines` | Add/update line (server resolves price via SKU management). |
| GET | `/api/v1/carts/:id` | Cart + computed totals. |
| POST | `/api/v1/carts/:id/checkout` | Build invoice, call Payux, return payment redirect URL or client secret. |
| GET | `/api/v1/checkouts/:invoice_id` | Status: pending / paid / failed + `receipt_id` when available. |

Auth: bind cart to **principal** or session; enforce **tenant** scope.

---

## 7. Persistence

- Cart + lines + checkout session tables; idempotency on `invoice_id` when calling Payux.
- **No ORM** as abstraction per platform policy; explicit accessors.

---

## 8. Observability

Structured logs; propagate `cart_id`, `invoice_id`, `correlation_id` through Payux calls and host webhook/job.

---

## 9. Security

- Server-authoritative **totals**; validate qty limits and SKU allowlists.
- Do not expose **raw** processor secrets to browser except via Payux/client patterns Payux defines.

---

## 10. Implementation sequencing (suggested)

1. Cart CRUD + line add with **SKU-MANAGEMENT** `resolve` integration.
2. Checkout: build invoice, call Payux session create.
3. Receipt polling + **completed** transition.
4. Host **entitlement grant** integration (sync API or queue).
5. Abandonment, expiry, and idempotent retry paths.

---

## 11. Reference index

| Topic | Document |
|-------|----------|
| Ecosystem | `empirestate/BILLING.md` |
| App-level stub | `empirestate/app-documentation/SKU-and-Cart.md` |
| SKU & prices | `design/SKU-MANAGEMENT.md` |
| Billing | `design/PAYUX.md` |
| Design index | `design/README.md` |
| Agent constraints | `GUIDE.md` |
