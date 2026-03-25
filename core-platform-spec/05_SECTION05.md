# 5. Subsystems (Black-Boxable Components)

Subsystems are major functional components that can be treated as black boxes behind strict contracts. They may be enabled, disabled, replaced, or deployed differently across operating profiles without altering the higher-level platform contracts.

Subsystems must:

- Expose explicit, versioned interfaces.
- Avoid leaking internal implementation details.
- Respect cross-cutting aspects (observability, identity, entitlements, deployment).
- Remain transport-agnostic wherever possible.

Subsystems are not aspects and are not products. They are replaceable engines behind stable boundaries.

---

# 5.1 Ergo Execution Subsystem

The Ergo subsystem is the distributed execution and orchestration engine of the platform.

It is not embedded into application code as a traditional SDK dependency. Instead, Ergo is a runtime that wraps injected behavior.

**Transport:** Kafka/RabbitMQ is one example stack. Ergo runs on a variety of transports: RabbitMQ, Kafka, Azure Service Bus, HTTP/REST, STDIO, function call stack. Transport choice is configuration, not a fixed requirement.

## 5.1.1 Core Responsibilities

- Durable event sourcing (Kafka or similar as System of Record in centralized profile)
- Message routing (RabbitMQ or similar as ephemeral router)
- Workflow orchestration
- Distributed functional execution
- Idempotent command handling
- Projection triggering

Ergo does not expose its internal topology to the application layer.

---

## 5.1.2 Message Lifecycle

Canonical flow:

```
Client -> BFF -> HTTP Gateway -> Kafka (SOR) -> Router -> Worker -> Kafka -> …
```

Key components:

- Kafka: authoritative event store
- RabbitMQ: transient routing fabric
- Worker runtimes: injectable pure functions wrapped by Ergo runtime
- DB Writer: convergence component responsible for writing projections

Rules:

- Kafka is the authoritative log.
- RabbitMQ does not hold durable truth.
- Workers are substrate-independent.
- Events are idempotent and state-based.

---

## 5.1.3 HTTP Gateway Patterns

The HTTP Gateway is the bridge between request-response systems and the Ergo network.

Supported patterns:

### 1. Fire-and-Forget

- Returns HTTP 200 on receipt.
- Guarantees message accepted into system.
- No guarantee of completion.
- Suitable for broadcast or non-critical operations.

### 2. Synchronous Resolution

- Gateway publishes command.
- Subscribes for correlated response.
- Returns HTTP 200 with result.
- Bounded by SLA and timeout rules.

### 3. Asynchronous Job

- Returns HTTP 202.
- Provides job identifier.
- Client checks status or observes projections.

All patterns must include:

- Correlation ID
- Scope envelope (tenant/app/principal)
- Version metadata

---

## 5.1.4 Write Convergence and DB Writer Model

Writes typically follow this model:

1. BFF issues command to Ergo.
2. Command is persisted to Kafka.
3. Workers process event.
4. DB Writer writes to Postgres projection.
5. Read models converge.

Rules:

- DB Writer must be idempotent.
- Ordering is enforced at partition level.
- No projection may assume exactly-once delivery.
- Read models are reconstructable from Kafka.

Reads never require Ergo mediation.

---

## 5.1.5 Substrate Variants

Ergo supports multiple communication substrates while preserving the same SDK contract.

Supported modes:

1. Local worker -> remote Kafka/RabbitMQ
2. Fully local Kafka/RabbitMQ
3. Pure stdio pipeline (sequential)
4. Single-process event loop with in-memory queue

The abstraction isolates the transport layer from business logic.

Worker functions remain transport-agnostic.

---

# 5.2 AI Subsystem

The AI subsystem provides model execution, embeddings, and related services behind a unified contract.

It must:

- Support multiple providers.
- Support local and cloud execution.
- Be tenant-aware.
- Be observable and auditable.

---

## 5.2.1 Model Broker

The Model Broker abstracts:

- Chat/completion models
- Tool invocation
- Embedding generation
- Moderation/safety checks

Supported backends include:

- Cloud providers (OpenAI, Anthropic, Google, etc.)
- Local providers (Ollama)
- LAN-hosted model services

Applications must not call vendors directly.

---

## 5.2.2 Vector Store Abstraction

Vector operations must be abstracted behind a contract:

- upsert(namespace, vectors)
- query(namespace, embedding)
- delete(namespace, ids)

Default backend:

- Postgres + pgvector

Alternative backends may be added later (e.g., dedicated vector DBs).

Namespace must include tenant and app scope.

---

## 5.2.3 STT / TTS Services

Speech-to-text and text-to-speech services must be adapter-driven.

- Whisper (local or hosted) as default STT baseline.
- TTS through provider abstraction.

No application may bind directly to a specific engine.

---

# 5.3 Object Storage Abstraction

Object storage is a subsystem behind a unified S3-compatible contract.

Responsibilities:

- Store large binary artifacts
- Store document bodies
- Support content-addressable patterns
- Maintain tenant isolation

---

## 5.3.1 Contract Requirements

Operations must include:

- put(object_key, stream)
- get(object_key)
- delete(object_key)
- list(prefix)

Keys must encode:

- tenant scope
- app domain
- version or content hash where applicable

---

## 5.3.2 Supported Backends

- Local filesystem
- LAN/NAS storage
- MinIO (self-host S3)
- Wasabi or other S3-compatible providers

The object storage contract must remain stable across backends.

---

# 5.4 BFF Subsystem (Experience Broker)

The BFF may be bundled with applications or deployed independently.

It is responsible for:

- Enforcing scope
- Translating request envelopes
- Serving projection-backed reads
- Forwarding commands to Ergo when required
- Returning canonical success/error envelopes

The BFF:

- Must not leak internal service topology.
- Must not bypass entitlements.
- Must enforce observability standards.
- May support plugin extensions under strict contracts.

---

# 5.5 Subsystem Governance Rules

All subsystems must:

1. Respect scope envelope (tenant/app/principal).
2. Emit structured logs with correlation.
3. Support versioned contracts.
4. Avoid embedding business logic into transport layers.
5. Remain independently testable.
6. Support isolation in enterprise deployment profiles.

Subsystem boundaries are stability zones.

Internal implementation may evolve.
External contracts must remain stable.