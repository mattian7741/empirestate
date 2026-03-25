# 7. BFF / Experience Layer

The BFF (Backend For Frontend) is the experience-facing server layer that brokers all UI interactions with backend systems. It exists to enforce scope, apply policy, normalize contracts, and protect internal topology.

The BFF is not the system of record.
The BFF is not the orchestration engine.
The BFF is not a projection builder.

The BFF is the experience abstraction boundary.

---

## 7.1 Responsibility Model

The BFF is responsible for:

- Enforcing tenant and app scope
- Enforcing authentication and session semantics
- Enforcing entitlement checks
- Validating request schemas
- Normalizing responses
- Translating errors into canonical envelopes
- Forwarding commands to the Ergo HTTP Gateway when required
- Serving projection-backed reads
- Providing SSR-compatible data access
- Providing consistent behavior across web, mobile, and desktop

The BFF must not:

- Directly manipulate the Kafka SOR
- Implement distributed orchestration logic
- Bypass the DB writer convergence model
- Contain business logic that belongs in domain workers

---

## 7.2 Specialized REST Dialect

The BFF exposes HTTP APIs using a controlled REST dialect.

It is not bound to strict REST dogma.

### Characteristics

- HTTP/JSON transport
- Stable route naming
- Explicit versioning
- Canonical request and response envelopes
- Operation identity independent of URL structure
- Explicit scoping via headers and/or validated tokens
- Structured error codes

The dialect may:

- Use dynamic path segments
- Break strict CRUD semantics where justified
- Return non-resource-based responses
- Align payload structure with Ergo messaging conventions

Transport remains simple (HTTP + JSON) even if semantics evolve.

---

## 7.3 Request Envelope Specification

Every inbound request must resolve into a validated envelope before reaching business logic.

### Envelope Components

- correlation_id
- request_id
- tenant_scope
- app_domain
- principal (authenticated identity)
- timestamp
- schema_version
- operation_id

These values must be:

- Validated
- Logged
- Propagated downstream
- Attached to outbound calls to Ergo or subsystems

Scope enforcement is mandatory at the envelope boundary.

No downstream system should need to infer tenant or app scope.

---

## 7.4 Response Envelope Specification

All responses must use a canonical envelope.

### Success Envelope

- ok: true
- data: payload
- meta:
  - correlation_id
  - request_id
  - timestamp
  - schema_version
  - optional warnings

### Failure Envelope

- ok: false
- error:
  - code (stable, namespaced)
  - message (safe, human-readable)
  - details (structured, optional)
  - retryable (boolean)
- meta:
  - correlation_id
  - request_id
  - timestamp
  - schema_version

HTTP status codes may vary, but the envelope is authoritative.

Exceptions must never cross the BFF boundary unwrapped.

---

## 7.5 Command Execution Patterns

When forwarding to Ergo via the HTTP Gateway, the BFF supports three execution modes.

### 1. Fire-and-Forget

- Gateway returns 200 when message is accepted.
- No guarantee of completion.
- Used for broadcast or non-critical operations.

BFF returns:
- ok: true
- correlation_id
- no result payload

---

### 2. Synchronous Execution

- Gateway publishes message.
- Gateway subscribes for a response.
- Gateway returns final result.

BFF returns:
- ok: true with data
  or
- ok: false with error envelope

Used when bounded latency is acceptable and completion is required.

---

### 3. Asynchronous Job

- Gateway returns 202.
- Message enters Ergo pipeline.
- Completion occurs later.

BFF returns:
- ok: true
- job_id
- initial status

Client retrieves status via:
- Polling endpoint
- Real-time channel (optional)
- Delta synchronization

---

## 7.6 Read Model Strategy

All read operations must:

- Query projection stores directly
- Avoid using Ergo as an intermediary
- Remain stateless
- Be safe under eventual consistency

Read endpoints may access:

- Postgres projections
- JSONB documents
- Object storage metadata
- Cached representations

Read paths must not mutate state.

---

## 7.7 Write Convergence Model

Write requests should:

- Be validated at BFF boundary
- Be forwarded as commands into Ergo
- Avoid direct mutation of projection stores

The DB writer component is responsible for durable state convergence.

Exceptions may exist for strictly local or ephemeral state, but:

- They must not violate eventual consistency guarantees
- They must not bypass idempotency discipline

---

## 7.8 Extension Model

The BFF supports both stock behavior and extension behavior.

### Stock Behavior

Core BFF responsibilities are platform-owned and stable.

Includes:

- Auth
- Scope validation
- Error translation
- Envelope enforcement
- Entitlement gating

### Extension Behavior

Apps may register additional handlers under strict contracts.

Extensions must:

- Declare namespace
- Declare compatible BFF contract version
- Declare required scopes
- Declare required entitlements
- Use canonical request/response envelopes
- Avoid bypassing core enforcement

Extensions may be:

- Bundled in the same immutable release image
- Deployed as separate units under the same contract

---

## 7.9 SSR Compatibility

The BFF must support server-side rendering workflows.

Requirements:

- Data-fetch methods usable in both SSR and CSR
- Deterministic responses
- No client-only assumptions
- Safe handling of authenticated SSR

SSR must not require special-case business logic.

---

## 7.10 Async Runtime Model

The BFF is async-native.

- All handlers execute inside an async runtime.
- IO is awaited.
- Pure domain logic remains synchronous.
- Blocking operations are prohibited.

Async must not leak into business logic unnecessarily.

---

## 7.11 Error Taxonomy

Error codes are grouped by layer, not by product.

Examples:

- AUTH.*
- ENTITLEMENT.*
- VALIDATION.*
- CONFLICT.*
- NOT_FOUND.*
- RATE_LIMIT.*
- UPSTREAM.*
- TIMEOUT.*
- INTERNAL.*

Domain-specific subcodes may exist beneath these namespaces.

Error codes must be stable across versions.

---

## 7.12 Boundary Discipline

The BFF is the enforcement boundary.

At the BFF root:

- Exceptions are caught
- Errors are translated
- Logs are emitted
- Correlation is preserved
- Sensitive information is stripped

The BFF must never leak internal stack traces or subsystem implementation details.

---

This section defines the formal responsibilities, contracts, and behavioral patterns of the experience abstraction layer. All frontend-facing behavior must pass through this boundary.