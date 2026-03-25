# EmpireStack Documentation

This folder contains the canonical documentation for EmpireStack—the source of truth and record for project specification.

## Purpose

Documentation here is **not** end-user or developer documentation. Those are generated as renderings from specific lenses into this corpus. This documentation captures facts concisely as isolated aspects that combine for rich multidimensional significance.

## Document Organization

| Document | Purpose |
|----------|---------|
| **OVERVIEW.md** | High-level project description |
| **STACK.md** | Selected tools and technologies |
| **TENETS.md** | **14** commandments, **significance-ordered** (1 = VCS from start … 6 = billing early … 7–14 = documentation) |
| **STANDARDS.md** | Development, implementation, and coding standards |
| **LOG.json** | Sequential record of user submissions to the specification thread |
| **README.md** | This file—organization and consumption guide |
| **OPENERGO.md** | OpenErgo's role as the backend integration substrate |
| **LEDGER.md** | Distributed datastore: append-only ledger of claims |
| **IDENTITY.md** | Universal user identity, authentication, authorization, and entitlements |
| **VIRAL.md** | User-initiated transactional invite; emergent-user participation without sign-up |
| **BILLING.md** | Universal billing gateway: invoice → payment → receipt |
| **DEPLOYMENT.md** | Target-agnostic deployment; infrastructure as disposable |
| **ROADMAP.md** | Strategic milestones; implementation order |
| **BACKLOG.md** | Iterations and subtasks; tracks progress |
| **APPLICATIONS.md** | Index of applications; links to app-documentation/ |
| **app-documentation/** | One document per application (Payux, WishingWell, HotPotato, etc.) |

**Detailed design (implementation handoffs):** `../design/` — e.g. `../design/PAYUX.md` for Payux.

Additional documents are introduced as needed.

## Consumption

- **Format**: Markdown (optimized for Obsidian vaults)
- **Structure**: Isolated aspect documents; combine via links, tags, or queries
- **Principle**: Each document captures one dimension; together they form the full specification
