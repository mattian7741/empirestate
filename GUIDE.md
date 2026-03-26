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

---

## 2. Starting **Payux** implementation

Use this ordered pack when kicking off Payux (M1) or handing work to an agent:

| Step | Read |
|------|------|
| 1 | **`empirestate/TENETS.md`** — execution rules (**1–6**) and doc rules (**7–14**) |
| 2 | **`empirestate/BILLING.md`** — ecosystem boundary: invoice/receipt, cart/SKU/host **not** inside Payux |
| 3 | **`funcspec/PAYUX.md`** — **what** Payux does for users/integrators; **UX ownership**; gateway isolated from catalog/cart |
| 4 | **`design/PAYUX.md`** — **how** to build: APIs, ledger, **pluggable processor** (M1 Stripe blackbox), sequencing |
| 5 | **`empirestate/STANDARDS.md`** + **`core-platform-spec/policies/NO_ORM_POLICY.md`** — data/accessor discipline |
| 6 | **`core-platform-spec/contracts/`** as needed — logging, errors, identity envelope, etc. |
| 7 | **`empirestate/DEPLOYMENT.md`** + **`core-platform-spec/02_SECTION02.md`** — deploy expectations |

**Do not** treat SKU, cart, or product specs as Payux requirements; those live in **`design/SHOPPING-CART.md`**, **`design/SKU-MANAGEMENT.md`**, and **`empirestate/app-documentation/SKU-and-Cart.md`**.

---

## 3. Starting **any other** new project or component

| Step | Read |
|------|------|
| 1 | **`empirestate/TENETS.md`** |
| 2 | **`empirestate/OVERVIEW.md`** — which **aspect(s)** the work belongs to |
| 3 | Relevant **aspect doc(s)** in `empirestate/` (e.g. `LEDGER.md`, `IDENTITY.md`, `DEPLOYMENT.md`, `OPENERGO.md`) |
| 4 | If the component is a named app: **`funcspec/<APP>.md`** when it exists, then **`design/<APP>.md`** (or equivalent) when it exists |
| 5 | **`empirestate/STANDARDS.md`** + **`core-platform-spec/`** policies and contracts that apply |
| 6 | **`empirestate/ROADMAP.md` / `BACKLOG.md`** — only if the work tracks **platform** milestones |

If there is **no** `funcspec` or `design` file yet, product truth still lives in **`empirestate/`**; add scoped specs when the slice is firm enough (**Tenets 7–14**: one home per fact, link the rest).

---

## 4. What each library is for

| Location | Role | When to open it |
|----------|------|-----------------|
| **`empirestate/`** | Product spec: **what** and **why**—aspects, tenets, roadmap, billing philosophy. | Strategy, boundaries, “is this Payux’s job?” |
| **`funcspec/`** | Per-application **functional** spec: flows, UX, plain-English behavior. | Before coding: what users/integrators experience |
| **`design/`** | **Implementation handoff** for a bounded slice: APIs, persistence, sequencing. | While building: concrete build contract |
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

### Applications & Payux stack

| Topic | Document |
|-------|----------|
| App index | `empirestate/APPLICATIONS.md`, `empirestate/app-documentation/README.md` |
| SKU + cart (not Payux) | `empirestate/app-documentation/SKU-and-Cart.md` |
| Payux stub | `empirestate/app-documentation/Payux.md` |
| Payux functional | `funcspec/PAYUX.md` |
| Payux detailed design | `design/PAYUX.md` |
| Design index | `design/README.md` |
| Funcspec index | `funcspec/README.md` |

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
9. **`empirestate/TENETS.md` 7–14** — documentation authority, single source per fact, concise structured spec, layering `empirestate` / `funcspec` / `design`.

---

## 7. When something is unclear

| Question type | Look here first |
|----------------|-----------------|
| “Should we build this? Is it in scope for Payux?” | `empirestate/BILLING.md`, `funcspec/PAYUX.md` |
| “What does the user see?” | `funcspec/` |
| “What do I implement (API/DB)?” | `design/` |
| “What does the platform believe?” | `empirestate/` aspect docs |
| “What are the engineering rules?” | `core-platform-spec/contracts/`, `policies/` |
| “How do we work as a team?” | `core-documentation/DEVELOPMENT_GUIDE.md`, `CODING_STANDARDS.md` |
| “What milestone are we in?” | `empirestate/ROADMAP.md`, `empirestate/BACKLOG.md` |

---

*EmpireStack documentation root: [`README.md`](README.md).*
