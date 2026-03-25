# 1. Philosophy and Architectural Principles

This section defines the non-negotiable philosophical constraints that govern all design and implementation decisions across the platform. These principles apply across languages, products, subsystems, and deployment profiles.

---

## 1.1 Platform Vision and Operating Modes

The platform is not a single application. It is a foundation capable of supporting multiple products and deployment profiles without architectural compromise.

### Core Vision

- Multiple applications share foundational capabilities.
- Applications may diverge in UX, workflows, and domain logic.
- The foundation must support:
  - Single-tenant lightweight deployments
  - Multi-tenant SaaS deployments
  - Hybrid local/remote deployments
  - Enterprise-isolated deployments
- No deployment profile is considered a “hack” or degraded mode.

### Operating Modes

The system supports materially different operating profiles. These profiles are not forced to be identical architectures scaled up or down. Instead, the system is constructed from aspects and subsystems that can be composed differently per profile.

The architecture must not assume:
- Always-on distributed infrastructure.
- Always-remote services.
- Always-centralized state.

Portability is a design constraint.

---

## 1.2 Aspect-Oriented Architecture Model

The architecture is organized around **aspects**, **products**, and **subsystems**.

### Aspects

Aspects are cross-cutting concerns that apply across all profiles and products. Examples include:

- Observability
- Deployment model
- Identity
- Entitlements
- Provisioning

Aspects:
- Must be pervasive.
- Must have clear contracts.
- Must not depend on specific deployment topology.
- Must be composable.

### Products

Products are deployable, standalone capabilities built on top of the platform (e.g., Billing).

Products:
- Use aspects.
- May depend on subsystems.
- Must not redefine aspects.

### Subsystems

Subsystems are black-boxable components (e.g., Ergo, AI services) that:
- Are accessed via strict contracts.
- May be replaced or disabled.
- Must not leak implementation details upward.

This separation prevents conceptual drift and uncontrolled coupling.

---

## 1.3 Contracts-First Doctrine

All boundaries are defined by explicit, versioned contracts.

A boundary includes:
- API surfaces (BFF endpoints)
- Message schemas (Ergo)
- Plugin interfaces
- SDK interfaces
- Worker function signatures
- Error envelopes
- Storage adapters

### Rules

- Shared code is not the contract.
- Schemas are versioned.
- Backward compatibility must be intentional.
- Runtime validation at boundaries is required.
- Internal implementations may evolve freely behind stable contracts.

Contracts exist independently of transport or deployment model.

---

## 1.4 Functional Core / Imperative Shell

Business logic is expressed as deterministic, testable, pure functions.

Imperative concerns (IO, networking, persistence, retries, instrumentation) are managed by runtimes and adapters.

### Functional Core

- Pure functions.
- No direct IO.
- No global state.
- Deterministic behavior.
- Explicit inputs and outputs.

### Imperative Shell

- Owns lifecycle.
- Owns IO.
- Owns transport.
- Owns retries and backoff.
- Owns instrumentation hooks.
- Owns idempotency enforcement.

This applies across:
- Python Ergo workers
- Node/TS BFF handlers
- BYOL stdio functions
- Frontend logic

---

## 1.5 Idempotency and State-Based Design

The system assumes:
- Duplicate delivery is possible.
- Reordering is possible.
- Partial failures are normal.
- Eventual consistency is the default.

### Idempotency

All write paths must be idempotent or safely retryable.

- Commands must include idempotency keys where appropriate.
- Database writes must not assume single execution.
- Side effects must guard against duplication.

### State-Based Messages

Messages and events should carry sufficient state to:
- Avoid reliance on strict sequencing.
- Enable safe replay.
- Enable reconciliation.
- Enable projection rebuild.

Change-based minimal deltas are discouraged unless sequencing guarantees are explicit.

---

## 1.6 Eventual Consistency as a First-Class Assumption

The platform does not assume immediate global consistency.

Consistency strategies include:

1. Real-time updates (speed over integrity)
2. Delta synchronization (corrective overlay)
3. Reconciliation (authoritative overlay)

The system must remain stable under temporary inconsistency.

Design implications:

- UI must tolerate lag.
- Projections must converge.
- Writes must not depend on immediate read confirmation.
- Race conditions must be preemptively guarded.

---

## 1.7 Immutable Artifacts and Release Philosophy

All deployable units are immutable artifacts.

### Rules

- Builds produce version-locked images or binaries.
- No mutable runtime patching.
- Version skew is resolved through release management, not runtime mixing.
- Rollbacks are done by reverting to prior artifacts.

Immutability:
- Reduces operational ambiguity.
- Supports reproducibility.
- Simplifies debugging.
- Prevents configuration drift.

---

## 1.8 Async-First Runtime Model

All executable entrypoints are async-native.

- Python: `asyncio.run()` at root.
- Node: async `main()` as entrypoint.
- Workers may be synchronous pure functions wrapped by async runtimes.

Rules:

- Async is for IO and orchestration.
- Domain logic remains synchronous and pure.
- No business logic should require event loop semantics.
- Async leakage into core domain logic is prohibited.

---

## 1.9 Strong Typing from Inception

Static typing is mandatory.

- TypeScript in strict mode.
- Python with mypy enforcement.
- Schemas validated at runtime at boundaries.

Types:
- Reduce ambiguity.
- Serve as executable documentation.
- Enforce contract discipline.
- Prevent runtime drift.

Typing is not optional and is not deferred.

---

## 1.10 Organizational Simplicity Over Cleverness

The architecture prefers:

- Explicit over magical.
- Structured over ad-hoc.
- Composable over hierarchical.
- Deterministic over implicit.
- Concise but canonical over verbose or cryptic.

Contractions and dense idioms are acceptable only when:
- They represent established, shared patterns.
- They reduce branching or entropy.
- They do not obscure business intent.

Reflection is permitted:
- For wiring.
- For instrumentation.
- For registration.
- During early convergence.

Reflection must not implement domain behavior.

---

## 1.11 Boundary Error Discipline

Errors must be:

- Specific at origin.
- Translated at boundary.
- Logged at root.
- Sanitized across contracts.

A canonical error envelope governs cross-boundary communication.

Exceptions must not leak across system boundaries without translation.

---

## 1.12 Organic Hierarchy Growth

Class hierarchies:

- Begin concrete.
- Abstract only when patterns emerge.
- Avoid taxonomy for its own sake.
- Prefer composition.
- Use inheritance intentionally.

One public class per file is preferred to preserve structural clarity.

---

This section defines the philosophical constraints that supersede framework choice, language choice, and deployment topology. Any future implementation decision must remain consistent with these principles.