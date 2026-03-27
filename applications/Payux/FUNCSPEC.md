# Functional specification — Payux

## 1. Purpose and single responsibility

**Payux** is a **white-labeled payment gateway**. In one sentence: it accepts a **payment request that states how much money is owed** (for a **one-time** purchase or as part of a **subscription** lifecycle), collects that money through a **hosted or embedded payment experience**, and produces an authoritative **receipt** each time a charge **successfully settles**.

Payux **does not** sell products, **does not** manage SKUs or prices, **does not** run a shopping cart, and **does not** decide what the user is entitled to after purchase. Those concerns belong to **separate systems** (and to the host application’s domain model). Payux only bridges **money owed → payment captured → proof of payment issued**—including **each** successful charge in a subscription (initial and renewals), without becoming the system of record for **plan definitions**, **commercial policy**, or **entitlement duration**.

### 1.1 Payment processor (pluggable; integrator-visible behavior)

Payux integrates with real card/network rails through a **replaceable payment processor** behind Payux-defined **internal** interfaces (**blackbox** implementations).

- **Integrators** (cart, host, back-office) use **only** Payux’s **public** APIs: invoice, payment session, receipt. Those contracts do **not** name Stripe, PayPal, or any other vendor.
- **M1** ships with **Stripe** as the **first** concrete implementation inside the blackbox. Operators configure credentials and webhook endpoints for that deployment; future processors are added by shipping **another** adapter implementation—**without** changing what integrators call.
- **Payers** may still see a **processor-hosted** payment UI (branding and legal copy can reflect that processor where required by law). Payux wraps or redirects into that experience but remains the **deployment’s** checkout orchestration layer.

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
| **Subscription commercial policy** — plan tiers, feature matrices, proration rules, “fair” upgrade/downgrade, contractual renewal terms | Host and/or SKU/cart stack; Payux stores **opaque** subscription correlation supplied by the integrator and honors **stated amounts** and processor-backed schedules, not product semantics |
| **Subscription lifecycle semantics** — what “canceled” or “paused” means for the customer’s access | Host application (Payux may reflect **payment** state and issue receipts per settled charge only) |
| **Entitlements** — granting, revoking, or interpreting access after purchase | Host application, using the **receipt** Payux returns as proof (including **per renewal** when subscriptions bill multiple times) |

Payux may **store** a blob of **metadata** attached to the invoice (for example a snapshot of line items the cart sends). That is for **audit and handoff** only. Payux **does not** validate that metadata against catalog rules, **does not** recompute totals from it, and **does not** display it as a canonical product breakdown unless the deployer chooses **cosmetic** echoing. The **only** amount Payux treats as authoritative for charging the card or wallet is the **`amount_due`** and **currency** on the invoice (for integrator-initiated sessions), or the **settlement amount** reported by the processor for a **verified recurring charge** (see §5.2)—**never** amounts inferred from metadata alone.

---

## 3. Transaction kinds (one-time and subscription)

Payux supports **both** of the following at the product level. Public integrator APIs remain **processor-agnostic**; subscription mechanics are realized inside the **processor adapter** (blackbox).

### 3.1 One-time payment (OTP)

A single **invoice** for a single checkout: integrator supplies `amount_due` and **currency**; Payux creates a **payment session**; on success, Payux mints **one** **receipt** for that settlement. This is the default “cart checkout” path.

### 3.2 Subscription (initial charge + renewals)

- **Initial subscription checkout:** The integrator still supplies an **invoice** (or equivalent declared charge) for the **first** payable event—setup fee, first period, proration amount, or trial transition—as determined **outside** Payux. The invoice may include an **opaque** `subscription` correlation (integrator-defined id or handle) in **metadata** so later events can be tied together for support and entitlements.
- **Recurring charges:** Subsequent charges are typically **scheduled and executed by the processor** (e.g. saved payment method, processor-native subscription object). Payux learns of each successful settlement via **verified webhooks**, **normalizes** them into the same **receipt** / ledger pattern as OTP, and exposes them to the integrator. **Each successful renewal yields its own receipt** (or ledger mint), suitable for entitlement extension and audit.
- **Integrator obligation:** The host/cart (or billing orchestration *outside* Payux) decides **what** subscription exists in domain terms and **what** to charge for the first invoice. Payux does **not** compute renewal amounts from catalog; it records settlement and issues proof. If the integrator requires Payux to **only** accept renewals that match a previously declared schedule, that **guard** is an implementation contract in `DESIGN.md` (e.g. optional expected amount or integrator webhook)—not a reinterpretation of SKU metadata.

**Out of scope for Payux (subscription):** dunning strategy as a *business* product, tax engine, proration calculator, and “what the subscriber is allowed to do mid-cycle”—unless expressed as **opaque policy** in metadata for display only. **PCI:** cardholder data still stays in the **processor** (hosted checkout or client-side tokenization via processor SDKs); Payux orchestrates **sessions and webhooks**, not raw PAN.

---

## 4. Primary actors

| Actor | Role |
|-------|------|
| **Payer** | Person (or delegated checkout) who completes payment in the processor’s UI or flow initiated by Payux. |
| **Integrator** | Shopping cart, back-office tool, or automated job that **creates** the invoice and **starts** payment; receives **receipt** after each successful settlement (OTP or subscription initial/renewal). |
| **Operator / deployer** | Configures branding, **which processor driver** the deployment uses (M1: Stripe), processor credentials (blackbox), webhook registration, URLs, and tenant-scoped settings. |

The **payer** never needs to know about SKUs inside Payux—they only see **amount**, **currency**, **merchant branding**, and whatever **legally required** payment descriptors the processor shows.

---

## 5. End-to-end flow (functional)

### 5.1 One-time payment (OTP)

1. **Integrator** finalizes “how much to charge” in its own system and builds an **invoice**: stable **invoice id**, **amount due**, **currency**, **payer reference** (opaque), success/cancel targets, optional **metadata** (opaque).
2. **Integrator** calls Payux to **start payment** for that invoice.
3. Payux creates a **payment session** with the processor for **exactly** the invoiced amount (no recomputation from metadata).
4. **Payer** is directed to a **Payux-branded or tenant-branded checkout surface** (hosted page or client-controlled redirect) to pay.
5. On **success**, Payux records settlement, **mints** a **receipt**, and makes it available to the integrator (callback, redirect, and/or polling—per integration contract).
6. On **cancel or hard failure**, Payux leaves **no receipt** for that attempt; integrator returns the payer to **cart or host UX** (not Payux’s job to redraw the cart).
7. **Integrator** hands the **receipt** to the **host application** so entitlements and fulfillment can proceed. Payux does **not** perform provisioning.

### 5.2 Subscription (initial + recurring)

1. **Integrator** defines subscription terms in **its** systems and builds an **invoice** for the **initial** payable event (amount, currency, ids, success/cancel URLs, optional **metadata** including an opaque **subscription** handle).
2. **Integrator** calls Payux to **start payment**; Payux creates a processor session that **may** attach a **recurring** mandate or processor-native subscription (details inside the adapter blackbox).
3. **Payer** completes the **initial** payment in processor-hosted or embedded UI; Payux **mints** a **receipt** for the successful **initial** settlement—same proof semantics as OTP.
4. On **each subsequent successful renewal** (processor-initiated charge), the processor sends **webhooks**; Payux **verifies**, **dedupes**, and **mints** a **receipt** **per successful charge**. The integrator uses **each** receipt to extend or confirm entitlements for that period (or to reconcile finance).
5. **Failed renewals, cancellations, chargebacks:** Payux may surface **payment** outcome via webhooks and/or integrator-facing APIs; **access policy** (grace period, downgrade) remains the **host**’s decision.

---

## 6. User experience — surfaces Payux owns

These are the **product** surfaces Payux is responsible for; everything else (product gallery, cart line items, promo codes, account settings) belongs upstream.

### 6.1 Checkout shell (white-label)

- **Header / logo / colors** per deployment (tenant), so the payer recognizes **who** they are paying.
- **Short legal/merchant identity** text as required for card payments (can mirror processor defaults + deployer copy).
- **Primary display** of **total to pay** (amount + currency) matching the invoice **amount_due**—not a recomputed cart breakdown unless deployer explicitly chooses to **repeat** integrator-supplied display strings (cosmetic only).

### 6.2 Transition states

- **Loading** state while the payment session is created.
- **Redirect or handoff** to the **active processor’s** hosted payment UI (M1: typically Stripe Checkout behind the scenes) — Payux does not replicate full card UX; it orchestrates the handoff. The **integrator** is not required to integrate with that vendor directly.
- **Return** from processor: Payux shows or redirects to **success** or **failure** pages **within its branded shell**, or immediately redirects to integrator-supplied URLs with **stable correlation parameters** (e.g. invoice id) so the host can resume.

### 6.3 Success presentation

- **Confirmation** that payment succeeded.
- **Receipt identifier** (and optionally amount paid, currency, timestamp) suitable for payer peace of mind.
- **Optional** “Return to merchant” control pointing to the integrator **success URL**. Payux does **not** describe *what* was purchased in domain terms unless that text is supplied by the integrator as **opaque display** fields—not interpreted by Payux.

### 6.4 Cancel / failure presentation

- Clear **non-success** state (user canceled, declined, or technical failure).
- **Return to merchant** (cancel URL) without issuing a receipt.

### 6.5 Operator / admin (minimal M1)

- Not required for payer path; if present: **read-only** views of recent sessions/receipts for support—**no** product catalog management.

---

## 7. User experience — surfaces Payux does **not** own

| Surface | Where it lives |
|---------|----------------|
| Browsing catalog, PDPs, plans | Host application |
| Cart drawer, quantity, coupons, tax/shipping **logic** | Shopping cart / host |
| Price labels and “why this total” | SKU + cart stack |
| Account creation, post-purchase “what you unlocked” | Host application |
| Entitlement management UI | Host application |
| Subscription **plan** marketing pages, “compare plans,” feature matrices | Host application |

The **integrator** may embed a “Pay” or “Subscribe” control that **calls Payux**; the **click** still originates from the host or cart UX.

---

## 8. Data semantics (functional)

- **Invoice:** “Charge **this** identity **this** much in **this** currency for correlation id **X**.” For subscriptions, **X** may be the **first** period’s correlation; subsequent charges may correlate via **processor references** and integrator-supplied **opaque subscription** ids carried in **metadata** / ledger snapshots (see `DESIGN.md`).
- **Receipt:** “**This** settlement succeeded; **this** is the durable proof id for finance and downstream provisioning.” For OTP, receipts typically pair **1:1** with the integrator’s **invoice id**. For renewals, **each** settled charge gets its own **receipt** (new id), optionally linked to the same logical subscription via opaque metadata.
- **Metadata:** Integrator-defined **opaque** payload; Payux persists and may echo for display only if configured; **never** drives amount on its own.

---

## 9. Trust and errors (product-level)

- **Never** allow the browser alone to set **amount_due** for integrator-initiated checkouts; the integrator server must submit the invoice Payux uses for the **initial** session. **Renewal** amounts come from **verified processor settlement** (webhooks), not from browser-supplied metadata.
- **Fraud and PCI**: card data stays in the **processor** (whichever adapter is configured); Payux stays in **token/session** orchestration via its **abstracted** processor layer—see `DESIGN.md` §2.5.
- **Idempotency:** Same **invoice id** should not double-charge for the **same** OTP intent; retries should surface a clear **already paid** or **reuse session** experience (exact behavior per `DESIGN.md`). For subscriptions, **processor event** idempotency prevents duplicate receipts for the **same** renewal settlement.

---

## 10. Relationship to other documents

| Document | Role |
|---|------|
| `empirestate/BILLING.md` | Ecosystem and billing philosophy |
| `WORKFLOW.md` (this folder) | Ordered use cases: **Payer**, **MerchantHost**, **CartService**, **Payux**, **Processor** (text sequence, not diagrams) |
| `DESIGN.md` (this folder) | APIs, ledger, webhooks, implementation sequencing |
| `../SKU-and-Cart/SKU-MANAGEMENT.md`, `../SKU-and-Cart/SHOPPING-CART.md` | Everything **before** and **after** the Payux boundary—**not** duplicated here |

---

## 11. One-line summary

**Payux is the checkout and receipt machine for stated debts—one-time or recurring—issuing a proof for every successful settlement; upstream names what is owed and spends each receipt—Payux does neither.**
