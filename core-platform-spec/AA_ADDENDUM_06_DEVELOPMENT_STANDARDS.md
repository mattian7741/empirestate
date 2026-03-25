# Development Standards — Philosophy, Principles, and Implementation Norms

This document defines **coding and development standards** for distribution to developers. It is **project-independent**: it does not name stacks, resources, platforms, languages, or product features. It sets expectations for quality, structure, and discipline that apply to **any** development effort adjacent to or consistent with the core platform.

**Use:** This addendum is intended as the **first document** provided to any developer before project-specific requirements. It establishes the shared philosophy, design principles, implementation standards, tenets, and constraints against which work will be evaluated.

**Relationship to core documentation:** These standards are consistent with the philosophy and coding sections of the Core Platform Architecture & Engineering Specification. They are restated here in a standalone, project-agnostic form so they can be applied without reference to a particular system or roadmap.

---

## 1. Philosophy

### 1.1 Contracts First

- All boundaries are defined by **explicit, versioned contracts**.
- Boundaries include: API surfaces, message or event schemas, plugin interfaces, worker or handler signatures, error envelopes, and storage or transport adapters.
- **Shared code is not the contract.** Contracts are specified, versioned, and validated at runtime at boundaries. Internal implementations may evolve behind stable contracts.
- Contracts exist independently of transport, deployment model, or technology choice.

### 1.2 Functional Core / Imperative Shell

- **Business logic** is expressed as **deterministic, testable, pure functions** (or equivalent units of behavior).
- **Imperative concerns**—IO, networking, persistence, retries, instrumentation, lifecycle—are owned by **runtimes and adapters**, not by domain logic.
- Domain logic must have **no awareness** of the runtime. It accepts explicit inputs, receives dependencies via parameters or capability objects, and returns deterministic outputs. It avoids direct IO, global state, and hidden side effects.
- The runtime invokes the behavior and manages all external interaction.

### 1.3 Eventual Consistency and Replay Safety

- **Eventual consistency** is a first-class assumption. The system must remain stable under temporary inconsistency.
- Duplicate delivery, reordering, and partial failures are **normal**. Write paths must be **idempotent** or safely retryable. Messages or events should carry sufficient state to support replay, reconciliation, and projection rebuild.
- Do not rely on strict sequencing or immediate read-after-write consistency unless explicitly guaranteed.

### 1.4 Portability and Profile Independence

- Design for **portability**. The architecture must not assume a single deployment topology, always-on distributed infrastructure, or a specific vendor.
- Different operating profiles (e.g. single-node, multi-tenant, hybrid, isolated) are first-class. No profile is a “hack” or degraded mode.
- Equity lives in **contracts and behavior**, not in a particular stack or platform.

### 1.5 Explicit Over Implicit

- Prefer **explicit** over magical, **structured** over ad-hoc, **composable** over rigid hierarchy, **deterministic** over implicit.
- Contractions and dense idioms are acceptable only when they represent established, shared patterns and do not obscure business intent.
- Reflection or dynamic wiring may be used for **wiring, instrumentation, or registration**—not for domain behavior or hidden control flow.

### 1.6 Immutable Artifacts and Reproducibility

- Deployable units are **immutable artifacts**. Builds produce version-locked outputs. No mutable runtime patching. Rollbacks are done by reverting to prior artifacts.
- All changes land in **version control**. No manual-only steps that cannot be reproduced from code and configuration.

---

## 2. Design Principles

### 2.1 Isolation of Concerns

- **Aspects** (e.g. observability, deployment, identity) are cross-cutting and have clear contracts. They must not depend on a specific deployment topology.
- **Subsystems** are black-boxable: accessed via strict contracts, replaceable or disableable, and must not leak implementation details upward.
- Separate **behavior** from **transport and environment**. Behavior can be tested and reasoned about in isolation; transport and environment are composition choices.

### 2.2 Composition Over Inheritance

- Prefer **composition**. Begin with concrete types. Introduce abstract bases or inheritance only when patterns stabilize and duplication is real.
- Avoid taxonomy-based hierarchies. Hierarchy must grow organically and intentionally. Use inheritance for protocol enforcement or removal of unavoidable structural duplication—not for categorization.

### 2.3 Determinism and Idempotency in Design

- All write operations must be **idempotent** or guarded against duplicate execution. Domain functions must tolerate reordering, duplicate execution, and partial failure.
- Idempotency is enforced at **runtime boundaries**. Use idempotency keys or equivalent mechanisms where appropriate.

### 2.4 Boundary Discipline

- At every boundary: validate input, translate exceptions, normalize responses, attach correlation identifiers, enforce schema or version compatibility.
- No raw exceptions, stack traces, or internal details may cross boundaries. Use a **canonical error envelope** with stable error code, safe message, optional structured details, retryable flag, and correlation metadata.

### 2.5 Instrumentation as an Aspect

- Instrumentation (logging, metrics, tracing) must be **attachable without modifying domain logic**. Use decorators, middleware, or a unified interface. Correlation identifiers must propagate across boundaries.
- Instrumentation must not contaminate pure functions or business logic.

---

## 3. Implementation Standards

### 3.1 Strong Typing

- **Static typing is mandatory.** Use a type system in strict or equivalent mode. No implicit “any” or untyped external input at boundaries.
- Types are part of the contract system. They reduce ambiguity, serve as documentation, and enforce contract discipline. Typing is not optional and is not deferred.
- At boundaries, use typed data structures and explicit validation. Prefer discriminated unions or equivalent for domain states.

### 3.2 Module and File Organization

- **One primary public construct per file** (e.g. one public class or one exported module). File names match the primary construct or purpose. Private helpers may exist but remain internal.
- Prefer **flat structures** over deeply nested folder hierarchies. Modules must have explicit public surfaces.

### 3.3 Branching and Control Flow

- **Guard clauses** and early returns. Limit nesting depth. Avoid deep conditionals. Branching must be explicit and shallow.
- Prefer early validation and result accumulation over single-exit tricks. Avoid contrived looping constructs to simulate control flow.

### 3.4 Functions and Helpers

- Every function belongs to a clear category: domain behavior (pure), method on a type that owns a capability, declarative wrapper (e.g. decorator, middleware), explicit reusable library function, or localized lambda.
- **Helpers** that only split large functions without structural clarity are discouraged. Helpers must improve clarity, reduce branching, represent reusable logic, and be clearly scoped.

### 3.5 Data Access

- **No ORM** as the default. Use **explicit data accessors**: explicit queries, parameterized statements only, explicit mapping from storage rows to domain objects. Clear separation between query logic and domain logic.
- Accessors must expose finite, explicit methods; hide storage implementation; be swappable via dependency injection; and avoid mixing business logic with query logic.

### 3.6 Error Handling

- Catch **specific** exceptions whenever possible. Never catch broad exceptions without reclassification. **Translate** exceptions at system boundaries. Log complete diagnostics at root entrypoints.
- All cross-boundary responses use the **canonical error envelope**. Internal exception details must not leak across boundaries.

### 3.7 Async and Concurrency

- Entrypoints are **async-native** where the runtime supports it. Async is for **IO and orchestration**; domain logic remains **synchronous and pure**. Async must not leak into core domain logic.
- Blocking operations must be explicitly isolated. Domain logic must not depend on event-loop semantics.

### 3.8 Reflection and Dynamic Behavior

- Reflection or dynamic wiring is permitted for: automatic handler or plugin registration, dependency injection wiring, schema-driven validation, instrumentation. It must not implement **business rules**, hidden control flow, or undocumented runtime side effects.

---

## 4. Tenets and Constraints

- **Runtime wraps behavior.** The runtime owns lifecycle, transport, retries, idempotency enforcement, and instrumentation. Injected behavior is invoked with arguments derived from message/config/context and must not invoke the runtime or open connections itself.
- **Message or event as contract.** Every unit of work crossing a boundary has a stable, documented shape (payload, routing/identity, scope/correlation). Wire format is a transport concern; the logical contract is stable across transports.
- **Config-driven binding where applicable.** Mapping from incoming payload or context to handler arguments should be declarative (config or schema), not hard-coded in behavior. Keeps behavior transport-agnostic and testable.
- **No mandatory “noise” fields.** Do not require estimates, due dates, or similar fields solely for reporting or process theatre if they invite low-quality or made-up data. Prefer inference, projection, or optional explicit over mandatory guesswork.
- **Rollback and destroy.** Every capability that can be provisioned or deployed must have a defined rollback or teardown that leaves no trace. Rollback must be codified, not manual-only.
- **Single source of truth.** Plans, contracts, and configuration live in version control. Documentation and diagrams are kept alongside code. Decisions are tied to artifacts.

---

## 5. Testing Standards

- **Test behavior, not implementation.** Test contracts at boundaries. Prefer deterministic tests. Avoid mocking core domain logic. Tests must reflect eventual consistency assumptions.
- **Pure logic:** Tested in isolation—no IO, no external state, no framework or environment. Exhaustive branching and edge cases where practical. Pure function tests form the bulk of the suite.
- **Boundaries:** Every system boundary has explicit **contract tests** (schema compliance, envelope correctness, error envelope, version compatibility, correlation propagation). If a contract changes, tests must fail.
- **Idempotency and replay:** Write paths and projection logic must be tested under duplicate delivery, out-of-order, and replay scenarios. No reliance on implicit ordering unless explicitly guaranteed.
- **Errors:** Error handling is part of the contract. Test exception translation, error envelope structure, and that sensitive data does not leak.
- **Isolation:** Tests must not depend on global mutable state, execution order, or prior test artifacts. Parallel execution must be supported.
- **Stability:** Tests must validate invariants and behavior, not incidental formatting or brittle snapshots. Coverage is a means; behavioral correctness is the goal.

---

## 6. Documentation and Process

- **Contracts are documented** independently of implementation. Schema identity, version, and compatibility rules are explicit.
- **Changes** are reviewed and tied to version control. No long-lived divergence between specification and implementation; either update the spec or correct the implementation.
- **Do not bypass defined boundaries** for convenience. Do not introduce implicit behavior across layers. Favor explicit contracts over shared internal assumptions.
- **Mature over obsolete.** When prior implementations conflict with refined strategy or standards, the refined strategy wins. Do not elevate obsolete or immature patterns above current standards.

---

## Summary

These standards emphasize: **contracts first**, **functional core with an imperative shell**, **eventual consistency and idempotency**, **explicit over implicit**, **strong typing and boundary discipline**, **no ORM and explicit data access**, **runtime wraps behavior**, **test behavior and contracts**, and **reproducibility and rollback**. They are project-agnostic and are intended to set a consistent quality bar for any development effort aligned with this philosophy.

---

*This addendum is consistent with the Core Platform Architecture & Engineering Specification (Sections 1, 10, 11, 12 and related addenda) but is standalone and does not reference project-specific requirements, stacks, or features.*
