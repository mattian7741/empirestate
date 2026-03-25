# 12. Testing Philosophy

This section defines the testing model across all languages, services, products, and deployment profiles. Testing must reinforce the architectural principles defined earlier: contracts-first, functional core, eventual consistency, and strict boundary discipline.

Testing is not an afterthought. It is a structural enforcement mechanism for architectural integrity.

---

## 12.1 Testing Principles

The platform adopts the following universal testing principles:

1. Test behavior, not implementation.
2. Test contracts at boundaries.
3. Prefer deterministic tests.
4. Avoid mocking core domain logic.
5. Avoid testing frameworks instead of system behavior.
6. Tests must reflect eventual consistency assumptions.

Testing is aligned with the functional core / imperative shell model.

---

## 12.2 Pure Function Testing

Pure functions are the primary unit of correctness.

### Scope

Applies to:
- Ergo worker functions
- Domain logic inside BFF
- Validation logic
- Transformation logic
- Entitlement evaluation logic
- Policy rules
- Mapping logic (row to object, schema to DTO)

### Requirements

- No IO in pure function tests.
- No external state.
- No framework dependencies.
- No environment configuration.
- No network.
- No database.

### Expectations

- Exhaustive branching coverage where practical.
- Edge cases explicitly tested.
- Property-based tests encouraged for deterministic logic.
- Tests must assert invariant preservation.

Pure function tests must be fast and numerous. They form the bulk of the test suite.

---

## 12.3 Runtime Integration Tests

Runtime integration tests validate that the imperative shell correctly orchestrates:

- IO adapters
- Async boundaries
- Error translation
- Instrumentation
- Dependency injection
- Message envelope compliance

These tests may involve:

- In-memory database instances
- Local SQLite
- Local object store
- Local message substrate
- Stubbed external services

Rules:

- No real external infrastructure in standard integration tests.
- Deterministic environment configuration.
- Full stack within a process or local container.

These tests validate wiring, not business rules.

---

## 12.4 Boundary Contract Tests

Every system boundary must have explicit contract tests.

Boundaries include:

- BFF endpoints
- Ergo HTTP gateway
- Worker function wrapper boundary
- Plugin registration boundary
- Storage adapters
- AI model broker
- Billing APIs
- StdIO wrapper interface

Contract tests assert:

- Schema compliance
- Envelope correctness
- Error envelope compliance
- Version compatibility
- Required metadata presence
- Correlation ID propagation

If a contract changes, tests must fail.

Contracts are authoritative. Shared code does not substitute for contract validation.

---

## 12.5 Projection and Convergence Tests

Given the system assumes eventual consistency, projection behavior must be validated explicitly.

Tests must cover:

- Idempotent write handling
- Duplicate event handling
- Out-of-order event handling
- Replay safety
- Projection rebuild correctness
- State-based message convergence

Projection tests must validate:

- Deterministic projection from event stream
- Stability under reprocessing
- No reliance on implicit ordering unless partitioned guarantees exist

Projection correctness is critical to system integrity.

---

## 12.6 Write Path Tests

Write paths must be tested under the three gateway patterns:

1. Fire-and-forget
2. Synchronous request-response
3. Async job with status

Tests must validate:

- Proper envelope response
- Correct correlation ID propagation
- Idempotency enforcement
- Error translation rules
- Retry safety

Write path tests must not assume immediate read consistency.

---

## 12.7 Error Handling Tests

Error handling is part of the contract and must be tested explicitly.

Test requirements:

- Specific exception types are raised at origin.
- Boundary translation produces correct error.code.
- Error envelope structure is correct.
- Sensitive data is not leaked in error responses.
- Root logging captures full diagnostic detail.

Tests must assert:

- Retryable flag correctness.
- Error classification stability.
- Sanitization integrity.

Errors are part of the public API surface.

---

## 12.8 Async Behavior Tests

Because the runtime is async-first:

- Async boundaries must be tested for race conditions.
- Cancellation behavior must be tested.
- Timeout behavior must be tested.
- Backpressure scenarios must be tested.
- Resource cleanup must be verified.

Async tests must not rely on sleep-based timing when avoidable.
Use deterministic coordination primitives where possible.

---

## 12.9 Deployment Profile Tests

Deployment profiles must be validated in isolation.

Profiles include:

- Single-host lightweight mode
- Multi-host standard mode
- Hybrid local-remote mode
- Enterprise isolated mode

Each profile must have:

- Smoke tests validating startup
- Health endpoint verification
- Configuration validation
- Service wiring validation

Profile tests ensure no profile becomes a second-class citizen.

---

## 12.10 Database Accessor Tests

Given the no-ORM policy:

- Each accessor method must have explicit tests.
- SQL must be validated against schema expectations.
- Mapping logic must be tested independently.
- Idempotent write semantics must be tested.

Where feasible:

- Use test containers or ephemeral databases.
- Avoid mocking SQL unless validating error paths.

Database tests must reflect real constraints.

---

## 12.11 AI Subsystem Tests

AI components must be tested at three levels:

1. Adapter correctness (provider contract compliance)
2. Vector store correctness (insert, query, namespace isolation)
3. Error propagation and rate-limit handling

AI tests must not rely on live cloud providers.
Providers must be mockable at adapter boundaries.

Local model tests must validate:

- Deterministic invocation
- Timeout handling
- Memory and resource cleanup

---

## 12.12 Observability Tests

Observability is an aspect and must be verified.

Tests must assert:

- Correlation ID propagation across boundaries
- Required log fields present
- Metrics emitted for key operations
- Audit events generated for sensitive operations

Instrumentation must not alter business behavior.

---

## 12.13 Test Isolation Rules

Tests must not:

- Depend on global mutable state.
- Depend on execution order.
- Depend on prior test artifacts.
- Leak environment configuration.

Parallel test execution must be supported.

---

## 12.14 Test Coverage Philosophy

Coverage percentage is not a goal.
Behavioral correctness is the goal.

However:

- Core domain logic should approach full branch coverage.
- Boundary contracts must be exhaustively tested.
- Projection logic must be thoroughly validated.
- Error handling paths must be explicitly exercised.

Low-value tests (e.g., trivial getters/setters) are discouraged.

---

## 12.15 Test Organization

Tests must mirror module structure.

For each module:

- Pure function tests
- Integration tests
- Contract tests
- Error tests

Tests must remain close to the code they validate.

---

## 12.16 Stability Over Fragility

Tests must not be brittle.

Avoid:

- Snapshot tests for complex dynamic payloads
- Tests that depend on log formatting beyond contract
- Tests that assert incidental behavior

Tests should validate invariants, not incidental formatting.

---

This testing philosophy ensures architectural integrity, enforces contract discipline, and supports eventual consistency across deployment profiles and runtime substrates.