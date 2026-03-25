# Core Documentation

Shared documentation that **commutes across projects**: development guide, documentation standards, and coding standards. No project-specific content. Originally from `project_base/dev/core-documentation`; now consolidated under EmpireStack docs.

**Use in two ways:**

1. **Reference from EmpireStack** — Use as sibling to empirestate and core-platform-spec. From empirestate, reference via `../core-documentation/` (e.g. `../core-documentation/DEVELOPMENT_GUIDE.md`).
2. **Copy into a project** — Copy the contents into an app's `docs/` (e.g. `docs/core/` or `docs/core-documentation/`) and reference from that project's docs.

**Entry point:** [DEVELOPMENT_GUIDE.md](DEVELOPMENT_GUIDE.md) — document map, prerequisites, how to use core and project docs together, and the list of project-specific documents to create.

## Contents

| Document | Purpose |
|----------|---------|
| **DEVELOPMENT_GUIDE.md** | Entry point. Map of documents; core vs project-specific; how to use them together. |
| **DOCUMENTATION_STANDARDS.md** | Markdown and structural conventions; uniform representation; required doc types and structure. |
| **CODING_STANDARDS.md** | How to implement: git, deployment, state-over-change, payload schemata, no ORMs, pure functions, etc. Overlay on default approach. |
