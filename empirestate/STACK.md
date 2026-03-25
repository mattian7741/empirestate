# EmpireStack — Stack

Selected tools and technologies. See [[ROADMAP]] for implementation order.

---

## Applications (In Scope)

| Component | Role |
|-----------|------|
| **Payux** | Billing application. M1 in roadmap. Invoice → payment → receipt (ledger-backed); no SKU semantics. Starts standalone (traditional backend + client); OpenErgo integration when backbone is ready. See [[BILLING]] |
| **OpenErgo** | Backend substrate. M3 in roadmap. Communications, real-time events, pipelines, HTTP resolution, inter-application integration. See [[OPENERGO]] |

---

## Infrastructure & Runtime

| Tool | Role |
|------|------|
| **Docker** | Container runtime; unit of portability. All services run in containers. See [[DEPLOYMENT]] |
| **Python** | Preferred backend language. See [[STANDARDS]] |

---

## OpenErgo Transports

OpenErgo is transport-agnostic. Supported (or planned) transports:

| Transport | Use |
|-----------|-----|
| **AMQP (RabbitMQ)** | Message broker; source/sink |
| **HTTP** | Gateway sink; routing key → URL path |
| **stdio** | Local/dev; any stdin/stdout process |
| **Kafka, NATS** | Alternative message layers |

---

## Standards (Tooling)

| Standard | Requirement |
|----------|-------------|
| **Strong typing** | Mypy (Python), TypeScript (JavaScript) |
| **Database** | Explicit abstractions; no ORM as the abstraction. SQL and NoSQL both valid. |
| **Interface-first** | Contracts before implementation |

---

## Deployment Targets

Web, Apple App Store, Google Play, Electron, native iOS/Windows. See [[DEPLOYMENT]].
