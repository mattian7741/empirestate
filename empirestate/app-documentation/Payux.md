# Payux — Unified Billing

The **billing** application: **invoice** (amount due from cart) → **payment** → **receipt** (ledger-backed). Does not understand SKUs, products, or entitlements—only amounts and opaque metadata. The **application** owns products/services and **entitlements** (with **backreference** to receipt or gift transaction); [[SKU-and-Cart]] maps those to SKUs and prices. See [[BILLING]].

**Roadmap:** M1 (highest priority). Drives deployment and delivers universal billing.

**Companion:** [[SKU-and-Cart]] (SKU ↔ product/service/entitlement, prices, cart, invoice).

**Implementation design:** [Design index](../../design/README.md); [PAYUX](../../design/PAYUX.md); companion [SKU management](../../design/SKU-MANAGEMENT.md), [shopping cart](../../design/SHOPPING-CART.md).
