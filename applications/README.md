# Applications

Applications that build on or consume EmpireStack. Each named application has a **folder** under `applications/` containing its stub or overview document (filename matches the app, e.g. `Payux/Payux.md`) plus, when they exist, **`FUNCSPEC.md`**, optional **`WORKFLOW.md`** (numbered text sequence / use-case order; not a diagram language), and **`DESIGN.md`**.

Details scraped from Projects and venture documentation. Not all projects in that source set are EmpireStack applications; some listed here have no dedicated specs yet.

## Applications

| Application | Documents |
|-------------|-----------|
| **Payux** | [Overview](Payux/Payux.md) — Billing: invoice → payment → receipt. M1 roadmap. [FUNCSPEC](Payux/FUNCSPEC.md) · [WORKFLOW](Payux/WORKFLOW.md) · [DESIGN](Payux/DESIGN.md) |
| **SKU management & cart** | [Overview](SKU-and-Cart/SKU-and-Cart.md) — Maps products/services/entitlements to SKUs + prices; basket; invoice to billing; white-labeled. [SKU-MANAGEMENT](SKU-and-Cart/SKU-MANAGEMENT.md) · [SHOPPING-CART](SKU-and-Cart/SHOPPING-CART.md) |
| **WishingWell** | [WishingWell](WishingWell/WishingWell.md) |
| **CircuitLeagues** | [CircuitLeagues](CircuitLeagues/CircuitLeagues.md) |
| **KidCritical** | [KidCritical](KidCritical/KidCritical.md) |
| **HotPotato** | [HotPotato](HotPotato/HotPotato.md) |
| **Plunder** | [Plunder](Plunder/Plunder.md) |
| **Message-in-a-Bottle** | [Message-in-a-Bottle](Message-in-a-Bottle/Message-in-a-Bottle.md) |
| **Budge-It** | [Budge-It](Budge-It/Budge-It.md) |
| **Emma** | [Emma](Emma/Emma.md) |
| **Venus** | [Venus](Venus/Venus.md) |
| **Undiet** | [Undiet](Undiet/Undiet.md) |
| **Encryption-in-plain-Sight** | [Encryption-in-plain-Sight](Encryption-in-plain-Sight/Encryption-in-plain-Sight.md) |
| **Plug-it** | [Plug-it](Plug-it/Plug-it.md) |
| **FileTriage** | [FileTriage](FileTriage/FileTriage.md) |
| **Busibooks** | [Busibooks](Busibooks/Busibooks.md) |
| **Cruisematch** | [Cruisematch](Cruisematch/Cruisematch.md) |
| **StockWatch** | [StockWatch](StockWatch/StockWatch.md) |
| **WebMetaSocial** | [WebMetaSocial](WebMetaSocial/WebMetaSocial.md) |
| **BrowserTile** | [BrowserTile](BrowserTile/BrowserTile.md) |

## Intersections

| Applications | Relationship |
|--------------|--------------|
| **HotPotato ↔ Emma** | Natural intersection in future implementations. HotPotato (task orchestration, tactical surfacing, delegation) and Emma (agentic orchestration, Zoe-driven task extraction to human executor) align on execution flow—Emma's tactical output could feed HotPotato's task layer; HotPotato could surface next actions for Emma's human counterpart. |
