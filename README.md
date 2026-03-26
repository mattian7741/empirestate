# EmpireStack Documentation

This folder contains documentation for the EmpireStack project. Primary libraries below; **concise/** is a dense platform-only slice (no consuming applications).

**How to use this documentation:** [GUIDE.md](GUIDE.md) — Canonical navigation: philosophy (`TENETS.md`), reading order for Payux and other work, corpus map, and implementation checklist. **Start agents here** before deep-diving folders.

## Structure

| Folder | Contents |
|--------|----------|
| **empirestate/** | EmpireStack project specification—aspect-based architecture, tenets, roadmap, applications, and aspect docs (OpenErgo, Ledger, Identity, Billing, etc.). Source of truth for what we're building and why. |
| **core-platform-spec/** | Core Platform implementation specification—formal sections (philosophy, deployment, BFF, data, sync, coding standards), addenda (Ergo formal spec, development standards), contracts, policies, and iteration plan. Source of truth for how the platform is built. |
| **core-documentation/** | Shared documentation that commutes across projects—development guide, documentation standards, and coding standards. Project-agnostic; use as reference or copy into applications. |
| **design/** | Implementation handoffs: [design/README.md](design/README.md) indexes **Payux**, **SKU management**, and **shopping cart** (`PAYUX.md`, `SKU-MANAGEMENT.md`, `SHOPPING-CART.md`). |
| **funcspec/** | [funcspec/README.md](funcspec/README.md) — **functional** specifications per application (behavior, UX, boundaries); pairs with `design/`. |
| **concise/** | [concise/README.md](concise/README.md) — lean **infrastructure + strategy** summary; excludes app catalog. |

## Consumption

- **empirestate/** — Optimized for Obsidian; aspect documents combine via links. See [[empirestate/README]] for detailed organization.
- **core-platform-spec/** — Numbered sections with role-based entry points. See `00A_DEVGUIDE.md` for role-specific reading paths. Contracts and policies in subfolders.
- **core-documentation/** — Start with [DEVELOPMENT_GUIDE.md](core-documentation/DEVELOPMENT_GUIDE.md). Use CODING_STANDARDS and DOCUMENTATION_STANDARDS when adopting the workflow.
- **design/** — Scoped blueprints; start with [design/README.md](design/README.md). Pair with [GUIDE.md](GUIDE.md) for implementation agents.
- **funcspec/** — What the app does and what UX it owns; [funcspec/README.md](funcspec/README.md). Read before `design/` when defining product behavior.
- **concise/** — Fast path: tenets, roadmap table, aspects, standards; platform only—[concise/README.md](concise/README.md).

## Relationship

| Library | Audience | Purpose |
|---------|----------|---------|
| **empirestate/** | Technical product manager | Product specification (product = infrastructure). Read, understand, augment. Defines the ideal solution. |
| **core-platform-spec/** | Engineers | Engineering specification. Implementers adhere to these. Tactical constraints and contracts. |
| **core-documentation/** | Engineers, app developers | Development workflow, doc conventions, coding overlay. Commutes across projects. |

Overlap between product spec and engineering spec is expected. empirestate defines *what* and *why*; core defines *how*. Core-documentation's doc model (EXECUTIVE_SUMMARY, DETAILED_DESIGN, etc.) targets application development; empirestate is product/aspect specification—a different doc type.
