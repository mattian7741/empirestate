# Roadmap — Strategic Milestones

High-level milestones that guide implementation order and maximize the chances of success. See [[BACKLOG]] for the granular iteration list.

**Scope:** This roadmap builds the EmpireStack platform, not the applications that consume it. The applications listed in [[APPLICATIONS]] are **reference only**—they illustrate what the stack enables but are not part of this roadmap. The only applications in scope here are **OpenErgo** (infrastructure/backbone) and **Payux** (billing).

**Priority:** Billing maintains first position. It is the first-order gateway for successful enterprise, narrowly scoped, and a good MVP/POC candidate. Other application tracks (e.g. Hot Potato) either take a back seat or require a strict MVP definition—the full Hot Potato vision is sophisticated and would block progress if pursued at full scope.

**Governing principle ([[TENETS]]):** Milestones obey **1–6** in order of significance — **version control from the outset (1)**, **scaffolding (2)**, **lean scope (3)**, **production deployment as done (4)**, **iterate on what is live (5)**, **billing-capable trajectory (6)**.

---

## M1: Payux — Billing Gateway (Highest Priority)

Payux drives the deployment initiative and delivers universal billing. It is the highest priority because it establishes both revenue capability and deployment discipline from day one.

**Goal:** Payux operational with traditional backend and traditional client—**standing alone**. **Invoice → payment → receipt** (cart/SKU stack may be mocked or minimal). **Able to collect money before we have anything to sell.** White-label ready. In production.

**Interfaces:** Payux starts with traditional REST (or equivalent) backend + client. It can and should have other interfaces. OpenErgo integration—where Payux receives events from the OpenErgo ecosystem—is built when OpenErgo surfaces in the roadmap; that integration makes Payux compatible with other OpenErgo applications.

---

## M2: Deployment & Core Infra

Deployment-first philosophy. Not all deployments depend on OpenErgo—a simple webpage with a simple backend does not. Even components that will ultimately use OpenErgo can start standalone.

**Goal:** Reproducible deployment from repo to any Docker-capable host. Local, test, and production share the same model. Time-to-move validated. Payux deployed via this pipeline. Infrastructure as disposable; declarative; containerized.

**Blocks:** All subsequent components deploy via this pipeline.

---

## M3: OpenErgo Backbone

The backend substrate for communications, events, and HTTP integration. Payux integrates with OpenErgo when this milestone is reached—enabling compatibility with other OpenErgo applications.

**Goal:** OpenErgo operational and integrated with at least one transport (e.g., AMQP, HTTP gateway). Core pipeline (provisioning, consume-once, service modes) verified. **In production**—deployed and running. Payux OpenErgo integration built.

**Blocks:** Ledger sync, future nano services, applications that rely on event-driven backbone.

---

## M4: Identity Engine

Participation-first, roundtrip auth, commuting identity. User, role, permission, entitlements.

**Goal:** Identity service operational. Roundtrip OTP/magic-link auth. Session persistence. Roles and entitlements modeled.

**Blocks:** Viral invites, application user flows.

---

## M5: Ledger & Distributed Datastore

Append-only ledger of claims. Local+sync first. Adjacent datastores as fallback.

**Goal:** Ledger protocol and projection layer. Client local store with sync process. Adjacent support nodes for churn scenarios.

**Blocks:** Applications requiring distributed state.

---

## M6: Viral Invite Pattern

User-initiated transactional invites. Emergent users without sign-up.

**Goal:** Invite flow implemented. Shareable links, bounded communication, operational rules (data minimization, opt-out, abuse prevention).

**Blocks:** Applications requiring viral/share mechanics.

---

## M7: Target Expansion

Deployment across all target platforms.

**Goal:** Web, Apple App Store, Google Play, Electron, and optionally native iOS/Windows. Same deployment principles applied across targets.

---

## M8: Stable Platform

Platform hardening, documentation completeness, operational runbooks.

**Goal:** EmpireStack as a stable product platform. Applications can be built on top with confidence. Support tooling and observability in place.
