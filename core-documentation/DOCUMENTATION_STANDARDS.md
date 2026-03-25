# Documentation standards

Markdown and structural conventions used across all project documentation so that docs have a **uniform representation**. Apply these when creating or editing any document under `docs/` in an application, or when editing core docs in this project. Documentation is split into **core** (this project — core-documentation; commutes across projects) and **project** (specific to each application; lives in `docs/` in that app's repo); see DEVELOPMENT_GUIDE.md for the full map.

**Relationship:** This document defines **how** docs are formatted and organized and **what** documents are required for a project (and whether each is core or project-specific). What to document in each comes from DEVELOPMENT_GUIDE and the purpose of each doc type.

---

## 1. Required and optional documents for a project

Every project that follows this system should have the documents below. **Core** documents live in this project (core-documentation) and are copied as-is or referenced as a sibling. **Project-specific** documents live in `docs/` in each application repo; their **structure** is defined here and in DEVELOPMENT_GUIDE, but their **content** is specific to the application. Having only the core docs gives a developer everything needed to know what project docs to create and how to structure them.

| Document | Location | Required | Purpose | Structure guidelines (project-specific only) |
|----------|----------|----------|---------|---------------------------------------------|
| **DEVELOPMENT_GUIDE.md** | core (this project) | Yes | Entry point; document map; how to use docs together; prerequisites; flow. | N/A (core). |
| **DOCUMENTATION_STANDARDS.md** | core (this project) | Yes | Markdown and doc conventions; required doc set; structure for project docs. | N/A (core). |
| **CODING_STANDARDS.md** | core (this project) | Yes | How to implement (git, deployment, state-over-change, payload schemata, etc.). Orthogonal to backlog. | N/A (core). |
| **BACKLOG.md** | docs/ (in each app) | Yes | Iterations to align implementation with spec. Outcome-focused; order by impact and blast radius. | Lead: ordering principle and gates (e.g. deployment before dev). One **## Iteration N — Title** per iteration; each has **Goal**, checklist `- [ ]` / `- [x]`, **Deliverables**. Optional gap/summary table. End: **Suggested order**, **Reference** list. See §10 BACKLOG. |
| **EXECUTIVE_SUMMARY.md** | docs/ (in each app) | Yes | One-page executive summary: product, audience, key capabilities, current status. For stakeholders and new contributors. | Lead: product and audience in one short paragraph. Sections: **Product**, **Key capabilities** (bullets), **Current status** (release/iteration, deployment). Optional **Roadmap** (one line or pointer to BACKLOG). One page; no implementation detail. See §10 EXECUTIVE_SUMMARY. |
| **DETAILED_DESIGN.md** | docs/ (in each app) | Yes | Technical approach per iteration. Just-in-time; by topology/feature. | Lead: scope (app-specific), structure (by topology/feature). Sections by application area. Reference BACKLOG iteration in section titles. Tables (Area \| Action) for artifacts to touch. See §10 DETAILED_DESIGN. |
| **Logical-spec** (e.g. PSEUDO_LOGIC.md) | docs/ (in each app) | Yes | Canonical logical rules (pseudo code): what the app computes. Single source for rules. | Name project-specific. Consistent pseudo-code style (`//` comments, `=`, `case | ... end`). Parts: governing principles, functional rules, UI discipline. |
| **FUNCTIONAL_SPECIFICATION.md** | docs/ (in each app) | Yes | Full functional behavior; user and system perspective. Expands logical-spec. | Sections by feature/flow. Behavior and invariants only; no implementation. Reference logical-spec as source of truth. |
| **FUNCTIONAL_CHEATSHEET.md** | docs/ (in each app) | Yes | Minimal behavior summary. Quick reference. | Short sections or bullets; key rules and outcomes only. |
| **Rules doc** (e.g. EXPLORER_RULES.md) | docs/ (in each app) | Yes | Mechanics and edge cases. Refined rules; correct here before code. | Name project-specific. Sections by domain. Tables/bullets for cases. Reference from contract and DETAILED_DESIGN. |
| **ISSUES.md** | docs/ (in each app) | Yes | Tactical issues (docs, code, UX). Ordered by priority. | Lead: order, when to fix, distinct from BACKLOG. Checklist `- [ ]` / `- [x]`; reorder by priority. See §10 ISSUES. |
| **API contract** (e.g. explorer_api_contract.md) | docs/ (in each app) | Yes (if app has API) | Routes, request/response shapes, errors. Contract at boundary. | Name project-specific. Numbered sections. Tables: endpoints, error codes, term mapping. Code blocks for envelope/shapes. Final line: *This contract is the single source of truth for …* See §10 API contract. |
| **REFACTOR_ITERATION_PLAN.md** | docs/ (in each app) | Optional | Development-standards refactor. Complements BACKLOG. | If present: iterations with goal, scope, deliverables. Reference CODING_STANDARDS and spec docs. |

*Logical-spec, rules doc, and API contract use project-chosen names (e.g. PSEUDO_LOGIC.md, EXPLORER_RULES.md, explorer_api_contract.md). The role and structure of each type commute.*

---

## 2. Document title and lead

- **First line:** Single `#` heading with the document name. Optional subtitle after an em dash: `# Name — Subtitle`.
- **Lead:** One to three short paragraphs under the title that state purpose and scope. No redundant "This document describes…" unless the doc is long or multi-part.
- **No trailing meta** (e.g. "Last updated") unless the project explicitly requires it.

---

## 3. Sectioning and horizontal rules

- **Major sections:** `## Section title`. Use sentence case or title case consistently within a doc; the project uses **sentence case** for most section titles (e.g. "Document map", "How to use them together").
- **Subsections:** `### Subsection` when a section has distinct parts. Deeper levels (`####`) only when necessary; prefer flattening or a short table/list.
- **Horizontal rules:** Use `---` (three hyphens, blank line above and below) to separate major sections. Do not use `---` between every subsection; use it to create visible "chapters" (e.g. after the lead, between each top-level `##` block).
- **Section references:** When pointing to another document or section, use **§** for section: e.g. "CODING_STANDARDS § Git management", "docs/BACKLOG § Deployment iteration", "see § Suggested order".

---

## 4. Emphasis and typography

- **Bold (`**text**`):** Document names (e.g. **docs/BACKLOG.md**), key terms (e.g. **canonical**, **negation**), UI/API names (e.g. **PUT `/api/vpath`**), and labels that must stand out (e.g. **Goal**, **Deliverables**). Use sparingly; overuse reduces effect.
- **Italic (`*text*`):** Notes, asides, or gentle emphasis in prose (e.g. *Update checkboxes as work completes.*). Not for document titles or technical identifiers.
- **Inline code (backticks):** File names (`package.json`), code identifiers (`tags_null`), API paths (`/api/listing`), keys, env vars, and short code snippets. Use `` ` `` not smart quotes.
- **Math / logic:** Use Unicode where it aids readability (e.g. ∪ ∖ ∈ →); otherwise use plain text or code (e.g. `vpath ?? path`).

---

## 5. Lists

- **Unordered:** Single hyphen and space: `- item`. One line per item unless a single item needs a short continuation; then indent the continuation or keep it on the same line.
- **Checkboxes (tasks / issues):** `- [ ]` for open, `- [x]` for done. Used in docs/BACKLOG.md and docs/ISSUES.md. Reorder by priority or dependency; do not mix checkbox style with plain bullets in the same list.
- **Numbered:** Use `1.` `2.` when order or count matters (e.g. "Three approaches: 1. Canonical …"). Nested sub-items: indent with two spaces and use `-` or a letter.
- **Nested lists:** Indent with two spaces under the parent item. Use `-` for nested bullets; for numbered sub-items, use `-` with bold number or letter if needed (e.g. **1.** … **2.** …).

---

## 6. Tables

- **Format:** Pipe tables with a header row and one alignment row: `| col1 | col2 |` then `|------|------|`. Align with spaces for readability is optional; at least one dash per column.
- **Cell content:** Keep concise. Use **bold** in cells for terms or labels; use `code` for identifiers, paths, or keys. Avoid full sentences in cells when a phrase suffices.
- **No leading/trailing pipes** required by standard Markdown; the project uses them consistently: `| A | B |`.

---

## 7. Code and pseudo-code blocks

- **Fenced blocks:** Use triple backticks and, when helpful, a language tag: ` ```json `, ` ```python `, ` ``` ` (plain). Put the opening fence on its own line; no inline fenced code.
- **Pseudo-code / grammar:** For logical or algebraic specs (project logical-spec), use a consistent style: `//` for comments, `=` for definition, `case | ... | ... end` for branching. Preserve that style within that document.
- **Inline code:** Backticks only; no fenced blocks mid-sentence.

---

## 8. Cross-references and document names

- **Core docs:** Refer as **FILENAME.md** or "CODING_STANDARDS" when in the same (core) project. From an application's docs, refer with the path to core (e.g. `../core-documentation/CODING_STANDARDS.md`). Add section when needed: "CODING_STANDARDS § Deployment and CI/CD".
- **Project docs:** In an application, refer as **docs/FILENAME.md** or "docs/BACKLOG" when the context is clear. Add section when needed: "docs/BACKLOG § Deployment iteration".
- **Same doc:** "See § Document map", "§ Deployment iteration".
- **Links:** Use `[text](url)` for external URLs. Internal links (e.g. `[BACKLOG](BACKLOG.md)` within docs/) are optional when the doc tree is known.

---

## 9. Discipline and convergence

- **Convergent:** Documentation should evolve so that it aligns across docs; no contradictory statements. When behavior or strategy changes, update all affected docs (see DEVELOPMENT_GUIDE).
- **Single source of truth:** For each topic (e.g. API contract, logical rules, coding strategy), one doc is authoritative; others **reference** it and do not duplicate its wording. Avoid copy-pasting spec or contract text into multiple files.
- **Brevity:** Prefer short sentences and lists over long paragraphs. Reduce wordiness, redundancy, and logical repetition. Challenge every paragraph: can it be a list or a table?

---

## 10. Document-specific patterns

These patterns are part of the uniform representation for each doc type. **Core** docs live in this project; **project** docs live in `docs/` in each application and follow the structure below.

- **DEVELOPMENT_GUIDE.md:** Two-part document map (Core \| Project-specific with structure guidelines). Numbered steps (**0.** **1.** …) for "How to use them together". Flow summary in a single fenced block (no language tag).
- **CODING_STANDARDS.md:** Top-level sections for first-class concerns (e.g. **Git management**, **Deployment and CI/CD**); then bullet lists of standards. Use **Bold** for the standard label at the start of each bullet where it helps scan.
- **docs/BACKLOG.md:** Each iteration has **Goal**, bullet tasks with `- [ ]` / `- [x]`, and **Deliverables**. Use a gap/summary table where useful. End with **Suggested order** and **Reference** list. *Update checkboxes as work completes.*
- **docs/EXECUTIVE_SUMMARY.md:** One-page summary for stakeholders and new contributors. Lead: product and audience. Sections: Product, Key capabilities, Current status; optional Roadmap. No implementation detail.
- **docs/DETAILED_DESIGN.md:** Sections by **topology/feature**, not by time. Reference BACKLOG iteration in section titles or lead (e.g. "Iteration 1 — …"). Tables for "Artifacts to touch" (Area \| Action).
- **docs/** logical-spec: Consistent pseudo-code style; governing principles, then functional rules, then UI discipline.
- **docs/FUNCTIONAL_SPECIFICATION.md:** Sections by feature/flow; behavior and invariants; no implementation detail.
- **docs/FUNCTIONAL_CHEATSHEET.md:** Short sections or bullets; key rules only.
- **docs/** rules doc: Sections by domain; tables and bullets for edge cases.
- **docs/ISSUES.md:** Lead (order, when to fix, distinct from BACKLOG). Checklist `- [ ]` / `- [x]`; reorder by priority.
- **docs/** API contract: Numbered top-level sections (0, 1, 2, …). Tables for endpoints, error codes, term mapping. JSON code blocks for envelope and shapes. Final line: *This contract is the single source of truth for …*

---

*Apply these standards when adding or editing any project documentation. For the document map and how docs work together, see DEVELOPMENT_GUIDE.md.*
