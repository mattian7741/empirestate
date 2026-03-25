# 9. Synchronization Model

This section defines the canonical data synchronization strategy between clients (web, mobile, desktop) and the backend platform.  
The system assumes eventual consistency and must remain stable under partial failure, delay, duplication, and reordering.

**Application-specific:** Sync patterns are chosen per application. Valid patterns include: **real-time** (central data domain + message bus, or to network of distributed) and **disconnected/isolated** (write to local data domain, sync to central; or sync to distributed network). Patterns are reusable; common infra where they align, implementation diverges only where purpose diverges. See empirestate/LEDGER for the pattern taxonomy.

Synchronization is defined as a layered model composed of three complementary strategies (real-time, delta, reconciliation).

---

## 9.1 Design Goals

The synchronization model must:

- Operate correctly under eventual consistency.
- Tolerate message duplication and reordering.
- Avoid reliance on strict sequencing.
- Support multiple deployment profiles (local, hybrid, SaaS).
- Be transport-agnostic.
- Function across all client targets (web, mobile, desktop).
- Degrade gracefully when real-time channels are unavailable.

Synchronization is a platform capability, not an app-specific invention.

---

## 9.2 Core Principles

### 9.2.1 State Over Change

State-based synchronization is authoritative.  
Change-based events may be used for speed, but state is the source of correction.

### 9.2.2 Idempotency

All synchronization operations must be safe to retry.

### 9.2.3 Transport Independence

Synchronization semantics must not depend on:

- WebSocket
- SSE
- Polling
- HTTP long-poll
- Local in-process events

Transport may vary by profile. Semantics may not.

### 9.2.4 Convergence Over Speed

The system is considered correct when it converges.  
Real-time speed is secondary.

---

## 9.3 Synchronization Layers

Synchronization consists of three layers:

1. Real-Time Channel (speed)
2. Delta Channel (corrective overlay)
3. Reconciliation Channel (authoritative convergence)

Each layer has a distinct purpose.

---

## 9.4 Real-Time Channel (Speed Layer)

### Purpose

Provide rapid feedback to the client.

### Characteristics

- Event-driven.
- Not authoritative.
- May arrive out of order.
- May be duplicated.
- May be missed entirely.

### Use Cases

- UI updates after write acceptance.
- Feed updates.
- Notifications.
- Job status hints.
- Presence signals.

### Guarantees

- Best-effort delivery.
- No integrity guarantees.
- Must never be the sole integrity mechanism.

### Transport

May use:

- WebSocket
- Server-Sent Events
- Long polling
- Local event loop
- In-memory pub/sub (local substrate mode)

Transport is implementation detail. Semantics remain constant.

---

## 9.5 Delta Channel (Corrective Overlay)

### Purpose

Correct and confirm state after real-time hints.

### Characteristics

- Query-based.
- Cursor or watermark driven.
- Deterministic.
- Idempotent.

### Client Responsibilities

- Maintain a synchronization cursor or watermark.
- Request updates since last known position.
- Apply returned state deterministically.

### Server Responsibilities

- Accept a cursor.
- Return:
  - Updated entities since cursor.
  - Deleted entities since cursor (if applicable).
  - New cursor.
- Never assume client has full prior context.

### Integrity Model

Delta synchronization ensures:

- Convergence toward correct state.
- Repair of missed real-time events.
- Detection of drift.

Delta is the primary operational synchronization mechanism.

---

## 9.6 Reconciliation Channel (Authoritative Convergence)

### Purpose

Guarantee eventual correctness through full overlay.

### Characteristics

- Full state export or authoritative snapshot.
- May be expensive.
- Used sparingly.

### Trigger Conditions

- Periodic scheduled reconciliation.
- Explicit user refresh.
- Drift detection.
- Deployment transitions.
- Integrity suspicion.

### Modes

Reconciliation may operate as:

- Full snapshot replacement.
- Partition-based reload.
- Checksum comparison followed by selective reload.

### Guarantee

After reconciliation, client state must match authoritative server state.

---

## 9.7 Write and Sync Interaction

When a write occurs:

1. Client submits command via BFF.
2. BFF forwards command to processing layer (Ergo or local path).
3. Write is acknowledged (fire-and-forget, sync, or async).
4. Real-time channel may emit hints.
5. Delta channel confirms state change.
6. Reconciliation guarantees long-term integrity.

The client must never rely solely on the real-time channel for correctness.

---

## 9.8 Job Model and Long-Running Operations

For asynchronous operations:

- A job identifier is returned.
- Job state transitions are observable.
- Job state is projection-backed, not in-memory only.

Job states are:

- queued
- running
- completed
- failed
- cancelled (if supported)

Completion must be reflected in projection state, not only in transient channels.

---

## 9.9 Projection-Backed Synchronization

All authoritative reads originate from projections.

Projections must:

- Be idempotent.
- Be rebuildable from event source of record.
- Support delta queries.
- Support reconciliation exports.

Clients must never query event logs directly.

---

## 9.10 Failure Tolerance

The synchronization model must tolerate:

- Missed real-time events.
- Client restarts.
- Network interruption.
- Partial projection rebuild.
- Duplicate delta requests.
- Delayed processing.

Design must assume:

- Real-time channel may fail silently.
- Delta channel may retry.
- Reconciliation may be necessary.

Correctness must not depend on continuous connectivity.

---

## 9.11 Scope and Isolation

All synchronization operations are scoped by:

- Tenant
- Application domain
- Principal (where applicable)

No cross-tenant leakage is permitted.

Cursors and watermarks must be scoped to tenant and app context.

---

## 9.12 Transport and Profile Variants

Different deployment profiles may implement synchronization differently:

- Local single-process mode may use in-memory events.
- Hybrid mode may use remote Kafka with local projections.
- SaaS mode may use distributed real-time infrastructure.

The synchronization contract must remain consistent across profiles.

Transport differences must not change:

- Envelope structure
- Cursor semantics
- Job model
- Error handling model

---

## 9.13 Observability and Sync

Synchronization events must emit:

- Correlation identifiers.
- Scope identifiers.
- Operation type.
- Latency metrics.

Reconciliation events should emit drift diagnostics when applicable.

---

## 9.14 Security and Integrity

Synchronization must enforce:

- Tenant isolation.
- Entitlement gating.
- Authorization checks at projection access.
- Sanitized error responses.

Delta and reconciliation endpoints must not expose data beyond authorized scope.

---

## 9.15 Summary

Synchronization is:

- Layered.
- Eventually consistent.
- Projection-backed.
- Transport-agnostic.
- Idempotent.
- Resilient to failure.

Real-time provides speed.  
Delta provides correction.  
Reconciliation guarantees integrity.

This model is foundational and applies uniformly across all clients and deployment profiles.