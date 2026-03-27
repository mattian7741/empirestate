# Billing — Universal Billing Gateway

Billing is its own application, **application- and domain-agnostic**. It does **not** interpret catalog semantics (SKUs, product definitions, or entitlements). It converts **money owed** into **payment** and issues a **receipt** (ledger-backed proof) that upstream systems use to complete fulfillment.

Billing implements the **ledger pattern** (see [[LEDGER]]). The billing ledger uses **financial currency**—mint/redeem/refund/transfer against amounts and receipts. The ledger machinery (append-only, immutable, projection-derived) shares common DNA with claims, social, and record-based ledgers across the platform.

**Typical topology:** Billing uses **centralized** data and services—a single authoritative store for the financial ledger (e.g. relational DB), not a distributed peer-to-peer claims ledger. **Partitions or silos** (tenant scope, separate deployments or schemas) are **optional** and introduced when **policy or regulation** requires them. The distributed claims model in [[LEDGER]] applies to other applications; Payux aligns with the centralized billing profile.

---

## Ecosystem (four parts)

Billing is one piece of a **white-labeled commerce flow**. The others are separate applications (or bounded contexts) with clear contracts.

| Layer | Role |
|-------|------|
| **Host application** | Owns **products and services** (what users may access). Access is expressed as **entitlements**—granted from a **receipt** after purchase (or from a gift flow with equivalent provenance). **Every entitlement SHALL carry a backreference** to the billing **transaction / receipt** that paid for it (or to the gift / grant transaction). Does not implement payment rails. **Does not own SKUs**—SKUs are commerce identifiers in the SKU layer. |
| **SKU management** | Relates **products, services, and purchasable entitlement offers** (application-defined) to **SKUs** and **prices** (static or dynamic). White-labeled. Billing does not own price resolution or this mapping. |
| **Shopping cart** | Collects **SKUs** (as priced sellable lines), maintains **balance** = sum of prices, produces an **invoice** (amount due + currency + identity/correlation). White-labeled. Maps SKUs to balance; billing maps balance to payment. |
| **Billing (Payux)** | Accepts an **invoice**. Runs the **transaction** that converts amount due → **payment** (processor) → **receipt** / ticket on the ledger. Returns the receipt to the cart. **No understanding of SKUs, products, or entitlements**—only amounts, currency, and **opaque metadata** (e.g. line-item snapshot) stored as a **record** for audit and handoff. |

**Nutshell**

1. Application defines **products and services**; users get access through **entitlements** (not through SKUs directly).  
2. SKU manager maps those domain concepts to **SKUs** and **prices**.  
3. Cart collects SKUs, totals prices, issues **invoice** to billing.  
4. Billing converts invoice → payment → **receipt**; metadata may include SKU or line payload but billing does not interpret it.  
5. Cart receives receipt; **application grants entitlements** (and any fulfillment), each entitlement **backreferencing** the purchase **receipt / billing transaction** (or gift grant).

**Critical constraint:** The end user never holds payment proof as the entitlement token in the billing domain—the billing backend issues the receipt; the **cart + application** orchestrate entitlement grants using that receipt. **Entitlements without a traceable purchase or gift provenance are out of model** for paid access.

---

## Core model (billing boundary)

| Concept | Meaning |
|---------|---------|
| **Input** | **Invoice** — amount due, currency, payer/correlation identifiers, optional opaque **metadata** (e.g. JSON line items with SKU ids copied from cart). |
| **Exchange** | Payment processor confirms settlement → billing **mints** ledger entry(ies) and issues **receipt**. |
| **Output** | **Receipt** returned to cart (and application) so the app can **grant entitlements** (with mandatory backreference to this transaction/receipt)—billing does not grant or interpret entitlements. |

Upstream systems integrate with **cart + provisioning**, not with payment mechanics inside billing.

---

## Abstraction

- **Billing** exposes: submit invoice / complete payment / retrieve receipt / refunds (later). It must **not** embed SKU catalog rules, entitlement rules, or cart business logic.  
- **SKU management + cart** are white-labeled and own **SKU↔price binding** and **basket** state; the host application remains authoritative for **product/service definitions** (SKU manager maps those into sellable SKUs).  
- **Applications** integrate with entitlements and fulfillment **after** the cart has a valid receipt—not with **payment-processor** keys or raw vendor contracts; they integrate with **Payux**, which abstracts processors behind **pluggable adapter** implementations (M1: Stripe in the blackbox—see `applications/Payux/DESIGN.md` §2.5, `applications/Payux/FUNCSPEC.md` §1.1). **Model rule:** paid (or gift-substituted) entitlements always reference the **billing receipt / transaction id** (or equivalent gift record) for audit and support.

---

## Properties

| Property | Description |
|----------|-------------|
| **White-label** | Billing UI/branding configurable per deployment (alongside cart and SKU manager). |
| **Domain-agnostic** | Invoice in, receipt out; metadata is storage-only from billing’s perspective. |

---

## Implementation order

Payux is **highest priority** on the roadmap—it drives deployment and universal **payment + receipt** capability. It starts **standalone** with traditional backend and client; OpenErgo integration follows M3. See [[ROADMAP]]. Companion design for **SKU management + shopping cart** is tracked separately (see `applications/SKU-and-Cart/SKU-and-Cart.md`).
