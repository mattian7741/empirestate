# How to use this documentation

**This file (`GUIDE.md`) is the entry point for humans and AI agents.** It explains **what to read, in what order, and why**, so implementation matches the philosophies and structure of the corpus. It does not duplicate every rule; it **routes** you to the right documents.

---

## 1. Always anchor on philosophy first

| Priority | Document | Why |
|----------|----------|-----|
| **1** | **`empirestate/TENETS.md`** | **Authoritative enumerated constraints.** Order = significance (1 = highest). Governs VCS, architecture fit, lean scope, deploy/iterate live, billing early, and how documentation is written. **Do not skip.** When two guidance sources conflict, **lower tenet numbers win**. |
| **2** | **`empirestate/OVERVIEW.md`** | The five **scaffolding aspects** so work “slots” into the right place. |
| **3** | **`empirestate/ROADMAP.md`** + **`empirestate/BACKLOG.md`** | **What comes when** and iteration-level tasks for the **platform** (not every host app). |

**`concise/`** ([`concise/README.md`](concise/README.md)) is an **optional fast path**: dense platform-only summary (aspects, roadmap table, pointers to full `TENETS.md`). Use it for quick context; it **does not replace** `empirestate/TENETS.md` for implementation discipline.

### Collaboration default: challenge breaks in logic

When working from this corpus, **assistants and reviewers treat pushback as normal**: if a proposal contradicts **tenets**, **documented invariants**, or **coherent reasoning**—including when the latest message conflicts with an earlier constraint—**say so briefly and why** before treating it as settled. The point is **early correction**, not friction for its own sake. When intent genuinely changes, **update the record** (see Tenet 13: synthesize into coherent spec) instead of improvising around contradictions.

---

## 2. Starting **Payux** implementation

Use this ordered pack when kicking off Payux (M1) or handing work to an agent:

| Step | Read |
|------|------|
| 1 | **`empirestate/TENETS.md`** — execution rules (**1–6**) and doc rules (**7–14**) |
| 2 | **`empirestate/BILLING.md`** — ecosystem boundary: invoice/receipt, cart/SKU/host **not** inside Payux |
| 3 | **`applications/Payux/FUNCSPEC.md`** — **what** Payux does for users/integrators; **UX ownership**; gateway isolated from catalog/cart |
| 4 | **`applications/Payux/WORKFLOW.md`** — **who** acts in order: text sequence for OTP, subscription initial, renewal, cancel (participants + numbered steps) |
| 5 | **`applications/Payux/DESIGN.md`** — **how** to build: APIs, ledger, **pluggable processor** (M1 Stripe blackbox), sequencing |
| 6 | **`empirestate/STANDARDS.md`** + **`core-platform-spec/policies/NO_ORM_POLICY.md`** — data/accessor discipline |
| 7 | **`core-platform-spec/contracts/`** as needed — logging, errors, identity envelope, etc. |
| 8 | **`empirestate/DEPLOYMENT.md`** + **`core-platform-spec/02_SECTION02.md`** — deploy expectations |

**Do not** treat SKU, cart, or product specs as Payux requirements; those live in **`applications/SKU-and-Cart/SHOPPING-CART.md`**, **`applications/SKU-and-Cart/SKU-MANAGEMENT.md`**, and **`applications/SKU-and-Cart/SKU-and-Cart.md`**.

---

## 3. Starting **any other** new project or component

| Step | Read |
|------|------|
| 1 | **`empirestate/TENETS.md`** |
| 2 | **`empirestate/OVERVIEW.md`** — which **aspect(s)** the work belongs to |
| 3 | Relevant **aspect doc(s)** in `empirestate/` (e.g. `LEDGER.md`, `IDENTITY.md`, `DEPLOYMENT.md`, `OPENERGO.md`) |
| 4 | If the component is a named app: **`applications/<AppName>/FUNCSPEC.md`** when it exists, optional **`WORKFLOW.md`** (text sequence / numbered use-case steps), then **`applications/<AppName>/DESIGN.md`** (or equivalent, e.g. multiple design files in one folder) when they exist |
| 5 | **`empirestate/STANDARDS.md`** + **`core-platform-spec/`** policies and contracts that apply |
| 6 | **`empirestate/ROADMAP.md` / `BACKLOG.md`** — only if the work tracks **platform** milestones |

If there is **no** `FUNCSPEC.md` / `DESIGN.md` under that application yet, product truth still lives in **`empirestate/`**; add scoped specs under **`applications/<AppName>/`** when the slice is firm enough (**Tenets 7–14**: one home per fact, link the rest). Add **`WORKFLOW.md`** when a **ordered multi-actor** narrative helps integrators without drawing diagrams.

---

## 4. What each library is for

| Location | Role | When to open it |
|----------|------|-----------------|
| **`empirestate/`** | Product spec: **what** and **why**—aspects, tenets, roadmap, billing philosophy. | Strategy, boundaries, “is this Payux’s job?” |
| **`applications/`** | Per-application folders: overview doc, optional **`FUNCSPEC.md`** (behavior, UX), optional **`WORKFLOW.md`** (text sequence / numbered steps), **`DESIGN.md`** (implementation handoff). | Before/during coding for a named app |
| **`design/`**, **`funcspec/`** | **Redirects** to `applications/` — bookmarks and short pointers only. | Use `applications/README.md` as the index |
| **`core-platform-spec/`** | Engineering spec: contracts, policies, numbered sections. | Cross-cutting **how** and compliance |
| **`core-documentation/`** | Dev workflow, coding overlay, doc conventions. | Process and day-to-day engineering habits |
| **`concise/`** | Short platform-only digest; links outward. | Quick orientation; **not** a substitute for **TENETS** |

**Principle:** Pick the **shallowest** document that answers the question; **link** instead of duplicating (**Tenet 8–9**).

---

## 5. Where to find common topics

### Core principles

| Topic | Document |
|-------|----------|
| **Tenets** | `empirestate/TENETS.md` |
| **Aspects (scaffolding)** | `empirestate/OVERVIEW.md` |
| **Roadmap / backlog** | `empirestate/ROADMAP.md`, `empirestate/BACKLOG.md` |

### Implementation standards

| Topic | Document |
|-------|----------|
| **Coding standards** | `empirestate/STANDARDS.md` |
| **No ORM policy** | `core-platform-spec/policies/NO_ORM_POLICY.md` |
| **One class per file** | `core-platform-spec/policies/ONE_CLASS_PER_FILE.md` |
| **Coding overlay** | `core-documentation/CODING_STANDARDS.md` |

### Contracts (cross-boundary)

| Topic | Document |
|-------|----------|
| Identity | `core-platform-spec/contracts/identity.md` |
| Logging | `core-platform-spec/contracts/logging-schema.md` |
| Errors | `core-platform-spec/contracts/error-codes.md` |
| Audit | `core-platform-spec/contracts/audit-event.md` |
| Projection / accessors | `core-platform-spec/contracts/projection-pattern.md` |

### Aspects (domain behavior)

| Topic | Document |
|-------|----------|
| OpenErgo | `empirestate/OPENERGO.md` |
| Ledger | `empirestate/LEDGER.md` |
| Identity | `empirestate/IDENTITY.md` |
| Billing | `empirestate/BILLING.md` |
| Deployment | `empirestate/DEPLOYMENT.md` |
| Viral invite | `empirestate/VIRAL.md` |

### Data & deployment (deeper)

| Topic | Document |
|-------|----------|
| Data stores | `empirestate/STANDARDS.md`, `core-platform-spec/06_SECTION06.md` |
| Sync | `empirestate/LEDGER.md`, `core-platform-spec/09_SECTION09.md` |
| Deploy layers | `empirestate/DEPLOYMENT.md`, `core-platform-spec/02_SECTION02.md` |
| ESB protocol stress test (lean YAML) | `esb-model/README.md`, `esb-model/ideal-system.esb.yaml` |

### Applications & Payux stack

| Topic | Document |
|-------|----------|
| App index | `empirestate/APPLICATIONS.md`, `applications/README.md` |
| SKU + cart (not Payux) | `applications/SKU-and-Cart/SKU-and-Cart.md` |
| Payux stub | `applications/Payux/Payux.md` |
| Payux functional | `applications/Payux/FUNCSPEC.md` |
| Payux workflow (text sequence) | `applications/Payux/WORKFLOW.md` |
| Payux detailed design | `applications/Payux/DESIGN.md` |
| Legacy index (redirects) | `design/README.md`, `funcspec/README.md` |

---

## 6. Checklist: constraints every implementer should honor

1. **`empirestate/TENETS.md` 1–6** — VCS from day one; fit the architecture; **lean** iteration scope; **deploy** defines done; iterate on **live** systems; **billing** trajectory when relevant. Lower numbers win on conflict.
2. **Accessor / data** — `NO_ORM_POLICY` + `STANDARDS.md`: explicit accessors; ORM only behind blackbox if used.
3. **Right store for the job** — SQL vs NoSQL vs streams per domain; no hype-driven substitution.
4. **Ledger pattern** where billing/datastore docs apply — append-only, immutable, projection-friendly.
5. **Identity defaults** — logical silos; physical silos opt-in; see `IDENTITY.md`.
6. **Deployment** — layered, deterministic, portable; see `DEPLOYMENT.md`.
7. **Contracts** — structured logs, canonical errors, `correlation_id` where specified.
8. **Typing & interface-first** — `STANDARDS.md`.
9. **`empirestate/TENETS.md` 7–14** — documentation authority, single source per fact, concise structured spec, layering `empirestate` / **`applications/<App>/`** (`FUNCSPEC.md`, optional `WORKFLOW.md`, `DESIGN.md`).

---

## 7. When something is unclear

| Question type | Look here first |
|----------------|-----------------|
| “Should we build this? Is it in scope for Payux?” | `empirestate/BILLING.md`, `applications/Payux/FUNCSPEC.md` |
| “What does the user see?” | `applications/<App>/FUNCSPEC.md` when present |
| “What do I implement (API/DB)?” | `applications/<App>/DESIGN.md` (or app-named design files in that folder) |
| “What does the platform believe?” | `empirestate/` aspect docs |
| “What are the engineering rules?” | `core-platform-spec/contracts/`, `policies/` |
| “How do we work as a team?” | `core-documentation/DEVELOPMENT_GUIDE.md`, `CODING_STANDARDS.md` |
| “What milestone are we in?” | `empirestate/ROADMAP.md`, `empirestate/BACKLOG.md` |

---

*EmpireStack documentation root: [`README.md`](README.md).*
