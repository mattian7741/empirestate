# Core Platform Specification

Implementation specification for the Core Platform. Originally from `project_base/dev/core-platform-spec`; now consolidated under EmpireStack docs.

## Entry points

- **00A_DEVGUIDE.md** — Role-based reading paths (frontend, BFF, database, Ergo engineers)
- **00B_TOC.md** — Full table of contents
- **REQUIREMENTS.md** — Iteration plan (Core Platform + Hot Potato)

## Structure

- **Sections 01–12** — Philosophy, deployment, cross-cutting aspects, billing, Ergo, data, BFF, frontend, sync, coding standards, contracts, testing
- **AA_ADDENDUM_*** — Formal specs (Ergo, Hot Potato, development standards) and reference notes
- **contracts/** — Canonical contract definitions (identity, audit-event, logging-schema, error-codes, projection-pattern). From core-platform.
- **policies/** — Coding policies (NO_ORM_POLICY, ONE_CLASS_PER_FILE). From core-platform.

## Relationship to EmpireStack

EmpireStack docs (`../empirestate/`) define the strategic architecture. This spec defines implementation constraints, contracts, and engineering norms.
