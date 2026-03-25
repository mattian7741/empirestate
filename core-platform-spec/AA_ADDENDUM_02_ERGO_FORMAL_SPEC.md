# Ergo Formal Specification — Tenets, Principles, and Philosophy

This addendum distils the **invariant core** of Ergo from ~15 years of implementations (MQ Plugins, Begroupd, Simcity, Bento, MDSDK, Ergo, Hypergo, OpenErgo, ServerGo, and related POCs). It complements **Section 5.1 (Ergo Execution Subsystem)** and **AA_ADDENDUM_01 § B.3 (Ergo Subsystem Framing)** by making explicit the tenets, principles, and philosophy that any Ergo-compliant runtime must preserve.

**Reference implementations analysed:** `~/project_base/ref_impl/ergo` (gitprojects/ergo, gitprojects/hypergo, gitprojects/nautilus-ergo, OpenErgo, more_ergo/ergo, ServerGo, and related codebases). **Additional lineage** (begroupd, bento): see **AA_ADDENDUM_05**; that addendum does not amend this spec.

---

## 1. Philosophy (Why Ergo Exists)

- **Business-first development.** The primary emphasis is to eliminate—as much as possible—boilerplate infrastructure common to most software stacks so that developer effort stays on business-oriented logic.
- **Substrate for coordinated microservices.** Ergo is the **substrate** for rapid development of coordinated microservices: it provides the execution and routing layer so that many small, focused units of behavior can be composed without each unit implementing transport, lifecycle, or topology.
- **Inversion of control and procedural injection.** The runtime owns transport, connection, ack/nack, retries, and topology. The developer supplies **behavior** (functions or callables); the runtime **injects** that behavior into the appropriate protocol and lifecycle. Behavior is referenced, not embedded (e.g. `module:function` or `pkg.mod:func`).
- **One artifact, many environments.** The same behavioral unit (function or pipeline) should be executable in multiple environments—e.g. console script, HTTP endpoint, MQ worker, stdio pipeline—without rewriting business logic. Variability is configuration and substrate, not code.

**Developer experience.** Developers are encouraged to focus on building isolated pure functions that are easy to unit test. In most cases they need not know how those functions fit into the larger workflow or pipeline; the runtime and config handle composition. That separation is intentional—it reduces cognitive load by keeping the developer’s surface area to “write a function, declare its contract.”

**Operational surface.** Centralizing lifecycle, transport, retries, DLQ, correlation, and provisioning in the runtime is a positive: one place to mature, document, and operate. After maturation, the operational surface is a strength, not a liability—teams don’t reinvent these concerns per service.

**Relation to serverless/FaaS.** Ergo’s multi-transport, config-driven binding, explicit routing and scope, and substrate abstraction go beyond typical “inject a function, we run it” offerings; the long lineage of refinement reflects that distinction.

These points appear consistently across the lineage (Ergo READMEs, OpenErgo overview, Hypergo/Executor design, and ref-impl code).

---

## 2. Core Tenets (Invariants)

The following are **non-negotiable** for any implementation that claims to be Ergo-compliant.

### 2.1 Runtime Wraps Behavior (Inversion)

- The **runtime** (Invoker/Executor/Orchestrator) owns:
  - Process lifecycle (start, drain, shutdown)
  - Transport (connect, consume, publish, ack/nack)
  - Error handling and dead-letter behaviour
  - Correlation and scope propagation
- **Behavior** is:
  - Injected by reference (e.g. `func: my_module:my_func` or `ref: pkg.mod:func`)
  - Invoked with arguments derived from **message + config** (and optionally context)
  - Pure or generator-shaped: no direct dependency on transport or broker APIs inside the handler
- So: *the runtime invokes the worker function; the worker function does not invoke the runtime.*

### 2.2 Message as First-Class Contract

- Every unit of work is represented by a **message** with a stable, documented shape.
- At minimum, the message carries:
  - **Payload** (e.g. `data` / `body`) — domain data
  - **Routing** (e.g. `key` / `routingkey`) — used for subscription and publication
  - **Scope** (e.g. correlation id, reply_to, tenant/app/principal when present) — for tracing and request-response
- Wire format (JSON, binary, etc.) and transport-specific headers are the concern of the substrate; the **logical** message contract is stable across substrates (AMQP, stdio, HTTP gateway, Azure Service Bus, etc.).

### 2.3 Config-Driven Binding (Payload → Handler Args)

- Mapping from **message + context** to **handler arguments** is **declarative** (config-driven), not hard-coded in the handler.
- Examples:
  - Ergo: `args: { x: data.x, y: data.y }` or `event: data`
  - Hypergo: `input_bindings` / `output_bindings` with deep paths
  - OpenErgo: bindings map manifest/message → args via templates
- Handlers declare parameters (and optionally types); the runtime assembles kwargs from message and config. This keeps handlers transport-agnostic and testable with plain data.

### 2.4 Substrate Abstraction (Transport Independence)

- **Substrate** = the concrete transport and topology (AMQP, Kafka, stdio, HTTP gateway, Azure Service Bus, etc.).
- Worker **behavior** must not depend on the substrate. The same function can run:
  - As an AMQP consumer
  - Behind an HTTP gateway (sync or async)
  - In a stdio pipeline
  - In a single-process event loop
- Substrate is chosen at deploy/run time (namespace, deploy manifest, or CLI). Local vs remote is a **substrate concern**, not an API concern.

### 2.5 Routing Semantics as Data

- **Subscribe** (sub) and **publish** (pub) semantics are expressed as data (topics, routing keys, bindings), not as code inside the handler.
- Broker bindings (e.g. queue ↔ routing key) are derived deterministically from config (e.g. subtopic, pubtopic, or token lists). Application-level filtering remains the source of truth; broker routing is an optimization.
- This enables declarative pipelines, reproducible provisioning, and safe reprovisioning (e.g. replace bindings for `{sys}.{pid}` without external state files).

### 2.6 Idempotency and Ack Policy

- Processing is **replay-safe by contract**: retries are expected; handlers (or the runtime’s use of them) must be idempotent or externally deduplicated (e.g. idempotency keys at the boundary).
- Ack policy is explicit: e.g. ack only on **total success** by default; optional policies (always, never) for special cases. No partial acks that leave “half-processed” messages.

### 2.7 Failure Isolation and Error Propagation

- Exceptions are captured at the runtime boundary (first decorator or invoker), not swallowed. Errors are attached to the message (e.g. `message.error`) and can be published to error queues or sinks.
- Failure handling (DLQ, retry, logging) is a runtime/decorator concern. Handlers may raise; the runtime decides whether to nack, republish, or forward to an error sink.

---

## 3. Principles (Design Levers)

These principles appear across the lineage and should guide design of core-ergo and any future Ergo implementations.

- **Universal function injection:** Load behavior by reference; support sync fn, sync gen, async fn, async gen via a single normalized execution shape.
- **Fixed, extensible execution pipeline:** A defined order of operations (e.g. exceptions → substitutions → bindings → execute). Users enable/disable features; they do not reorder them. Predictable semantics.
- **Deep templating with relative scope:** Recursive resolution (e.g. `{{ ... }}`) with relative addressing (e.g. `.`, `..`, `...`) and optional fixpoint convergence. Same templating for manifests and runtime data.
- **Deterministic outbound routing:** Publish key derived from config and/or scope; overridable via template (e.g. `routingkey.*` in context). Cartesian or multi-destination publish where supported.
- **Provisioning without external state:** Plan from manifests; apply idempotently; deterministic queue/naming (e.g. `{sys}.{pid}`). Replace stale bindings; preserve correct ones. No TF-style state files required.
- **Source/sink abstraction:** Transport-agnostic sources (AMQP, stdio, Kafka, HTTP in) and sinks (HTTP gateway, AMQP, files, DB). Same pipeline logic regardless of source/sink pair.
- **Stdio as universal adapter:** Any process with stdin/stdout can be treated as a first-class function (cross-language: Bash, Node, Python), enabling composition without rewriting the pipeline.
- **Behavior–environment split:** Behavior manifest has no environment coupling; deploy manifest binds behavior to transports. CLI or env overrides for ad-hoc routing and debugging.
- **Observability hooks:** Decorator tap points for metrics, tracing, logs, with manifest-driven identifiers (sys, pid, rk). Vendor-neutral telemetry (e.g. OTel-ready).
- **Determinism and reproducibility:** Same manifest runs in stdio and in production transports. No hidden env coupling; parameterization makes variability explicit.

---

## 4. Contract Invariants (Summary)

For interoperability and convergence (e.g. core-ergo with core-platform, homelab, or other runtimes), the following must hold:

| Invariant | Description |
|-----------|-------------|
| **Runtime owns lifecycle and transport** | Worker code does not open connections, ack, or publish; it receives data and returns (or yields) results. |
| **Message contract is stable** | Payload + routing + scope shape is versioned and documented; envelopes align with platform envelope spec where BFF and Ergo meet. |
| **Binding is config-driven** | Message → handler args is defined by config/templates, not by hard-coded paths in the handler. |
| **Substrate is pluggable** | AMQP, stdio, HTTP gateway, Kafka, etc. are interchangeable behind a small set of interfaces (source/sink or invoker). |
| **Ack on success by default** | Replay-safe; no partial ack; idempotency at boundary where required. |
| **Errors surface to runtime** | Handler exceptions are captured and propagated (e.g. to error queue or sink); not swallowed. |
| **Same behavior, many transports** | One referenced function or pipeline can run in multiple execution environments without code change. |

---

## 5. Relationship to Platform Spec

- **Section 5.1** defines the Ergo subsystem’s role in the platform (Kafka SOR, RabbitMQ routing, HTTP gateway, DB writer, substrate variants).
- **Section 10.2** codifies the runtime-wraps-behavior pattern (Ergo inversion).
- **AA_ADDENDUM_01 § B.3** frames Ergo as a black-box subsystem over pluggable substrates and states that local vs remote is a substrate concern.

This addendum makes explicit the **tenets and principles** that all Ergo implementations in the lineage have converged on, so that core-ergo and future work remain aligned with the philosophy while evolving substrates and tooling.

---

*Document derived from analysis of ref_impl/ergo (Ergo, Hypergo, OpenErgo, ServerGo, Nautilus-Ergo, and related projects).*
