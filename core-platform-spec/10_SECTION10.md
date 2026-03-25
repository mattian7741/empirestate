# 10. Coding Standards and Development Strategy

This section defines the mandatory coding philosophy across Python, Node.js/TypeScript, and BYOL components. These rules are architectural constraints, not stylistic preferences.

The objective is structural coherence, low entropy, predictable behavior, and portability across runtimes.

---

## 10.1 Object-Oriented Core with Injected Pure Functions

The platform uses an object-oriented runtime model combined with injected pure behavior.

### Core Principle

- Runtimes are object-oriented.
- Domain logic is expressed as pure functions.
- The runtime wraps and orchestrates behavior.
- Domain logic must have no awareness of the runtime.

### Runtime Responsibilities

Runtimes own:

- IO (network, DB, filesystem)
- Transport (HTTP, Kafka, RMQ, stdio)
- Retries and backoff
- Idempotency enforcement
- Instrumentation hooks
- Error translation at boundaries
- Lifecycle management

### Worker Function Model

Injected behavior must:

- Accept explicit inputs.
- Receive dependencies via parameters or capability objects.
- Return deterministic outputs.
- Avoid direct IO.
- Avoid global state.
- Avoid hidden side effects.

The runtime invokes the worker function and manages all external interaction.

---

## 10.2 Async-First Execution Model

All executable processes are async-native.

### Rules

- Python entrypoints must execute under asyncio.run().
- Node.js entrypoints must use async main functions.
- IO operations must be async where supported.
- Domain logic remains synchronous and pure.

### Constraints

- Async must not leak into core domain logic.
- Pure functions must not depend on event loop behavior.
- Blocking operations must be explicitly isolated.

Async is an orchestration concern, not a domain concern.

---

## 10.3 Composition Over Inheritance

Inheritance is not the default abstraction tool.

### Rules

- Prefer composition.
- Begin with concrete classes.
- Introduce abstract bases only when patterns stabilize.
- Introduce parent classes only to eliminate real duplication.
- Avoid taxonomy-based hierarchies.

### Allowed Use Cases for Inheritance

- Protocol enforcement
- Stable interface contracts
- Removal of unavoidable structural duplication

Hierarchy must grow organically and intentionally.

---

## 10.4 Controlled Reflection and Dynamic Inference

Reflection and dynamic wiring are permitted but constrained.

### Allowed Uses

- Automatic handler registration
- Dependency injection wiring
- Schema-driven validation
- Instrumentation weaving
- Plugin registration

### Prohibited Uses

- Business rule execution via reflection
- Hidden control flow
- Implicit behavior based on naming conventions
- Undocumented runtime side effects

Reflection reduces wiring duplication, not domain complexity.

---

## 10.5 Canonical Idioms and Contractions

Dense expressions are permitted when they represent established and predictable patterns.

### Rules

- Prefer canonical idioms over verbose branching.
- Use concise constructs that reduce branching.
- Avoid clever or obscure syntax.
- Avoid macro-like behavior.
- Avoid unreadable symbolic constructs.

Contractions must be familiar, stable patterns. They must not require reverse engineering.

Business logic must remain obvious.

---

## 10.6 Helper Function Policy

Arbitrary helper functions are discouraged.

Functions must belong to one of the following categories:

1. Worker function (pure domain behavior)
2. Method on a class that owns a capability
3. Declarative wrapper (decorator, middleware, annotation)
4. Explicit reusable library function
5. Inline lambda where appropriate

Helper functions that exist only to split large functions without structural clarity are discouraged.

If a helper exists, it must:
- Improve clarity.
- Reduce branching.
- Represent reusable logic.
- Be clearly scoped.

Inner functions are acceptable when locality is intentional.

---

## 10.7 Branching Discipline and Guard Clauses

Deep nesting is prohibited.

### Rules

- Use guard clauses.
- Limit nesting depth.
- Prefer early validation checks.
- Use result accumulation when single-exit behavior is required.
- Avoid contrived looping constructs to simulate control flow.

Branching must be explicit and shallow.

The goal is fewer branches and clearer invariants.

---

## 10.8 Strong Typing Requirements

Typing is mandatory.

### TypeScript

- Strict mode enabled.
- No implicit any.
- No unchecked external input.
- Discriminated unions preferred for domain states.

### Python

- mypy enforced.
- Explicit type annotations required.
- Typed data structures preferred.
- Avoid untyped dict usage at boundaries.

Types are part of the contract system.

---

## 10.9 Module and File Organization

One public class per file is the default.

### Rules

- Files should expose one primary public construct.
- Private helpers may exist but must remain internal.
- File names must match primary class or module purpose.
- Modules must have explicit public surfaces.

Flat structures are preferred over deeply nested folder hierarchies.

---

## 10.10 Data Access Pattern (No ORM Policy)

ORM frameworks are prohibited.

### Rationale

- Hidden behavior
- Performance unpredictability
- Implicit state tracking
- Abstraction leakage
- Debugging complexity

### Required Pattern

- Explicit data accessor classes.
- Explicit SQL queries.
- Parameterized statements only.
- Mapping from rows to domain objects explicitly coded.
- Clear separation between query logic and domain logic.

Data accessor classes must:

- Expose finite, explicit methods.
- Hide underlying storage implementation.
- Be swappable via dependency injection.
- Avoid mixing business logic with query logic.

Stored procedures are allowed when justified, but must not obscure domain rules.

---

## 10.11 Error Handling Discipline

Errors must be structured and controlled.

### Exception Rules

- Catch specific exceptions whenever possible.
- Never catch broad exceptions without reclassification.
- Translate exceptions at system boundaries.
- Log complete diagnostics at root entrypoints.

### Error Envelope

All cross-boundary responses must use a canonical error envelope.

Error responses must contain:

- Stable error code
- Safe human-readable message
- Optional structured details
- Retryable flag
- Correlation metadata

Internal exception details must not leak across boundaries.

---

## 10.12 Boundary Translation Rules

Boundaries include:

- HTTP endpoints
- Ergo gateway handlers
- Worker runtime wrappers
- stdio bridges
- Plugin interfaces

At boundaries:

- Validate input.
- Translate exceptions.
- Normalize responses.
- Attach correlation identifiers.
- Enforce schema version compatibility.

No raw exceptions or stack traces may cross boundaries.

---

## 10.13 Instrumentation as an Aspect

Instrumentation must be attachable without modifying domain logic.

### Requirements

- Decorators or middleware for timing and tracing.
- Correlation ID propagation.
- Structured log output.
- Metrics emission via unified interface.

Instrumentation must not contaminate pure functions.

---

## 10.14 Determinism and Idempotency in Code

All write operations must:

- Be idempotent.
- Guard against duplicate execution.
- Avoid read-then-write race conditions.
- Avoid reliance on in-memory state.

Domain functions must tolerate:

- Reordering.
- Duplicate execution.
- Partial failure.

Idempotency is enforced at runtime boundaries.

---

## 10.15 Testing Strategy Alignment

Tests must align with architectural philosophy.

### Pure Domain Testing

- Direct invocation of pure worker functions.
- No runtime or IO dependencies.

### Integration Testing

- Runtime integration tests at boundaries.
- Explicit contract tests for envelope compliance.

### Projections and Convergence

- Deterministic tests for projection correctness.
- Replay tests where appropriate.

Testing must reinforce purity and boundary discipline.

---

This coding standard is mandatory across Python, TypeScript/Node.js, and BYOL components. Any deviation must be justified at architectural review level.