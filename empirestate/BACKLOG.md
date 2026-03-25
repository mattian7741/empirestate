# Backlog — Iterations and Tasks

An evolving list of iterations with subtasks. Heavily influenced by [[ROADMAP]]. Organized for impact, sequential dependencies, and completeness at each stage.

**Principle ([[TENETS]]):** Iterations obey **1–6** — **(1)** work in VCS from the start, **(2)** architecture fit, **(3)** minimal scope, **(4–5)** deployable, iterative on live systems, **(6)** billing trajectory where applicable. Documentation tasks obey **7–14**.

Use `- [ ]` for pending, `- [x]` for complete.

---

## M1: Payux — Billing Gateway

### Iteration 1.1 — Billing Core (Standalone)
- [ ] Invoice → payment → receipt flow implemented (`amount_due` authoritative; metadata opaque)
- [ ] **Payment integration: able to collect money** (minimal; before products exist)
- [ ] Ledger-backed receipt mint + retrieval; **no** SKU or entitlement interpretation in Payux
- [ ] White-label/skinning support
- [ ] Traditional backend + traditional client (REST or equivalent)
- [ ] **Payux in production** (collect-first capability deployed; standing alone)

### Iteration 1.2 — Deployment of Payux
- [ ] Payux deployed via declarative pipeline
- [ ] Zero-to-operational documented for Payux

---

## M2: Deployment & Core Infra

*Payux drives this. Deployment-first: not all deployments need OpenErgo (e.g., simple webpage + backend).*

### Iteration 2.1 — Declarative Deployment
- [ ] All services defined in version-controlled files
- [ ] Docker images for each component
- [ ] Zero-to-operational on fresh host documented
- [ ] External inputs (credentials, DNS) documented
- [ ] **First production deployment** (Payux or equivalent on live host)

### Iteration 2.2 — Pipeline Automation
- [ ] Provisioning automated
- [ ] Deployment automated
- [ ] Health validation automated
- [ ] Time-to-move measured and baseline established

### Iteration 2.3 — Rebuildability
- [ ] Destroy-and-recreate test passed
- [ ] No hidden dependencies proven

---

## M3: OpenErgo Backbone

### Iteration 3.1 — OpenErgo Core
- [ ] Provision OpenErgo from manifests
- [ ] Verify AMQP source/sink
- [ ] Verify HTTP gateway sink
- [ ] Run-once and service modes operational
- [ ] Function injection (ref) working
- [ ] Template resolution verified

### Iteration 3.2 — Integration Validation
- [ ] RabbitMQ → OpenErgo → HTTP gateway pipeline smoke test
- [ ] Provisioning idempotency validated
- [ ] Error handling and ack policy verified
- [ ] **OpenErgo in production** (deployed and running)

### Iteration 3.3 — Payux OpenErgo Integration
- [ ] Payux receives events from OpenErgo ecosystem
- [ ] Payux compatible with other OpenErgo applications
- [ ] Entitlement provisioning hook for OpenErgo flow

---

## M4: Identity Engine

### Iteration 4.1 — Identity Core
- [ ] User schema (shared, application-agnostic)
- [ ] OTP/magic-link roundtrip
- [ ] Session creation and persistence
- [ ] Token validation endpoint

### Iteration 4.2 — Authorization Layer
- [ ] Roles and permissions modeled
- [ ] Entitlements at decision time
- [ ] Discriminators and partitioning support

---

## M5: Ledger & Distributed Datastore

### Iteration 5.1 — Ledger Protocol
- [ ] Event structure and signing
- [ ] Append-only log semantics
- [ ] Gossip/anti-entropy sync

### Iteration 5.2 — Client Architecture
- [ ] Local client datastore
- [ ] Sync as isolated process
- [ ] Abstraction layer between client and network

### Iteration 5.3 — Adjacent Support
- [ ] Fallback datastore when clients can't combine
- [ ] Churn-handling behavior

---

## M6: Viral Invite Pattern

### Iteration 6.1 — Invite Flow
- [ ] User-initiated invite
- [ ] Shareable link/code
- [ ] Single-purpose notification
- [ ] Bounded communication

### Iteration 6.2 — Operational Rules
- [ ] Data minimization
- [ ] Purpose limitation
- [ ] Recipient control (opt-out)
- [ ] Abuse prevention (rate limits)

---

## M7: Target Expansion

### Iteration 7.1 — Web
- [ ] Desktop web deployment
- [ ] Mobile web deployment

### Iteration 7.2 — Mobile Stores
- [ ] Apple App Store (iPhone, webview)
- [ ] Google Play Store (Android, webview)

### Iteration 7.3 — Desktop & Native
- [ ] Electron deployment
- [ ] Native iOS/Windows (if applicable)

---

## M8: Stable Platform

### Iteration 8.1 — Hardening
- [ ] Observability (metrics, tracing, logs)
- [ ] Operational runbooks
- [ ] Failure recovery validated

### Iteration 8.2 — Documentation
- [ ] Application developer guide (generated from docs)
- [ ] Deployment guide
- [ ] API documentation
