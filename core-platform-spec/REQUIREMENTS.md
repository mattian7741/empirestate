# Core Platform Architecture & Engineering Specification
## Dual Project Plan: Core Platform + Hot Potato (Flagship App)

This plan defines:
- Independent iteration tracks
- Clear prerequisites
- Small, shippable milestones
- Strict dependency mapping

**Relationship to EmpireStack roadmap:** Billing (Payux) maintains first position in the platform strategy—first-order gateway, narrowly scoped, good MVP. This document centers on Hot Potato as the flagship app that consumes the platform. Hot Potato was conceived to be developed in flight and manage its own backlog (self-fulfilling feedback loop). If Hot Potato keeps high priority here, it must be pared down to its simplest workable form—a strict MVP that handles simple backlog for this and other projects. The full Hot Potato vision is sophisticated; pursuing it at full scope would block all other progress.

Hot Potato is bundled with the platform, but platform evolution must not be dictated by Hot Potato shortcuts.

**Convergence with formal addenda:** The iteration plan is aligned with **AA_ADDENDUM_02 (Ergo Formal Spec)** and **AA_ADDENDUM_03 (Hot Potato Formal Spec)**. Those addenda state tenets, principles, and philosophy; this document states the ordered delivery plan. Implementations should satisfy both.

---

# Repository Layout

Platform and app code live in separate repos; contracts and iteration plans stay aligned.

| Repo | Purpose |
|------|--------|
| **core-platform-spec** | Normative specification and iteration plan (this doc). |
| **core-platform** | BFF/frontend foundation: contracts (envelope, logging), TypeScript core, shared docs. |
| **core-ergo** | Ergo execution subsystem: Python worker runtime, envelope/logging (Python), gateway and SOR integration (later iterations). |
| **hotpotatoai** | Hot Potato (flagship app); depends on core-platform and core-ergo. |

core-platform holds canonical contract definitions; core-ergo implements them for the worker side. Both conform to core-platform-spec.

---

# CORE PLATFORM — Iteration Plan

## CP-0 — Engineering Baseline
- [x] Done

Goal: enforce architectural doctrine before feature work.

Deliverables:
- Repo structure (platform core, apps, shared contracts) — **core-platform**; Ergo worker runtime in **core-ergo**.
- Strong typing enabled (TS strict in core-platform; mypy strict in core-ergo)
- Async-first runtime pattern template (Node in core-platform; Python in core-ergo)
- Canonical error envelope spec
- Structured logging + correlation ID standard
- One-public-class-per-file rule codified
- No-ORM policy documented

Unlocks:
Everything.

---

## CP-1 — Minimal BFF + SSR Shell
- [x] Done

Goal: working web app infrastructure without domain logic.

Deliverables:
- SSR web shell
- BFF with specialized REST dialect
- Request/response envelope implemented
- Basic auth placeholder (principal in context)
- Immutable container build
- Ansible deploy (single host)

Unlocks:
Hot Potato HP-1

---

## CP-2 — Multi-Tenancy + Identity + Provisioning
- [x] Done

Goal: tenant-scoped principals + shadow principal support.

Deliverables:
- Tenant model (consumer-first default)
- Principal model (email-based, shadow capable)
- Role model (2 roles)
- Provisioning flow
- Correlated audit events

Unlocks:
HP-2, HP-3

---

## CP-3 — Data Access Layer + Projection Model (No Ergo)
- [x] Done

Goal: stable read/write abstraction before async complexity.

Deliverables:
- Postgres integration (explicit accessors)
- Projection schema pattern
- Idempotency key model
- Migration mechanism
- Projection read endpoints

Unlocks:
HP-1 (basic tasks), HP-2

---

## CP-4 — Ergo Gateway (Async 202 Only)
- [ ] Done

Goal: introduce async write path safely.

Deliverables:
- BFF → Ergo HTTP gateway adapter
- 202 job response contract
- Job status endpoint
- Minimal worker that writes projection
- Replay-safe idempotent DB writer

Unlocks:
HP-4

---

## CP-5 — Full Write Convergence (Kafka SOR + Workers)
- [ ] Done

Goal: real event-driven persistence.

Deliverables:
- Kafka SOR active
- Worker consumption
- Projection rebuild tool
- Partitioning model (tenant-aware)
- Replay consistency verified

Unlocks:
HP-5

---

## CP-6 — Email/SMS Ingress Adapters
- [ ] Done

Goal: channel-agnostic command ingestion.

Deliverables:
- Email webhook adapter
- SMS webhook adapter
- Capability token generation system
- Token verification + expiry logic
- Inbound envelope normalization

Unlocks:
HP-3, HP-4

---

## CP-7 — Microfrontend Composition
- [ ] Done

Goal: multiple MFEs inside one immutable image.

Deliverables:
- MFE manifest contract
- Shared frontend SDK
- Entitlement gating in UI
- Independent MFE build + compose

Unlocks:
HP-6+

---

## CP-8 — Real-Time + Delta Sync Channels
- [ ] Done

Goal: platform-level synchronization model.

Deliverables:
- Real-time push channel (SSE/WS)
- Delta fetch endpoint with watermark
- Reconciliation endpoint
- Projection watermarking

Unlocks:
HP-7

---

## CP-9 — AI Broker (Minimal)
- [ ] Done

Goal: platform AI capability.

Deliverables:
- Model broker abstraction
- One cloud provider
- Ollama local provider
- pgvector integration

Unlocks:
HP-8

---

## CP-10 — Hardening & Observability Completion
- [ ] Done

Goal: production-level reliability.

Deliverables:
- Audit log export
- Structured metrics
- Replay stress tests
- Token abuse protections
- Rate limiting

Unlocks:
HP scale and enterprise path

---

# HOT POTATO — Iteration Plan (Flagship App)

Each iteration lists required Core Platform prerequisites.

---

## HP-1 — Web Task Creation (Existing Users Only)
- [ ] Done

Prerequisites: CP-1, CP-3

Deliverables:
- Create task (web)
- Complete task (web)
- Task projection read
- Basic dashboard

No email yet.

---

## HP-2 — Hierarchical Tasks + Strategic View
- [ ] Done

Prerequisites: CP-2, CP-3

Deliverables:
- Parent/child task graph
- Deadline support (per HP formal spec: projected delivery and/or optional lock date—not mandatory user estimates)
- Tree-based strategic UI
- Projection queries for hierarchy

---

## HP-3 — Email Assignment (Shadow Principal Creation)
- [ ] Done

Prerequisites: CP-2, CP-3, CP-6

Deliverables:
- Assign task to email
- Shadow principal creation
- Email invite sent
- Invite recorded in projection

No action links yet.

---

## HP-4 — Action Links (One-Click + Confirm Page)
- [ ] Done

Prerequisites: CP-4, CP-6

Deliverables:
- Capability tokens
- One-click action endpoint
- Confirm-page action endpoint
- Idempotent action handling
- Status projection update

---

## HP-5 — Async Write + Projection Convergence
- [ ] Done

Prerequisites: CP-5

Deliverables:
- All task writes routed through Ergo
- Projection convergence verified
- Replay rebuild tested
- Status timeline view

---

## HP-6 — Tactical "Next Best Task"
- [ ] Done

Prerequisites: CP-7

Deliverables:
- Simple deterministic prioritization algorithm
- Tactical view UI
- Real-time projection updates
- Delegation status impact on priority

---

## HP-7 — Real-Time + Delta Sync Hardening
- [ ] Done

Prerequisites: CP-8

Deliverables:
- Web live updates
- Delta reconciliation on reload
- Drift detection logic
- Multi-tab consistency

---

## HP-8 — AI-Assisted Prioritization (Optional Enhancement)
- [ ] Done

Prerequisites: CP-9

Deliverables:
- AI-based scoring model
- Model broker usage
- Feature-gated enhancement
- Fallback to deterministic algorithm

---

## HP-9 — Viral Loop Optimization
- [ ] Done

Prerequisites: CP-10

Deliverables:
- Token expiry tuning
- Abuse/rate controls
- Invite friction reduction
- Minimal onboarding UX
- Growth instrumentation

---

# Structural Discipline Rule

Hot Potato may not:
- Access DB directly
- Bypass BFF
- Bypass gateway for writes (after CP-5)
- Implement its own identity model
- Circumvent entitlement checks

If it needs a capability, it must be unlocked in Core Platform first.

---

# Convergence Outcome

When both tracks complete:

Platform:
- Multi-tenant
- Event-driven
- Async-safe
- Channel-agnostic
- AI-capable
- Microfrontend-based
- Deployment-agnostic

Hot Potato:
- Viral consumer product
- Email-driven delegation engine
- Event-driven task orchestration system
- Real-time + eventual consistency aligned
- Platform-native flagship

---
