# Deployment — Target-Agnostic

Infrastructure is **disposable**, not foundational. The system is the codebase and its deployment definition—not the machine it runs on. Any host is a temporary execution target.

**Summary:** Infrastructure is interchangeable, deployment is deterministic, and movement between environments is routine rather than exceptional.

---

## Automation Strategy

**Priorities:** (1) Deployment automation can stand alone. (2) IaC in place; determinism drives deployments. Both are required.

**Layered model:** Deployment is expressed as layers. High-level intent is the top-most declarative layer (e.g. "Deploy X version a.b.c in production with dependencies Y and Z"—each with its own declarative specification). Under that: a deterministic state-realizing solution (e.g. Ansible, current choice; could be displaced under scrutiny). Under that: deploy operations. Under that: Docker containers. Under that: libraries (built and published in a similar automated way). Layers upon layers.

---

## Named artifacts and implementation binding

**Author-facing statements use names, not wiring.** Environments, networks, data planes, and similar **targets** are **named artifacts** in the spec (e.g. `xyz`). A component or manifest at the **highest layer** only asserts a relationship to that name—for example: *OpenErgo components run on network **`xyz`***. No RabbitMQ URLs, no Docker syntax, no vendor API detail at that layer.

**Binding:** Each name resolves to a **technical implementation** stored elsewhere (version-controlled registry, env catalog, or materialized config). Example: `xyz` → *OpenErgo transport is AMQP; broker URL …; vhost …; credential reference …; tuning …*. Replacing *how* `xyz` is realized (different broker, host, or orchestration) updates **the binding for `xyz`**; consumers that say *run on `xyz`* stay unchanged.

**Separation:**

| Layer | Says |
|-------|------|
| **Component / intent** | *Runs on **`xyz`*** (or deploy to **`staging`**, attach store **`ledger-db`**, etc.) |
| **Named artifact + binding** | **`xyz`** is defined by **concrete** endpoints, secrets references, images, playbooks, or generated low-level files |
| **Execution** | Docker, Ansible, cloud APIs, etc.—**means**, not the conceptual definition |

**Goal:** Lowest barrier to entry for authors—**declare affiliation to a stable name**—while keeping **determinism and audit** in the binding layer so deployments remain reproducible.

---

## Empire State Build (ESB)

**Empire State Build (ESB)** is the name of the **lay deployment descriptor grammar**: the smallest human-facing surface that states **what** runs, **where** (in logical terms), and **what it attaches to**—by **name**—while the system **infers** the rest via **named bindings**, conventions, and versioned rules.

**Principle:** A human can describe a deployment with a **very small set of parameters**; the author does not treat Dockerfiles, inventory, or shell as the **source of truth**. Those artifacts are **materialized** to satisfy an end state derived from the ESB.

**ESB document** (authoring layer):

- Short, stable fields: e.g. **what** runs (application / component id + version or digest), **where** in logical terms (**environment** name, **network** name like `xyz`, optional **profile**), and **dependencies** referenced by **name** (other apps, stores, buses).
- The grammar starts **minimal** and **grows only** when a real gap appears; every new field must earn its place (see `TENETS.md`).

**Inference chain** (current stack direction):

```text
ESB (lay descriptor)  →  expanded / normalized spec (machine-readable)  →  state realizer (e.g. Ansible)  →  concrete actions (Docker pull/run, files, systemd, APIs, …)  →  measured end state
```

- **Ansible** (or successor) is the **deterministic state-realizing** layer today: idempotent tasks that close the gap between “what we declared” and “what is running.” It is an **implementation choice**, not the eternal definition—another engine could replace it if it honors the same **expanded spec** contract.
- **Docker, bash, cloud APIs** sit **below** that layer: **means** to converge the host, not the vocabulary the human masters first.

**Relationship to named artifacts:** ESB **references** names (`staging`, `xyz`, `payux:1.2.3`); **bindings** supply the heavy detail; the realizer **consumes** the expanded result and never asks the author for RabbitMQ URLs at the top.

**Non-goal:** A single giant schema on day one. Goal is **grammar discipline**—small surface, clear inference, audit trail from ESB through materialized artifacts.

### Representative ESB examples (illustrative — for review)

The blocks below are **not** a frozen schema. They show the **shape** we want authors to think in: few fields, names instead of wiring, room for inference (*profile*, *depends_on*). Field names and nesting are candidates for your feedback.

**Example A — minimal single component**

A single service version on a logical environment; bindings define URLs, secrets, and image pull specifics.

```yaml
# Illustrative only — field names TBD
esb: "0.1"
component:
  id: payux
  version: "1.2.3"
environment: staging
```

**Example B — logical network / data plane by name**

Same idea as *OpenErgo runs on network `xyz`*: the author names the plane; the binding carries broker URL, vhost, credentials references, tuning.

```yaml
esb: "0.1"
component:
  id: open-ergo-worker
  version: "0.9.1"
environment: production
network: xyz
```

**Example C — named dependencies (stores, buses, sibling apps)**

Dependencies are **references by name**; each name resolves in the binding catalog (connection strings, queues, peer service discovery—not spelled out here).

```yaml
esb: "0.1"
component:
  id: payux
  version: "1.2.3"
environment: staging
depends_on:
  - ledger-store
  - payments-bus
```

**Example D — profile for sizing or tier**

*Profile* is a convention hook: small/medium/large (or product-specific tiers) maps to inferred limits, replicas, or resource classes in the **expanded spec**—still without surfacing Docker or cloud APIs in ESB.

```yaml
esb: "0.1"
component:
  id: payux
  version: "1.2.3"
environment: staging
profile: small
```

**Example E — stack slice (multiple components, shared context)**

Multiple components that share **environment** (and optionally **network**) so the pipeline can expand one coherent slice; each row stays thin.

```yaml
esb: "0.1"
environment: staging
network: xyz
components:
  - { id: payux-api, version: "1.2.3" }
  - { id: payux-worker, version: "1.2.3" }
```

---

## Core Principles

### Target Agnosticism from the Start
No part of the application or deployment depends on a specific provider, managed service, or proprietary runtime. If a component cannot be recreated elsewhere from first principles, it is a liability and is excluded from the core path.

### Fully Declarative
Everything required to run—services, networking, routing, configuration—is expressed in version-controlled files. No manual steps, no hidden state, no reliance on prior machine history. A fresh host must go from zero to fully operational using only the repository and a small set of external inputs (credentials, DNS, etc.).

### Containerization
All services run in Docker containers with well-defined inputs and outputs. The container image is the unit of portability. If it runs locally, it runs unchanged on any target host that supports Docker.

### Separation of Concerns

| Concern | Rule |
|---------|------|
| **Compute** | Ephemeral |
| **State** | Externalized |
| **Configuration** | Injected |
| **Secrets** | Managed outside the host |

The host never owns anything critical. Databases, file storage, and persistent data are external services or use explicit backup/restore. The application tolerates host destruction at any time.

---

## Operational Requirements

### Repeatable, Deterministic Deployment
Provisioning, configuration, deployment, and health validation are automated. The process must succeed on a brand-new machine without modification. If it cannot be run end-to-end repeatedly, it is not complete.

### Continuous Validation of Rebuildability
Regularly destroy and recreate environments from scratch to prove no hidden dependencies exist. A successful deployment is defined by the ability to reproduce the system on demand—not by uptime.

### No Portability-Hiding Abstractions
Provider convenience features are acceptable only if they introduce no lock-in. Prefer explicit, portable mechanisms over implicit platform features.

### Target as Input
Switching from one host to another is a parameter change, not a redesign. The deployment pipeline accepts a new target and produces a working system with minimal friction.

### Simplicity Over Scale
The system must be understandable, inspectable, and debuggable without specialized tooling. Avoid premature complexity (e.g., orchestration platforms) unless strictly necessary.

### Failure and Recovery
Assume any host can disappear. Recovery is redeployment, not repair. The fastest path: provision a new target and redeploy from source.

### Tight Feedback Loop
Local development, test, and production use the same deployment model. Minimize differences between environments to reduce drift.

### Time-to-Move as Core Metric
Success means standing up a new environment, deploying the application, and cutting over traffic in minutes. Anything that slows this is scrutinized and removed.

---

## Deployment Targets

Deployment strategies include (and are not limited to):

| Target | Description |
|--------|-------------|
| **Apple App Store** | iPhone apps (including mobile webview) |
| **Google Play Store** | Android apps (including mobile webview) |
| **Web** | Desktop and mobile web |
| **Electron** | Webview applications for desktop |
| **Native** | Possibly native iOS and Windows applications |

---

## Cloud Independence

The system does not rely on large cloud providers as a foundation. They may be used opportunistically but never as a requirement. The system must remain viable on self-hosted hardware, rented bare metal, or low-cost VPS providers without modification.
