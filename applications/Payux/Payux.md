# Payux — Unified Billing

The **billing** application: **invoice** (amount due from cart) → **payment** → **receipt** (ledger-backed). Does not understand SKUs, products, or entitlements—only amounts and opaque metadata. The **application** owns products/services and **entitlements** (with **backreference** to receipt or gift transaction); [SKU and cart](../SKU-and-Cart/SKU-and-Cart.md) maps those to SKUs and prices. See [BILLING](../../empirestate/BILLING.md).

**Roadmap:** M1 (highest priority). Drives deployment and delivers universal billing.

**Companion:** [SKU and cart](../SKU-and-Cart/SKU-and-Cart.md) (SKU ↔ product/service/entitlement, prices, cart, invoice).

**Functional specification (behavior & UX, gateway isolated from cart/SKU/product):** [FUNCSPEC.md](FUNCSPEC.md).

**Workflow (text sequence — who acts in order, OTP / subscription / cancel):** [WORKFLOW.md](WORKFLOW.md).

**Implementation design:** [DESIGN.md](DESIGN.md); companion [SKU management](../SKU-and-Cart/SKU-MANAGEMENT.md), [shopping cart](../SKU-and-Cart/SHOPPING-CART.md). Application index: [applications/README.md](../README.md).
