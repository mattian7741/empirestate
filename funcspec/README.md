# Functional specifications

**Purpose:** Describe **how each application behaves** from a product perspective—user flows, screens, copy expectations, and boundaries—in **plain English**. These docs complement **`design/`** (engineering handoffs: APIs, persistence, sequencing).

| Document | Application |
|----------|-------------|
| [PAYUX.md](PAYUX.md) | Payux — payment gateway only (invoice → pay → receipt) |

**Boundary:** Functional specs state **what Payux is responsible for**. SKU management, cart, catalog, and entitlement provisioning are **other** applications; they are referenced only as **upstream/downstream neighbors**, not specified here.

**Product truth:** `empirestate/BILLING.md` • **Detailed design:** `../design/PAYUX.md`
