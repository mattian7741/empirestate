# Aspects — scaffolding

Five dimensions. Components **slot here** as built.

| Aspect | Role |
|--------|------|
| **Event-driven nano services** | **OpenErgo** — manifests, injected functions, transport-agnostic sources/sinks (AMQP, HTTP, Kafka, stdio, …). |
| **Distributed datastore / ledger** | Append-only, immutable, **projection-derived** truth. **Claims** ledger: multi-writer, no central authority required; contradictions resolved by new events. **Financial** ledger (billing): typically **centralized** DB. Same pattern DNA; **topology varies by product**. |
| **Identity** | Shared user schema, **logical silos by default** (`tenant_id` / discriminators), **physical silos** opt-in. **Roundtrip** identifier proof (OTP / magic link). **Participation-first** — no formal sign-up wall. **Commuting identity** where policy white-lists domains. |
| **Billing gateway** | **Payux:** invoice → payment → **receipt**; ledger-backed; **opaque metadata** only (no SKU/product/entitlement semantics). **White-label SKU + cart** (separate): maps sellables → SKU/price; basket → invoice. **Domain workloads** own products/services/**entitlements**; grants use receipt / billing txn **backreference** (or gift/admin provenance). |
| **Deployment** | **Disposable** hosts; **declarative** layers (intent → state tool e.g. Ansible → ops → **Docker** → libs); zero-to-op on fresh machine + credentials. **Named artifacts** (`xyz`, `staging`, …): authors say *component runs on **`xyz`***; **bindings** resolve names to concrete URLs, brokers, images—Docker/vendor detail is execution, not top-layer vocabulary. |

**Viral invite (pattern, M6):** User A pulls user B into **one** activity; B acts without account; narrow purpose; no list-building.

---

## OpenErgo (summary)

Behavior in manifests; environment in deploy config. Fixed decorator pipeline; **idempotent** ack. Routing as data; deep templating; stdio = any process as function.

## Ledger (summary)

**Claims path:** Events are **speech acts**, not enforced truth; validity ≠ correctness; history monotonic. **Client:** local store + **sync** process. **Billing path:** mint/redeem/refund; centralized store optional partitions.

## Identity (summary)

User = person; **participant** = role in context; **entitlement** = access (authorization time). Identity **not** owned by a single app silo.

## Billing commerce stack (summary)

| Piece | Does |
|-------|------|
| **Payux** | Money: `amount_due` authoritative; receipt mint. |
| **SKU layer** | Catalog keys → SKU + price resolution. |
| **Cart** | Lines, total, invoice call, receipt handoff to domain grant. |

## Deployment (summary)

No lock-in to one cloud; containers portable; state externalized; secrets outside host. **Intent uses names** (e.g. OpenErgo in **logical** namespace **`xyz`**); at deploy time, **environment** + resolver picks the binding row that maps **`xyz`** to **concrete** targets (URLs, creds, …)—swap binding, same ESB sentence. **Empire State Build (ESB):** few human parameters (no per-environment copy); **environment** from pipeline + **namespace** from ESB → resolved binding key (e.g. `<namespace>-<environment>`) → expanded spec → **Ansible** (for now) → Docker/shell/APIs (`empirestate/DEPLOYMENT.md` § Empire State Build).


Expand: `../empirestate/OPENERGO.md`, `LEDGER.md`, `IDENTITY.md`, `BILLING.md`, `DEPLOYMENT.md`, `VIRAL.md`.
