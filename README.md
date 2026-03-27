# EmpireStack Documentation

This folder contains documentation for the EmpireStack project. Primary libraries below; **concise/** is a dense platform-only slice (no consuming applications).

**How to use this documentation:** [GUIDE.md](GUIDE.md) — Canonical navigation: philosophy (`TENETS.md`), reading order for Payux and other work, corpus map, and implementation checklist. **Start agents here** before deep-diving folders.

## Structure

| Folder | Contents |
|--------|----------|
| **empirestate/** | EmpireStack project specification—aspect-based architecture, tenets, roadmap, applications, and aspect docs (OpenErgo, Ledger, Identity, Billing, etc.). Source of truth for what we're building and why. |
| **core-platform-spec/** | Core Platform implementation specification—formal sections (philosophy, deployment, BFF, data, sync, coding standards), addenda (Ergo formal spec, development standards), contracts, policies, and iteration plan. Source of truth for how the platform is built. |
| **core-documentation/** | Shared documentation that commutes across projects—development guide, documentation standards, and coding standards. Project-agnostic; use as reference or copy into applications. |
| **applications/** | **Per-application documentation**: overview + optional `FUNCSPEC.md` / `DESIGN.md`. Index: [applications/README.md](applications/README.md) (Payux, SKU-and-Cart stack, host apps). |
| **design/** | Redirect hub → `applications/` ([design/README.md](design/README.md)). |
| **funcspec/** | Redirect hub → `applications/` ([funcspec/README.md](funcspec/README.md)). |
| **concise/** | [concise/README.md](concise/README.md) — lean **infrastructure + strategy** summary; excludes app catalog. |
| **esb-model/** | [esb-model/README.md](esb-model/README.md) — **ESB exercise**: ideal system intent in YAML; evaluates grammar before execution tooling exists. |

## Consumption

- **empirestate/** — Optimized for Obsidian; aspect documents combine via links. See [[empirestate/README]] for detailed organization.
- **core-platform-spec/** — Numbered sections with role-based entry points. See `00A_DEVGUIDE.md` for role-specific reading paths. Contracts and policies in subfolders.
- **core-documentation/** — Start with [DEVELOPMENT_GUIDE.md](core-documentation/DEVELOPMENT_GUIDE.md). Use CODING_STANDARDS and DOCUMENTATION_STANDARDS when adopting the workflow.
- **applications/** — [applications/README.md](applications/README.md); read `FUNCSPEC.md` before `DESIGN.md` for the same app. Pair with [GUIDE.md](GUIDE.md).
- **design/** / **funcspec/** — Short redirects to `applications/` for bookmarks.
- **concise/** — Fast path: tenets, roadmap table, aspects, standards; platform only—[concise/README.md](concise/README.md).
- **esb-model/** — Empire State Build modeling of ideal end-state; see [esb-model/README.md](esb-model/README.md).

## Relationship

| Library | Audience | Purpose |
|---------|----------|---------|
| **empirestate/** | Technical product manager | Product specification (product = infrastructure). Read, understand, augment. Defines the ideal solution. |
| **core-platform-spec/** | Engineers | Engineering specification. Implementers adhere to these. Tactical constraints and contracts. |
| **core-documentation/** | Engineers, app developers | Development workflow, doc conventions, coding overlay. Commutes across projects. |

Overlap between product spec and engineering spec is expected. empirestate defines *what* and *why*; core defines *how*. Core-documentation's doc model (EXECUTIVE_SUMMARY, DETAILED_DESIGN, etc.) targets application development; empirestate is product/aspect specification—a different doc type.
