# EmpireStack — AI Agent Guide

This document orients a new AI agent session to the EmpireStack documentation. Use it to locate principles, constraints, and implementation guidance. When implementing, adhere to the specifications referenced here.

---

## Purpose

EmpireStack is a universal software architecture—event-driven nano services, distributed ledger, identity, billing, deployment—unified into one coherent stack. Documentation is organized into **product specification** (what/why) and **engineering specification** (how). You should leverage both.

---

## Documentation Structure

| Location | Role | When to use |
|----------|------|-------------|
| **empirestate/** | Product spec. Defines ideal solution, aspects, roadmap. TPM audience. | Context, strategic direction, aspect behavior. |
| **core-platform-spec/** | Engineering spec. Contracts, policies, formal sections. Engineer audience. | Implementation constraints, APIs, coding rules. |
| **core-documentation/** | Dev workflow, doc standards, coding overlay. Commutes across projects. | Process, conventions, deployment/coding patterns. |

**Principle:** empirestate = *what* and *why*; core = *how*. There is overlap; both matter for implementation.

**Fast path (platform only, no app catalog):** `concise/README.md`

---

## Where to Find Important Information

### Core Principles (read first)

| Topic | Document | Key points |
|-------|----------|------------|
| **Tenets** | `empirestate/TENETS.md` | **14 enumerated commandments (order = significance):** **source control first (1)**, scaffolding (**2**), lean (**3**), deploy + iterate live (**4–5**), billing early (**6**), documentation (**7–14**). |
| **Overview** | `empirestate/OVERVIEW.md` | Five aspects: OpenErgo, Ledger, Identity, Billing, Deployment. |
| **Roadmap** | `empirestate/ROADMAP.md` | M1 Payux, M2 Deployment, M3 OpenErgo, M4 Identity, M5 Ledger, M6 Viral. Billing first. |

### Implementation Standards

| Topic | Document | Key points |
|-------|----------|------------|
| **Coding standards** | `empirestate/STANDARDS.md` | No ORM as abstraction (accessor blackbox OK). SQL/NoSQL/event streams all valid; right tool for job. OOP + pure functions. Strong typing. Interface-first. Python preferred. |
| **No ORM policy** | `core-platform-spec/policies/NO_ORM_POLICY.md` | Accessor interface is contract; ORM behind blackbox is implementation detail. |
| **One class per file** | `core-platform-spec/policies/ONE_CLASS_PER_FILE.md` | One primary public construct per file; flat structures. |
| **Coding overlay** | `core-documentation/CODING_STANDARDS.md` | Git hygiene, deployment prerequisite, state over change, payload schemata. |

### Contracts (cross-boundary)

| Topic | Document | Key points |
|-------|----------|------------|
| **Identity** | `core-platform-spec/contracts/identity.md` | Tenant, principal, logical vs physical silos, white-listing for commuting. |
| **Logging** | `core-platform-spec/contracts/logging-schema.md` | Structured JSON, correlation_id, minimum fields. |
| **Error codes** | `core-platform-spec/contracts/error-codes.md` | AUTH, VALIDATION, NOT_FOUND, etc. Canonical envelope. |
| **Audit events** | `core-platform-spec/contracts/audit-event.md` | actor, target, action, timestamp, correlation_id. |
| **Projection pattern** | `core-platform-spec/contracts/projection-pattern.md` | Migrations, accessors, idempotency keys, tenant scope. |

### Aspects (domain behavior)

| Topic | Document | Key points |
|-------|----------|------------|
| **OpenErgo** | `empirestate/OPENERGO.md` | Transport-agnostic substrate. Kafka/RMQ is example stack; many transports supported. |
| **Ledger** | `empirestate/LEDGER.md` | Ledger pattern: append-only, immutable, projection-derived. Applied to financial, claims, social, record-based. Datastore topology (distributed vs centralized) is application-dependent. |
| **Identity** | `empirestate/IDENTITY.md` | Participation-first, roundtrip auth, commuting identity. Logical silos default; physical opt-in; white-list for sharing. |
| **Billing** | `empirestate/BILLING.md` | Invoice → payment → receipt; opaque metadata only. App owns products/services + entitlements (backref receipt/gift); SKU manager maps to SKUs/prices. Uses ledger pattern. |
| **Deployment** | `empirestate/DEPLOYMENT.md` | Target-agnostic, declarative, Docker. Automation stands alone; IaC; layered model (intent → Ansible → containers → libraries). |

### Data Architecture

| Topic | Document | Key points |
|-------|----------|------------|
| **Data stores** | `empirestate/STANDARDS.md` (§ Data Stores), `core-platform-spec/06_SECTION06.md` | SQL, NoSQL, event streams valid. Kafka = event SSOT; RabbitMQ = ephemeral router (example). Right tool for job; no wrong-domain substitution. |
| **Sync model** | `empirestate/LEDGER.md` (§ Sync Model), `core-platform-spec/09_SECTION09.md` | Application-specific. Real-time (central+bus or distributed) or disconnected (local→central or local→distributed). 3-layer model: real-time, delta, reconciliation. |

### Deployment

| Topic | Document | Key points |
|-------|----------|------------|
| **Deployment** | `empirestate/DEPLOYMENT.md`, `core-platform-spec/02_SECTION02.md` | Layered: intent → state-realizing (e.g. Ansible) → deploy ops → Docker → libraries. Ansible is current choice, not fixed. Automation stands alone; determinism drives deployments. |

### Applications

| Topic | Document | Key points |
|-------|----------|------------|
| **App index** | `empirestate/APPLICATIONS.md`, `empirestate/app-documentation/README.md` | One doc per app. Payux and OpenErgo only in roadmap; others reference. |
| **SKU + cart** | `empirestate/app-documentation/SKU-and-Cart.md` | Maps domain catalog to SKUs/prices; basket; invoice to Payux. |
| **Payux** | `empirestate/app-documentation/Payux.md`, `empirestate/BILLING.md` | M1. Billing: invoice in, receipt out. |
| **Payux detailed design** | `design/PAYUX.md` | Implementation handoff: API, ledger, adapters, sequencing. |

### Detailed design (implementation handoffs)

| Topic | Document | Key points |
|-------|----------|------------|
| **Design folder** | `design/README.md` | Commerce + billing stack index; reading order; cross-cutting rules. |
| **Payux** | `design/PAYUX.md` | Invoice → payment → receipt; ledger; Payux API. |
| **SKU management** | `design/SKU-MANAGEMENT.md` | Catalog → SKU + price; `prices/resolve`; no cart or payment. |
| **Shopping cart** | `design/SHOPPING-CART.md` | Basket, invoice build, Payux handoff, receipt → entitlement grant hook. |

---

## Summary: Constraints to Honor

When implementing, respect:

1. **Tenets 1–6 (significance-ordered)** — **Source control from the start (1)**; then **scaffolding (2)**, **lean (3)**, **production deployment completes work (4)**, **iterate on deployed systems (5)**, **billing early (6)** — `empirestate/TENETS.md`. When guidance conflicts, **lower numbers win**.
2. **Accessor interface** — No ORM as primary abstraction; explicit accessors; ORM behind blackbox OK.
3. **Right tool for job** — SQL, NoSQL, event streams; no ideological substitution.
4. **Ledger pattern** — Append-only, immutable, projection-derived. Shared DNA across financial, claims, social, record-based.
5. **Identity** — Logical silos default; physical opt-in; white-list for commuting. Participation-first.
6. **Deployment** — Layered, deterministic, automation stands alone. Ansible current; transport/tool choice is configuration.
7. **Transport agnostic** — OpenErgo, deployment: no single transport or provider required.
8. **Application-specific** — Datastore topology, sync patterns, data stores: chosen per application.
9. **Strong typing, interface-first** — All languages; define contracts before implementation.
10. **Tenets 7–14** — Documentation authority, form, and layering — `empirestate/TENETS.md`.

---

## When Unclear

- **Behavior / requirements** → `empirestate/` aspect docs (LEDGER, IDENTITY, BILLING, OPENERGO, DEPLOYMENT).
- **Implementation detail / contracts** → `core-platform-spec/` sections and `contracts/`, `policies/`.
- **Process / workflow** → `core-documentation/DEVELOPMENT_GUIDE.md`, `CODING_STANDARDS.md`.
- **Iteration order** → `empirestate/ROADMAP.md`, `empirestate/BACKLOG.md`.
