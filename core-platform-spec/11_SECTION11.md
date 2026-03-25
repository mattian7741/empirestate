# 11. Contracts and Versioning

This section defines how contracts are specified, versioned, evolved, and enforced across the platform. Contracts exist independently of implementation and are the foundation of interoperability between products, aspects, subsystems, runtimes, and deployment profiles.

Contracts are mandatory at every boundary.

---

## 11.1 Contract Definition Principles

A contract defines:

- Data shape (schema)
- Semantic meaning
- Valid states
- Error behavior
- Version identity
- Compatibility guarantees

Contracts apply to:

- HTTP APIs (BFF endpoints)
- Ergo message schemas
- Plugin interfaces
- SDK interfaces
- Worker function signatures
- Error envelopes
- Storage adapter interfaces
- Synchronization envelopes

### Non-Negotiable Rules

1. Contracts must be explicit.
2. Contracts must be versioned.
3. Contracts must be validated at runtime at system boundaries.
4. Shared code is not the contract.
5. Contracts must be transport-agnostic.
6. Contracts must be documented independently of implementation.

---

## 11.2 Schema Versioning Strategy

Every externally visible schema must carry a version identifier.

### Version Types

There are two types of versioning:

1. Envelope Version  
   Governs the outer response/request structure.

2. Payload Version  
   Governs the internal data structure.

These may evolve independently.

### Version Identification

Each contract must include:

- version (string or integer)
- namespace (to prevent collision)
- schema identity (logical name)

Example structure (conceptual):

- namespace: identity.user.profile
- version: 2

### Compatibility Rules

Backward-compatible changes:
- Adding optional fields
- Adding new enum values (if consumers tolerate unknown values)
- Expanding payloads without altering semantics

Breaking changes:
- Removing fields
- Renaming fields
- Changing field types
- Changing semantic meaning of a field

Breaking changes require version increment and coexistence strategy.

---

## 11.3 Backward Compatibility Rules

Backward compatibility is not accidental.

The system must explicitly define:

- Which versions are supported
- For how long
- At which boundaries

### Boundary-Specific Rules

BFF HTTP APIs:
- Must support at least N-1 versions when feasible.
- Version may be expressed in path, header, or envelope field.
- Breaking changes require explicit new version routes or negotiation.

Ergo Messages:
- Consumers must tolerate unknown fields.
- Producers must not remove fields without coordinated migration.
- State-based message design is preferred to reduce version fragility.

SDK Interfaces:
- SDK version must match declared contract version.
- Microfrontends must target one SDK version only.
- Shell must not mix incompatible SDK versions in the same artifact.

---

## 11.4 Envelope Versioning

All cross-boundary responses must conform to a canonical envelope.

Envelope versioning governs:

- Success structure
- Error structure
- Meta fields

Envelope version changes only when:

- Structural envelope fields change
- Error format changes
- Required meta fields change

Payload changes alone do not require envelope version change.

---

## 11.5 Error Code Versioning

Error codes are part of the contract.

Rules:

- Error codes are stable identifiers.
- Error messages are not stable identifiers.
- Error codes must be namespaced.
- Error codes must not be reused with different meanings.

Top-level classification is layer-based, not domain-based.

Examples of top-level namespaces:

- AUTH
- VALIDATION
- CONFLICT
- NOT_FOUND
- RATE_LIMIT
- UPSTREAM
- TIMEOUT
- INTERNAL

Domain-specific subcodes may extend these.

---

## 11.6 SDK Versioning and Microfrontend Compatibility

The SDK defines the contract between:

- Shell runtime
- Microfrontends
- BFF client layer

### Rules

- A microfrontend must declare its compatible SDK version.
- A build artifact must contain only one SDK version.
- Runtime mixing of SDK versions is prohibited.
- Version skew is resolved by release management, not runtime negotiation.

Compatibility may be supported via:

- Adapter layers
- Deprecation periods
- Explicit migration plans

---

## 11.7 Plugin and Extension Versioning

Plugins (BFF extensions, microfrontend plugins, AI adapters, etc.) must declare:

- Compatible core version
- Required capabilities
- Required envelope version

Core systems must validate plugin compatibility at startup.

Plugins must not:

- Override core contracts
- Modify envelope structure
- Introduce incompatible error formats

---

## 11.8 Message Contract Discipline (Ergo)

Ergo message contracts must adhere to:

- Immutable schema definitions
- Explicit version tags
- State-based event preference
- Idempotency keys when required

Consumers must:

- Ignore unknown fields
- Validate required fields
- Reject unsupported major versions

Producers must:

- Avoid silent semantic changes
- Avoid removal of required fields without coordinated migration

---

## 11.9 Data Access Contract Versioning

Database accessors are internal contracts but must still observe discipline.

Rules:

- Accessor method signatures are versioned by code, not dynamically.
- SQL shape changes that affect projections must be versioned at projection layer.
- Schema migrations must be versioned and reversible where possible.
- Read models must tolerate historical data shapes during migration windows.

ORMs are prohibited; contracts are expressed via explicit accessor interfaces.

---

## 11.10 Deprecation Policy

Deprecation must be explicit and time-bound.

Each deprecated contract must define:

- Replacement contract
- Sunset timeline
- Compatibility guarantees during transition

Silent removal is prohibited.

---

## 11.11 Contract Testing

Every contract must have automated validation:

- Schema validation tests
- Round-trip serialization tests
- Error envelope validation tests
- Compatibility tests across versions

Integration tests must verify:

- BFF-to-Ergo contract integrity
- Projection consistency under version drift
- Plugin compatibility enforcement

Contracts are not considered stable unless covered by contract-level tests.

---

## 11.12 Canonical Source of Truth

Each contract must have a canonical definition source:

- Schema file
- Interface definition
- Message definition artifact

This artifact must be:

- Version-controlled
- Reviewed
- Traceable to release artifacts

Generated code is permitted but must derive from canonical definitions.

Manual drift between implementation and contract is prohibited.

---

This section defines how stability, evolution, and interoperability are enforced across the platform. All subsystems, products, and aspects must comply with these rules to prevent long-term fragmentation and contract erosion.