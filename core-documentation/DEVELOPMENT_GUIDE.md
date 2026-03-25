# Development Guide

Entry point for development. This guide names the project's key documents and describes how to use them together to advance the project. Documentation is split into **core** (this project — core-documentation; commutes across projects; copy as-is or reference as sibling) and **project** (specific to each application; create in `docs/` in that application's repo).

**Backlog:** Evaluate, re-evaluate, and update the backlog as context about the project improves. **All other documentation:** When you find conflicts, contradictions, divergences, or strategic shifts, raise them and remediate (fix the docs or the code so they align).

**Documentation discipline:** All documents must remain under heavy scrutiny and be challenged on any frictional or inconsistent encounter. Documentation must evolve in a **convergent** way. Prioritize and claim any opportunity to reduce documentation complexity, wordiness, logical load, or redundancy.

**First-order prerequisites:** **Git hygiene** (see CODING_STANDARDS § Git management) and **deployment strategy and CI/CD** (see CODING_STANDARDS § Deployment and CI/CD). Neither is optional for non-sandbox projects: resolve both before development. Deployment requires a chosen approach and a hello-world MVP run through the deployment workflow.

---

## Document map

### Core documents (this project — copy as-is or use as sibling)

These documents commute: they contain no project-specific behavior or design. Copy this project (or its contents) into your workspace, or keep it as a sibling (e.g. `docs/core-documentation`) and reference it from application docs.

| Document | Purpose |
|----------|---------|
| **DEVELOPMENT_GUIDE.md** (this file) | Entry point. Map of documents; which are core vs project-specific; how to use them together. |
| **DOCUMENTATION_STANDARDS.md** | Markdown and structural conventions for all docs; uniform representation. Defines required vs optional doc types and structure for project-specific docs. |
| **CODING_STANDARDS.md** | How to implement: overlay on default approach (no ORMs, pure functions, interfaces, config-driven, deployment, etc.). Orthogonal to backlog. |

---

### Project-specific documents (create in `docs/` in each application — structure commutes, content is specific)

You must create these documents when starting a new project. The **structure** of each is defined here and in DOCUMENTATION_STANDARDS.md; the **content** is specific to your application. Having only the core docs (this project) gives you everything needed to know *what* to create and *how* to structure it.

| Document | Purpose | Structure guidelines |
|----------|---------|----------------------|
| **docs/EXECUTIVE_SUMMARY.md** | One-page executive summary: what the product is, who it's for, key capabilities, current status. For stakeholders, new contributors, and decision-makers. | Lead: one short paragraph stating product and audience. Sections: **Product** (what it is), **Key capabilities** (bullets or short list), **Current status** (release/iteration state, deployment). Optional: **Roadmap** (one line or pointer to BACKLOG). Keep to one page; no implementation detail. See DOCUMENTATION_STANDARDS § EXECUTIVE_SUMMARY. |
| **docs/BACKLOG.md** | Iterations to align implementation with the spec. Outcome-focused tasks; order by impact and blast radius. | Lead with ordering principle and any gates (e.g. deployment before dev). Sections: gap/summary table (optional), then one **## Iteration N — Title** per iteration. Each iteration: **Goal**; checklist tasks `- [ ]` / `- [x]`; **Deliverables**. End with **Suggested order** (flow or diagram) and **Reference** list. See DOCUMENTATION_STANDARDS § BACKLOG. |
| **docs/DETAILED_DESIGN.md** | Technical approach and implementation strategy per iteration. Just-in-time; driven by backlog, CODING_STANDARDS, and codebase. | Lead: scope (app-specific only), structure (by topology/feature, not by time). Sections by **application area** (e.g. meta layer, API routes, frontend). When adding work, reference BACKLOG iteration in section title or lead. Tables for "Artifacts to touch" (Area \| Action) where useful. See DOCUMENTATION_STANDARDS § DETAILED_DESIGN. |
| **docs/** *logical-spec* | Canonical logical rules (pseudo code): what the app computes (path, state, tags, visibility, etc.). Single source for "what" at the rule level. | Name and format are project-specific (e.g. PSEUDO_LOGIC.md with `//` comments and `case \| ... end`). Content: governing principles, then functional rules, then UI discipline. Reference from FUNCTIONAL_SPECIFICATION and API contract. |
| **docs/FUNCTIONAL_SPECIFICATION.md** | Full functional behavior from user and system perspective. Expands and aligns with the logical-spec. | Sections by feature or flow. No implementation detail; behavior and invariants. Reference the logical-spec and rules doc as source of truth. |
| **docs/FUNCTIONAL_CHEATSHEET.md** | Minimal summary of behavior. Quick reference; not a substitute for the spec when behavior is ambiguous. | Short sections or bullet list. Key rules and outcomes only. |
| **docs/** *rules* | Mechanics and edge cases (e.g. move, rename, scope, visibility). Refined rules; correct here before changing code. | Name project-specific (e.g. EXPLORER_RULES.md). Sections by domain (e.g. move/rename, scope, empty). Tables and bullets for cases. Reference from contract and DETAILED_DESIGN. |
| **docs/ISSUES.md** | Tactical issues (docs, code, UX). Bugs/friction in current state; ordered by priority. | Lead: order (priority), when to fix (proximity, user complaint), distinct from BACKLOG. Section of checklist items `- [ ]` / `- [x]`; reorder by priority. See DOCUMENTATION_STANDARDS § ISSUES. |
| **docs/** *api_contract* | API routes, request/response shapes, errors. Contract at the HTTP (or other) boundary. | Name project-specific (e.g. explorer_api_contract.md). Numbered top-level sections. Tables for endpoints, error codes, term mapping. JSON/code blocks for envelope and shapes. Final line: *This contract is the single source of truth for …* See DOCUMENTATION_STANDARDS § API contract. |
| **docs/REFACTOR_ITERATION_PLAN.md** | Optional. Development-standards refactor (contracts, domain extraction, testing). Complements BACKLOG. | If present: iterations with goal, scope, deliverables. Reference CODING_STANDARDS and project spec docs as source of truth. |

*In the table above, *logical-spec*, *rules*, and *api_contract* are placeholders for whatever naming the project uses (e.g. PSEUDO_LOGIC.md, EXPLORER_RULES.md, explorer_api_contract.md). The structure and role of each type commute.*

---

## How to use them together

**0. Prerequisites (must be satisfied before development)**

- **Git:** Active repo; no unresolved unstaged changes; work on a branch, not default (CODING_STANDARDS § Git management).
- **Deployment and CI/CD:** Choose one of three approaches and complete the deployment workflow with a hello-world MVP (CODING_STANDARDS § Deployment and CI/CD):
  - **Canonical** — Use the deployment strategy defined in CODING_STANDARDS (§ Canonical deployment strategy); shared across projects.
  - **Custom** — Use a custom strategy with an **up-front, documented justification** for not using the canonical approach. Justification is the only valid substitute for the prerequisite.
  - **No deployment** — Sandbox or other **justified** reason not to have a deployment strategy; document the reason. Unjustified "no deployment" does not satisfy the prerequisite.
- A **hello-world MVP** must be implemented and run through the deployment workflow to complete this prerequisite.
- **If you are midstream on an iteration:** Complete that iteration first (merge, push, update docs/BACKLOG). Then satisfy the deployment prerequisite before starting the next development iteration. See docs/BACKLOG § Deployment iteration (or equivalent gate).

**1. Starting development**

- Read this guide and the project's **executive summary** (docs/EXECUTIVE_SUMMARY.md) and **functional cheatsheet** (docs/FUNCTIONAL_CHEATSHEET.md or equivalent) to understand what the app does.
- Use the project's **logical-spec** when you need the exact rule.
- Use the project's **rules** doc and **FUNCTIONAL_SPECIFICATION** when behavior or edge cases are unclear.

**2. Choosing work**

- Work from **docs/BACKLOG.md**. Pick an iteration (or the next unchecked task in the suggested order).
- **Do not start a new development iteration** until any **Deployment iteration** (or equivalent gate) is satisfied. See docs/BACKLOG § Suggested order.
- Backlog tasks are outcomes, not implementation steps. Use the spec docs to know *what* must be true when the task is done.

**3. Detailed design (just-in-time)**

- Before implementing an iteration, **create a new branch** (do not work on the default branch). Then capture the technical approach in **docs/DETAILED_DESIGN.md**. It is driven by: the backlog (requirements for that iteration), CODING_STANDARDS (constraints), and the existing codebase.
- **Structure DETAILED_DESIGN by application topology and features**, not by the order in which work was done. Place content under the relevant area (e.g. meta layer, API routes, frontend) rather than appending at the end.

**4. Implementing**

- Apply **CODING_STANDARDS.md** while implementing backlog items. Standards define *how*; backlog defines *what*.
- At boundaries (API, DB, UI), follow the project's **API contract** (docs/*.md) for routes, payloads, and errors.
- When a rule is ambiguous, resolve it in the project's rules doc or logical-spec first, then change code.

**5. Completing work**

- Before considering an iteration complete: update **docs/BACKLOG.md** checkboxes; update **docs/DETAILED_DESIGN.md** for any touched areas (under the right topology/feature); run tests and add or update them when behavior or contract changes; merge iteration branch to default, commit, and push (see CODING_STANDARDS § Git management); remediate any doc conflicts you found.
- When implementing, consider fixing a **small number of docs/ISSUES.md** items that have proximity to the current iteration (or that the user has raised); do not overload the iteration with issues.
- In DETAILED_DESIGN, reference the backlog iteration (e.g. "Iteration 2") in sections you add or change so design and work stay traceable.

**6. Cross-cutting concerns**

- **New behavior or rule** → Document in the project's logical-spec and/or rules doc; add or adjust backlog items if implementation is needed.
- **Incidental issue** (doc, code, or UX friction) → Add to **docs/ISSUES.md** with checkbox; order by priority. Consider including a small number in the current iteration if they have proximity or the user has raised them.
- **API change** → Update the project's API contract and any spec that references the API.
- **Refactor for structure only** → docs/REFACTOR_ITERATION_PLAN may apply; keep behavior aligned with the spec docs.

---

## Where to capture strategy decisions

When an implementation-strategy decision is made and codified, decide whether it is **generally applicable** (to any application) or **specific** (to this application).

- **Generally applicable** → Capture in **CODING_STANDARDS.md**. These are constraints and approaches that apply regardless of the current app (e.g. no ORMs, pure functions at the core, interfaces for contracts).
- **Specific** → Capture in **docs/DETAILED_DESIGN.md** (or in docs/FUNCTIONAL_SPECIFICATION or the project's rules doc if the decision forces a re-evaluation of requirements).

docs/DETAILED_DESIGN should not relitigate standards already established in CODING_STANDARDS, and should not repeat them. It builds on those standards with app-specific technical choices.

---

## Flow summary

```
Prerequisites          →  Git (branch, clean tree) + Deployment (canonical | custom justified | no-deployment justified) + hello-world MVP
Understand behavior    →  docs CHEATSHEET, SPEC, logical-spec, rules
Choose work            →  docs/BACKLOG
Design (JIT)           →  docs/DETAILED_DESIGN (by topology/features)
Implement              →  CODING_STANDARDS + docs API contract
Resolve ambiguity      →  docs rules / logical-spec first, then code
Log / fix tactical     →  docs/ISSUES (priority order; few per iteration when proximal or user-raised)
```

Start here; use the map and flow above to navigate the rest. For the full list of required documents and their structure, see DOCUMENTATION_STANDARDS.md § Required and optional documents.
