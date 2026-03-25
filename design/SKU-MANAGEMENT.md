# Detailed Design — SKU Management

**Scope:** **SKU management only**—the subsystem that defines **sellable SKUs**, binds **application catalog concepts** to those SKUs, and resolves **prices**. **Not** shopping cart sessions, invoices, payment, or entitlement storage (those belong to `SHOPPING-CART.md` and `PAYUX.md`). **No OpenErgo** required for M1 unless you choose it.

**Handoff:** Implement per this document and `GUIDE.md`. Product truth: `empirestate/BILLING.md`, `empirestate/app-documentation/SKU-and-Cart.md`. Billing contract consumed by cart: `design/PAYUX.md` §2.1 (invoice shape—cart fills `amount_due` using prices from **this** system).

**Deployment note:** May ship as a **service**, a **library** inside a combined “commerce” app, or alongside the cart in one white-labeled product. Boundaries in this doc are **logical**, not prescriptive of repo layout.

---

## 0. Boundaries

| SKU management **does** | SKU management **does not** |
|-------------------------|-----------------------------|
| Maintain **SKU** identifiers and **price** resolution (per currency, tenant, effective window if needed) | Hold **cart** or **checkout** state |
| Map **host-defined** products / services / **purchasable entitlement offers** → SKU + price rules | Process **payments** or issue **receipts** |
| Expose **price resolution** APIs (or events) for cart line items | Grant **entitlements** (host application) |
| Support **static** SKUs and **dynamic** price rules (e.g. quotes, promos—policy in this layer) | Trust arbitrary client-submitted prices without server reconciliation |

**Host application** remains authoritative for **what** a product/service is and **what** an entitlement means. SKU management only answers: “**Which SKU and unit price apply** for this sellable context?”

---

## 1. Domain model (SKU layer)

### 1.1 Core entities (suggested)

- **SKU** — Stable sellable identifier (`sku_id` string or UUID). May map 1:1 to a product variant or to a logical bundle code.
- **Offer binding** — Links host **catalog keys** (e.g. `product_id`, `plan_id`, `entitlement_offer_id`) to a `sku_id`. Cardinality: many catalog shapes can converge on one SKU or diverge to many; document your deployment’s rules.
- **Price book** — `sku_id` + `currency` → `unit_amount_minor` (integer) **or** rule reference (dynamic evaluation). Optional: `tenant_id`, `effective_from` / `effective_to`.
- **Dynamic price rule** (optional) — Named evaluator (e.g. internal promo engine, external quote id). Cart **must** re-resolve or freeze price at checkout per policy.

### 1.2 Purchasable offer vs granted entitlement

- **Purchasable offer** — Something the user can **buy** (maps to SKU + price). Stored or mirrored from host catalog into SKU management.
- **Granted entitlement** — Runtime access record in the **host app** after receipt. SKU management **never** stores granted entitlements.

---

## 2. Price resolution

**Inputs:** `sku_id`, `quantity`, `currency`, optional `tenant_id`, optional context (user tier, promo token, quote id).

**Output:** `unit_price_minor`, `line_total_minor`, optional `price_rule_id` for audit.

**Rules:**

- All **monetary** amounts exposed to cart use **minor units** + **ISO 4217** currency, consistent with Payux invoice fields.
- If the client displays prices, **server-side resolution** at **add-to-cart** and again at **invoice build** is recommended to reduce tampering; cart still sends **authoritative `amount_due`** to Payux.

---

## 3. API surface (illustrative REST)

Version prefix e.g. `/api/v1`. Adjust to your deployment.

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health`, `/version` | Ops. |
| GET | `/api/v1/skus/:sku_id` | SKU metadata (display hints—not Payux’s concern). |
| POST | `/api/v1/prices/resolve` | Body: sku(s), qty, currency, context → priced lines. |
| GET | `/api/v1/offer-bindings` | Admin: list/map catalog keys → `sku_id` (authz gated). |
| PUT | `/api/v1/offer-bindings/...` | Admin: maintain bindings (or use migrations/seed). |

Internal **catalog sync** from host (batch job, queue, or read-through API) is implementation-specific; document **source of truth** per field (host vs SKU DB).

---

## 4. Persistence

- Relational or document store per `STANDARDS.md` / `NO_ORM_POLICY.md`: explicit accessors, migrations.
- Tables (conceptual): `sku`, `offer_binding`, `price_book`, optional `price_rule`, audit log for admin price changes.

---

## 5. White-labeling

Per-tenant branding is optional here (mostly admin/API). **Cart and Payux** carry more user-facing white-label; keep **SKU ids** stable across brands unless you intentionally namespace by tenant.

---

## 6. Security

- Authenticate admin write APIs; rate-limit public `resolve` if exposed.
- Never accept **unit price** from browser as sole authority—recompute or validate server-side before invoice.

---

## 7. Implementation sequencing (suggested)

1. Schema: `sku`, `price_book`, `offer_binding`.
2. `POST /prices/resolve` for single- and multi-line carts.
3. Admin or seed path for bindings and prices.
4. Optional: effective dating, tenant-scoped books, dynamic rules.
5. Integration tests with **SHOPPING-CART** invoice totals.

---

## 8. Reference index

| Topic | Document |
|-------|----------|
| Ecosystem | `empirestate/BILLING.md` |
| App-level stub | `empirestate/app-documentation/SKU-and-Cart.md` |
| Cart & invoice | `design/SHOPPING-CART.md` |
| Billing | `design/PAYUX.md` |
| Design index | `design/README.md` |
| Agent constraints | `GUIDE.md` |
