# Design — redirect

**Application implementation handoffs** (Payux, SKU management, shopping cart, and other app `DESIGN.md` files) now live under **`applications/<AppName>/`**.

- **Index:** [applications/README.md](../applications/README.md)
- **Payux:** [applications/Payux/DESIGN.md](../applications/Payux/DESIGN.md) (with [FUNCSPEC.md](../applications/Payux/FUNCSPEC.md), [WORKFLOW.md](../applications/Payux/WORKFLOW.md))
- **SKU & cart stack:** [applications/SKU-and-Cart/SKU-MANAGEMENT.md](../applications/SKU-and-Cart/SKU-MANAGEMENT.md), [applications/SKU-and-Cart/SHOPPING-CART.md](../applications/SKU-and-Cart/SHOPPING-CART.md)

**Agent handoff:** Pair any design doc with [`GUIDE.md`](../GUIDE.md). Obey `empirestate/TENETS.md`.

**Commerce flow (reading order):** `SKU-MANAGEMENT.md` → `SHOPPING-CART.md` → `Payux/FUNCSPEC.md` → `Payux/WORKFLOW.md` → `Payux/DESIGN.md` (paths relative to `applications/`).
