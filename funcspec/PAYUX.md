# Functional specification — Payux

## 1. Purpose and single responsibility

**Payux** is a **white-labeled payment gateway**. In one sentence: it accepts a **payment request that states how much money is owed**, collects that money through a **hosted or embedded payment experience**, and produces an authoritative **receipt** when payment succeeds.

Payux **does not** sell products, **does not** manage SKUs or prices, **does not** run a shopping cart, and **does not** decide what the user is entitled to after purchase. Those concerns belong to **separate systems** (and to the host application’s domain model). Payux only bridges **money owed → payment captured → proof of payment issued**.

---

## 2. What is explicitly out of scope (neighbor systems)

To avoid leakage of responsibility, the following live **outside** Payux:

| Concern | Owner (not Payux) |
|---------|-------------------|
| **Products and services** — what exists for sale, described in domain language | Host application |
| **SKUs** — commerce identifiers, bundles, variants | SKU management application |
| **Prices** — list price, discounts, dynamic quotes, tax display rules | SKU management (and/or host policy); the **cart** computes totals |
| **Shopping cart** — add/remove lines, session, checkout UX before payment | Shopping cart application |
| **Invoice composition** — line items, subtotals, `amount_due` that matches business rules | Shopping cart (or authorized client) prepares the invoice and calls Payux |
| **Entitlements** — granting, revoking, or interpreting access after purchase | Host application, using the **receipt** Payux returns as proof |

Payux may **store** a blob of **metadata** attached to the invoice (for example a snapshot of line items the cart sends). That is for **audit and handoff** only. Payux **does not** validate that metadata against catalog rules, **does not** recompute totals from it, and **does not** display it as a canonical product breakdown unless the deployer chooses **cosmetic** echoing. The **only** amount Payux treats as authoritative for charging the card or wallet is the **`amount_due`** and **currency** on the invoice.

---

## 3. Primary actors

| Actor | Role |
|-------|------|
| **Payer** | Person (or delegated checkout) who completes payment in the processor’s UI or flow initiated by Payux. |
| **Integrator** | Shopping cart, back-office tool, or automated job that **creates** the invoice and **starts** payment; receives **receipt** after success. |
| **Operator / deployer** | Configures branding, processor keys, URLs, and tenant-scoped settings. |

The **payer** never needs to know about SKUs inside Payux—they only see **amount**, **currency**, **merchant branding**, and whatever **legally required** payment descriptors the processor shows.

---

## 4. End-to-end flow (functional)

1. **Integrator** finalizes “how much to charge” in its own system and builds an **invoice**: stable **invoice id**, **amount due**, **currency**, **payer reference** (opaque), success/cancel targets, optional **metadata** (opaque).
2. **Integrator** calls Payux to **start payment** for that invoice.
3. Payux creates a **payment session** with the processor for **exactly** the invoiced amount (no recomputation from metadata).
4. **Payer** is directed to a **Payux-branded or tenant-branded checkout surface** (hosted page or client-controlled redirect) to pay.
5. On **success**, Payux records settlement, **mints** a **receipt**, and makes it available to the integrator (callback, redirect, and/or polling—per integration contract).
6. On **cancel or hard failure**, Payux leaves **no receipt** for that attempt; integrator returns the payer to **cart or host UX** (not Payux’s job to redraw the cart).
7. **Integrator** hands the **receipt** to the **host application** so entitlements and fulfillment can proceed. Payux does **not** perform provisioning.

---

## 5. User experience — surfaces Payux owns

These are the **product** surfaces Payux is responsible for; everything else (product gallery, cart line items, promo codes, account settings) belongs upstream.

### 5.1 Checkout shell (white-label)

- **Header / logo / colors** per deployment (tenant), so the payer recognizes **who** they are paying.
- **Short legal/merchant identity** text as required for card payments (can mirror processor defaults + deployer copy).
- **Primary display** of **total to pay** (amount + currency) matching the invoice **amount_due**—not a recomputed cart breakdown unless deployer explicitly chooses to **repeat** integrator-supplied display strings (cosmetic only).

### 5.2 Transition states

- **Loading** state while the payment session is created.
- **Redirect or handoff** to the **payment processor**’s UI (Stripe Checkout, etc.) — Payux does not replicate full card UX; it orchestrates the handoff.
- **Return** from processor: Payux shows or redirects to **success** or **failure** pages **within its branded shell**, or immediately redirects to integrator-supplied URLs with **stable correlation parameters** (e.g. invoice id) so the host can resume.

### 5.3 Success presentation

- **Confirmation** that payment succeeded.
- **Receipt identifier** (and optionally amount paid, currency, timestamp) suitable for payer peace of mind.
- **Optional** “Return to merchant” control pointing to the integrator **success URL**. Payux does **not** describe *what* was purchased in domain terms unless that text is supplied by the integrator as **opaque display** fields—not interpreted by Payux.

### 5.4 Cancel / failure presentation

- Clear **non-success** state (user canceled, declined, or technical failure).
- **Return to merchant** (cancel URL) without issuing a receipt.

### 5.5 Operator / admin (minimal M1)

- Not required for payer path; if present: **read-only** views of recent sessions/receipts for support—**no** product catalog management.

---

## 6. User experience — surfaces Payux does **not** own

| Surface | Where it lives |
|---------|----------------|
| Browsing catalog, PDPs, plans | Host application |
| Cart drawer, quantity, coupons, tax/shipping **logic** | Shopping cart / host |
| Price labels and “why this total” | SKU + cart stack |
| Account creation, post-purchase “what you unlocked” | Host application |
| Entitlement management UI | Host application |

The **integrator** may embed a “Pay” button that **calls Payux**; the **click** still originates from the cart’s UX.

---

## 7. Data semantics (functional)

- **Invoice:** “Charge **this** identity **this** much in **this** currency for correlation id **X**.”
- **Receipt:** “Payment for invoice **X** succeeded; **this** is the durable proof id for finance and downstream provisioning.”
- **Metadata:** Integrator-defined **opaque** payload; Payux persists and may echo for display only if configured; **never** drives amount.

---

## 8. Trust and errors (product-level)

- **Never** allow the browser alone to set **amount_due**; integrator server must submit the invoice Payux charges against.
- **Fraud and PCI**: card data stays in the **processor**; Payux stays in **token/session** orchestration—consistent with detailed design.
- **Idempotency:** Same **invoice id** should not double-charge; retries should surface a clear **already paid** or **reuse session** experience (exact behavior per `design/PAYUX.md`).

---

## 9. Relationship to other documents

| Document | Role |
|---|------|
| `empirestate/BILLING.md` | Ecosystem and billing philosophy |
| `design/PAYUX.md` | APIs, ledger, webhooks, implementation sequencing |
| `design/SKU-MANAGEMENT.md`, `design/SHOPPING-CART.md` | Everything **before** and **after** the Payux boundary—**not** duplicated here |

---

## 10. One-line summary

**Payux is the checkout and receipt machine for a stated debt; the cart names the debt, and the host application spends the receipt—Payux does neither.**
