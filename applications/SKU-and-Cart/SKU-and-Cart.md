# SKU Management & Shopping Cart

**Separate white-labeled application(s)** from Payux billing. Owns **SKU mapping**, **pricing**, and **basket**—not payment rails. The host application owns **products, services, and entitlements**; it does **not** own SKUs.

| Responsibility | Owner |
|----------------|--------|
| **Products and services**; user access via **entitlements** | Host **application** |
| Relate **products / services** (and **purchasable entitlement offers**, i.e. what can be sold—not already-granted rows) → **SKUs** and **prices** | SKU management |
| Collect SKUs, **balance** = sum of prices, **invoice** to billing | Shopping cart |
| **Invoice → payment → receipt** | [Payux](../Payux/Payux.md) (`../../empirestate/BILLING.md`, [DESIGN.md](../Payux/DESIGN.md)) |
| **Grant entitlements** using receipt; **each entitlement backreferences** purchase receipt/billing txn (or gift/grant txn) | Application (cart orchestrates handoff) |

**Flow:** Cart sends **invoice** (amount due + opaque line metadata) to Payux; Payux returns **receipt**; cart completes checkout; application **grants entitlements** tied to that receipt (or gift provenance)—never “SKU” as the long-lived access primitive in the app domain.

**Detailed design:** [applications index](../README.md); [SKU-MANAGEMENT.md](SKU-MANAGEMENT.md), [SHOPPING-CART.md](SHOPPING-CART.md); [Payux design](../Payux/DESIGN.md).
