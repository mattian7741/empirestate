## How to Use This Document

This document defines the architectural constraints, contracts, and engineering philosophy of the Core Platform. It is normative. It describes *how the system must behave*, not how any single feature is implemented.

Different roles should use this document differently. The sections below outline how each role should approach it.

---

### 1. Frontend Engineers (Web, Mobile, Desktop)

Primary Sections:
- 7. BFF / Experience Layer
- 8. Frontend Platform
- 9. Synchronization Model
- 10. Coding Standards and Development Strategy
- 11. Contracts and Versioning

Focus:
- Adhere strictly to the SDK contract between frontend and BFF.
- Do not bypass BFF for data access.
- Assume eventual consistency.
- Follow the microfrontend composition rules.
- Treat entitlements and provisioning as platform-level aspects, not local logic.
- Use canonical error and result envelopes.
- Respect async-first runtime rules.

You are responsible for:
- Maintaining clean module boundaries.
- Avoiding direct knowledge of Ergo internals.
- Preserving portability across web, mobile, and desktop.

---

### 2. BFF / Experience API Engineers

Primary Sections:
- 6. Data Architecture
- 7. BFF / Experience Layer
- 9. Synchronization Model
- 10. Coding Standards and Development Strategy
- 11. Contracts and Versioning

Focus:
- Enforce request/response envelopes.
- Translate domain exceptions into canonical error envelopes.
- Distinguish clearly between projection-backed reads and command forwarding.
- Never bypass write convergence rules.
- Maintain stateless service behavior.

You are responsible for:
- Maintaining strict boundary contracts.
- Preserving idempotency and correlation across calls.
- Preventing leakage of internal subsystem details.

---

### 3. Database and Projection Engineers

Primary Sections:
- 5. Subsystems (Ergo Execution)
- 6. Data Architecture
- 9. Synchronization Model
- 10. Coding Standards (No ORM Policy)

Focus:
- Maintain Kafka as the System of Record.
- Implement projection workers that are deterministic and idempotent.
- Keep Postgres as a read model, not a command authority.
- Use explicit SQL through accessor classes.
- Avoid ORMs.

You are responsible for:
- Write convergence correctness.
- Partitioning and isolation guarantees.
- Schema versioning discipline.
- Ensuring projection rebuildability from SOR.

---

### 4. Comms / Ergo Engineers

Primary Sections:
- 5. Subsystems (Ergo Execution)
- 6.1 System of Record
- 7.5 Sync vs Async vs Fire-and-Forget Contracts
- 10. Coding Standards

Focus:
- Maintain substrate abstraction (Kafka, RMQ, local loop, stdio).
- Keep worker functions pure and runtime-wrapped.
- Preserve state-based event discipline.
- Ensure idempotency and retry safety.
- Implement HTTP gateway patterns precisely.

You are responsible for:
- Message lifecycle integrity.
- Substrate independence.
- Stable execution semantics across profiles.

---

### 5. AI Platform Engineers

Primary Sections:
- 5.2 AI Subsystem
- 6. Data Architecture (Vector + Storage)
- 10. Coding Standards
- 11. Contracts and Versioning

Focus:
- Implement model broker abstraction.
- Keep provider-specific logic behind adapters.
- Enforce tenant/app scoping in AI operations.
- Maintain auditability and observability.

You are responsible for:
- Avoiding vendor lock-in.
- Ensuring deterministic behavior where possible.
- Preserving privacy boundaries.

---

### 6. Security Engineers

Primary Sections:
- 3. Cross-Cutting Aspects (Identity, Entitlements, Observability)
- 7. BFF Layer
- 10. Error Handling Strategy
- 11. Contracts and Versioning

Focus:
- Validate scope enforcement at every boundary.
- Ensure correlation IDs and audit trails propagate.
- Enforce specific exception handling and logging discipline.
- Review idempotency and replay safety.

You are responsible for:
- Guarding tenant isolation.
- Validating entitlement enforcement.
- Ensuring no sensitive leakage in error envelopes.

---

### 7. DevOps / Deployment Engineers

Primary Sections:
- 2. Operating Profiles and Deployment Philosophy
- 3. Observability Aspect
- 5. Subsystems
- 6. Data Architecture

Focus:
- Implement Ansible-first infrastructure automation.
- Maintain immutable artifact discipline.
- Ensure consistent deployment across profiles.
- Enforce health checks, versioning, and configuration consistency.
- Maintain observability hooks everywhere.

You are responsible for:
- Profile reproducibility.
- Upgrade and rollback safety.
- Environment isolation guarantees.
- Operational stability.

---

### 8. Platform Architects / Lead Engineers

Primary Sections:
- Entire Document

Focus:
- Preserve philosophical coherence.
- Prevent architectural drift.
- Guard boundary contracts.
- Enforce consistent patterns across languages and subsystems.

You are responsible for:
- Rejecting convenience shortcuts that break invariants.
- Maintaining cross-aspect consistency.
- Deciding when reflection and flexibility should calcify into strict contracts.

---

## General Guidance for All Roles

- Do not bypass defined boundaries for convenience.
- Do not introduce implicit behavior across layers.
- Favor explicit contracts over shared internal assumptions.
- Preserve eventual consistency assumptions.
- Respect idempotency and state-based design.
- Treat every boundary as versioned and auditable.

This document is the authority for architectural decisions. If implementation and specification diverge, the specification must be updated or the implementation corrected.