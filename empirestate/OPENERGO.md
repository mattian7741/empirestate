# OpenErgo — Role in EmpireStack

OpenErgo is the **substrate** for backend integration in EmpireStack. It provides a single, declarative layer over the following concerns:

| Concern | OpenErgo's Role |
|---------|-----------------|
| **Backend communications** | Source/sink abstraction over transports (AMQP, stdio, Kafka, NATS, HTTP, Azure Service Bus, function call stack); behavior manifests independent of transport. Kafka/RabbitMQ is one example stack—no single transport is required. |
| **Real-time events** | Event-driven producer/consumer pipelines with deterministic routing semantics |
| **Scheduled pipelines** | Run-once and service modes; same manifest for ad-hoc and long-running consumers |
| **HTTP request resolution** | HTTP gateway sink; routing key → URL path mapping; template-driven request/response |
| **Inter-application integration** | Cross-system bridging (e.g., RabbitMQ ↔ NATS) via composable sources and sinks |

## Architecture Principles

- **Behavior–environment split**: Component manifests describe *what*; deployment manifests describe *where* and *how*. No environment coupling in behavior.
- **Fixed execution pipeline**: Decorators in enforced order (@exceptions → @substitutions → @bindings → …). Predictable semantics; users toggle, never reorder.
- **Transport agnostic**: Same component runs locally (stdio) and in production (AMQP, HTTP, Kafka, RabbitMQ, Azure Service Bus, etc.). Deterministic and reproducible. Transport choice is configuration, not code.
- **Idempotence by default**: Ack only on full success. Replay-safe; retries expected.

## Key Mechanisms

- **Universal function injection**: `ref: "pkg.mod:func"` loads code; bindings map manifest/message → args via templates. Stdio adapter treats any stdin/stdout process as a first-class function (Bash, Node.js, Python).
- **Deep templating**: Recursive `{{ … }}` resolution with relative scope (. .. ...). Same engine in manifests and runtime.
- **Routing as data**: `sub`/`pub` tokens; broker bindings derived deterministically; app-level filtering is source of truth.
- **Provisioning without external state**: Plan from manifests; apply idempotently; deterministic queue naming `{sys}.{pid}`.

## Relationship to EmpireStack Aspects

OpenErgo underpins the **event-driven nano services** aspect (see [[OVERVIEW]]). Nano services are implemented as OpenErgo components: declarative manifests, injected functions, and transport-agnostic deployment. Other EmpireStack aspects (distributed datastore, identity engine, billing gateway) will integrate via OpenErgo sinks and sources where backend coordination is required.
