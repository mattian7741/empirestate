# 6. Data Architecture

This section defines how data is captured, stored, projected, and accessed across the platform.  
It formalizes the separation between system-of-record, projections, and read/query surfaces.

**Data store selection:** SQL, NoSQL, and event streams are all valid depending on application. Under OpenErgo: Kafka (or similar) is the event convergence center / SSOT for durable events; RabbitMQ (or similar) is the ephemeral router. SQL and NoSQL serve at-rest data stores; NoSQL can also play a role in event management adjacent to streams. Use the right tool for the domain—do not substitute NoSQL where SQL thrives, or vice versa.

The data architecture assumes:
- Eventual consistency
- Idempotent writes
- State-based events
- Explicit contracts
- No ORM as primary abstraction (see policies/NO_ORM_POLICY)

---

## 6.1 System of Record (Kafka)

Kafka (or similar) is the authoritative System of Record (SOR) for durable events in the centralized/OpenErgo profile.

### Purpose

- Capture all durable domain events.
- Serve as replayable tape.
- Enable projection rebuilding.
- Enable audit reconstruction.
- Enable reconciliation.

Kafka is not:
- A query store.
- A direct read surface for UI.
- A replacement for relational projections.

### Design Rules

- All durable writes that affect domain state should result in an event written to Kafka.
- Events must be idempotent or safely replayable.
- Events must carry sufficient state to allow reconstruction without implicit ordering assumptions.
- Event schemas are versioned.
- Partitioning strategy must align with tenant/app scoping to preserve deterministic processing where required.

Kafka is treated as immutable append-only tape.

---

## 6.2 Projection and Read Model Strategy

Projections are derived views of domain state built from Kafka events.

### Responsibilities

- Translate event streams into query-optimized relational models.
- Maintain state snapshots per entity.
- Support read-heavy workloads efficiently.
- Allow rebuild from SOR when required.

### Processing Model

- Ergo workers consume Kafka events.
- Workers update Postgres projections.
- Writes to projections must be idempotent.
- Projections must tolerate replay and duplication.

Projections are not authoritative. They are derived.

---

## 6.3 Postgres as Primary Query Store

Postgres is the primary relational query store for this profile. NoSQL stores (document, key-value) are valid for at-rest data where they fit the domain; do not substitute one for the other based on ideology.

### Responsibilities

- Serve all standard read paths.
- Support multi-tenant schemas.
- Support transactional integrity for projection writes.
- Support JSONB for semi-structured data.
- Support pgvector for AI embeddings where applicable.

### Design Constraints

- No ORMs.
- All access via explicit data accessor classes.
- SQL must be parameterized.
- Queries must be finite and explicit.
- Stored procedures may be used when justified (performance, governance, atomicity).

Postgres is optimized for:

- Relational queries
- Indexed lookups
- Projection state
- Reporting queries
- Entity snapshots

---

## 6.4 JSONB as Semi-Structured Store

JSONB is used for semi-structured or evolving schemas.

### Use Cases

- Flexible metadata
- Version-tolerant document fragments
- Configuration blobs
- AI-related payloads
- Sparse attributes

### Constraints

- JSONB must not replace relational modeling where relational structure is stable.
- JSONB fields must be indexed intentionally.
- Access patterns must be explicit.
- JSONB is still subject to version discipline.

JSONB is a controlled escape hatch, not a schema avoidance strategy.

---

## 6.5 SQLite and Lightweight Modes

SQLite supports:

- POC deployments
- Single-tenant offline modes
- Embedded desktop/mobile modes
- Self-contained test environments

### Rules

- SQLite implementations must conform to the same accessor interface as Postgres.
- No SQLite-specific domain logic.
- Projection behavior must remain consistent across stores.
- Schema migrations must exist for SQLite as well.

SQLite is a deployment profile, not a separate architecture.

---

## 6.6 Event Streams as Tape

Event streams represent append-only historical records.

### Properties

- Immutable.
- Replayable.
- Partitioned.
- Durable.

### Usage

- Capture domain writes.
- Capture user activity streams.
- Capture audit-level events.
- Enable rebuild of projections.
- Enable reconciliation.

Event streams are not optimized for query-by-UI.

Kafka is the canonical tape. Postgres event tables may exist in lightweight modes but are not primary SOR in distributed profiles.

---

## 6.7 Large Document Strategy

Large documents are separated into:

- Metadata (stored in Postgres)
- Content (stored in object storage)

### Object Storage Contract

All large binary content must use an S3-compatible abstraction.

Supported backends:
- Local filesystem
- LAN/NAS storage
- MinIO
- Wasabi
- Other S3-compatible providers

### Rules

- Object keys are deterministic and namespaced.
- Metadata must track version, checksum, size, and ownership.
- Access control enforced at BFF or service boundary.
- Large documents must not be stored in Postgres blobs.

Large content is externalized to preserve DB performance and portability.

---

## 6.8 No ORM Policy and Data Accessor Pattern

ORM frameworks are prohibited.

### Rationale

- Hidden performance characteristics
- Implicit query generation
- Difficult debugging
- Runtime abstraction leakage
- Cross-language inconsistency

### Data Access Pattern

Each datastore interaction is encapsulated in a data accessor class.

Rules:

- One accessor class per logical domain area.
- Explicit method names describing intent.
- SQL defined explicitly.
- Mapping from rows to domain structures is explicit.
- No automatic lazy loading.
- No implicit relationships.

Example structure (conceptual):
```
UserAccessor
	•	get_user_by_id(…)
	•	get_users_by_tenant(…)
	•	insert_user(…)
	•	update_user_state(…)
```

Accessors are injected via capability interfaces.

If datastore changes:
- Replace accessor implementation.
- Preserve contract.
- No domain rewrite required.

---

## 6.9 Write Convergence Model

All domain writes follow a convergence pattern:

1. Command enters system (via BFF -> Ergo gateway).
2. Kafka records event.
3. Worker processes event.
4. Projection updated in Postgres.
5. Read surfaces reflect new state.

Direct DB writes are allowed only when:

- They do not affect authoritative domain state.
- They are strictly local or ephemeral.
- They do not violate event-sourcing assumptions.

Projection writes must be:

- Idempotent
- Safe under replay
- Safe under reordering within partition guarantees

---

## 6.10 Multi-Tenancy in Data Layer

Multi-tenancy is supported at:

- Event partitioning level
- Projection schema level
- Object storage key namespace
- Accessor filtering level

Rules:

- Tenant/app scope must be explicit in all data operations.
- No implicit global queries.
- Partitioning strategy must align with tenant isolation goals.
- Enterprise isolated mode may physically isolate stores per tenant.

Isolation is a design requirement, not an afterthought.

---

## 6.11 Rebuild and Reconciliation Strategy

The system must support:

- Full projection rebuild from Kafka.
- Delta reconciliation.
- Integrity checks.
- Snapshot overlay.

Projection rebuilding must not require application rewrite.

Data architecture must assume that:

- Drift can occur.
- Events can be replayed.
- Projections may be rebuilt.
- State must converge without manual patching.

---

This data architecture supports:

- Distributed event-driven workflows
- Local execution modes
- Portable deployment
- Explicit control over data access
- Deterministic behavior under eventual consistency
- Absence of hidden persistence abstractions