# Deployment — Target-Agnostic

Infrastructure is **disposable**, not foundational. The system is the codebase and its deployment definition—not the machine it runs on. Any host is a temporary execution target.

**Summary:** Infrastructure is interchangeable, deployment is deterministic, and movement between environments is routine rather than exceptional.

---

## Automation Strategy

**Priorities:** (1) Deployment automation can stand alone. (2) IaC in place; determinism drives deployments. Both are required.

**Layered model:** Deployment is expressed as layers. High-level intent is the top-most declarative layer (e.g. "Deploy X version a.b.c in production with dependencies Y and Z"—each with its own declarative specification). Under that: a deterministic state-realizing solution (e.g. Ansible, current choice; could be displaced under scrutiny). Under that: deploy operations. Under that: Docker containers. Under that: libraries (built and published in a similar automated way). Layers upon layers.

---

## Named artifacts and implementation binding

**Author-facing statements use names, not wiring.** **Namespaces** (logical deployment contexts), data planes, and similar **targets** are **named artifacts**. The ESB carries a **base namespace** (e.g. `acme`, `xyz`); **environment** is **not** authored there (see **Environment injection** under ESB). After injection, the **resolved context** (e.g. `acme-staging`) selects a binding. Example phrasing: *OpenErgo components use **namespace `xyz`***—same ESB line for every environment. No RabbitMQ URLs, no Docker syntax, no vendor API detail at that layer.

**Note:** ESB **namespace** means a **logical handle** for “which slice of the platform this attaches to”—not necessarily the same thing as a Kubernetes `metadata.namespace` (though a binding *may* set that as an implementation detail).

**Logical vs concrete:** In the ESB, **namespace** is a **logical reference**—it identifies a *role or slice in the product model* (which bus plane, tenant context, application family attachment point), **not** a particular host, URL, cluster ID, or broker instance. **Environment**, supplied at **deployment time**, is the coordinate that turns that logic into **selection of a binding**. The binding row holds **references to concrete artifacts**: real endpoints, vhosts, clusters, image digests, credential mounts, volume IDs, and whatever else is *actually* touched in that environment. **Same** logical `namespace` string in the ESB across promotions; **different** `environment` at invocation → **different concrete realization**, without editing the ESB.

**Binding:** Each **resolved context key** maps to that **technical implementation** record (version-controlled registry, env catalog, or materialized config). It holds **profile** (sizing/tier), broker URL, vhost, credential references, tuning, and other concrete detail. Example: key `xyz-production` → *profile: standard; OpenErgo transport is AMQP; broker URL …; vhost …; credential reference …; tuning …*. Replacing *how* that context is realized updates **that binding row**; the ESB line *namespace `xyz`* stays unchanged across environments.

**Separation:**

| Layer | Says |
|-------|------|
| **Component / intent** | *Runs with **namespace `acme`*** (or attach store **`ledger-db`**, etc.)—**logical** only; **same** intent document for staging and production |
| **Resolved context + binding** | Pipeline supplies **environment**; resolver forms **`<namespace>-<environment>`** (or an equivalent rule) to load **concrete** endpoints, secrets references, profile, images, playbooks, or generated low-level files |
| **Execution** | Docker, Ansible, cloud APIs, etc.—**means**, not the conceptual definition |

**Goal:** Lowest barrier to entry for authors—**declare affiliation to a stable name**—while keeping **determinism and audit** in the binding layer so deployments remain reproducible.

---

## Empire State Build (ESB)

**Empire State Build (ESB)** is the name of the **lay deployment descriptor grammar**: the smallest human-facing surface that states **what** runs (component identities under a **logical namespace**), and **optionally** a **sparse** set of logical **`depends_on`** edges—only when the binding cannot imply them—while the system **infers** the rest via **named bindings**, **pipeline-supplied environment**, and versioned rules.

**Principle:** A human can describe a deployment with a **very small set of parameters**; the author does not treat Dockerfiles, inventory, or shell as the **source of truth**. Those artifacts are **materialized** to satisfy an end state derived from the ESB.

**ESB document** (authoring layer):

- Short, stable fields: **what** runs (component `id` and **optional** `version`—see below) and **which logical namespace** (`xyz`, `acme`, …—**without** embedding environment).
- **`depends_on`:** **omit by default.** Use only when a **logical attachment** cannot be inferred from **namespace + binding** rules (exceptions should be **rare**); see **Dependencies (`depends_on`)** below.
- **No “why” in ESB:** No product essays, `summary`, `meta` strategy fields, or inline documentation in the artifact—ESB captures **deploy graph “what”**, not rationale. Strategy and narrative live **outside** the ESB (aspect docs, design files, evaluation notes).
- **The same ESB is reused for every environment.** **Environment** is supplied **outside** the ESB (see below).
- **Profile** (sizing/tier) and other operational defaults **live in the binding** for the **resolved** context, not in the lay ESB.
- The grammar starts **minimal** and **grows only** when a real gap appears; every new field must earn its place (see `TENETS.md`).

**Schema version (`esb`):** A small **grammar / document version** (e.g. `esb: "0.1"`) is appropriate so tooling knows how to parse the file. It is separate from component version policy (below).

**Component `version` (optional, low-friction):** Humans often **omit** an exact semver. The grammar supports: **(a)** **no** `version` field—defaults come from **platform convention** (e.g. channel `latest`, tag policy, or defaults in the binding); **(b)** **attributes** such as `latest`, `stable`, or a **channel / track** name agreed in policy; **(c)** an **explicit pin** (semver, digest) when reproducibility, audit, or rollback requires it. Allowed tokens and defaults are **catalog / policy**, not something every lay author repeats.

### Dependencies (`depends_on`)

**Sparse use is the default.** The list exists for **logical edges the namespace + binding do not imply**. Many capabilities (message bus reachability, channel credentials, default datastore handles) are **materialized by the binding** for the component’s **namespace**—components **do not** declare the bus, mailer, or peers **per OpenErgo-style design**: they **react when applicable** without hard-coded upstream/downstream wiring in the ESB.

When **`depends_on` appears**, each entry is still a **logical artifact name** (blackbox; type is **not** tagged in ESB—the **binding** concretizes).

**Uniqueness:** Names used in `depends_on` must remain **unique in the artifact catalog** when present.

**Resolution:** If present, dependency names follow the **same environment-scoped binding rules** as namespaces unless the catalog marks a name **global**.

**Rare cross-graph case:** When a deliverable truly needs a **named** logical attachment that **cannot** be implied (e.g. a **foreign** namespace’s store), a **minimal** `depends_on` may list it—**proof** in the expanded spec that the exception earned its place.

### Environment injection (outside ESB)

**Directive:** *Which environment* (staging, production, …) **must** exist for a deploy to be defined—but it **does not** belong in the ESB file.

**Where it lives:** Deployment **invocation**—CI matrix, promote pipeline parameter, release CLI, infrastructure entrypoint, or equivalent—so the pipeline knows which binding row to use **together with** the ESB.

**Resolution:** Combine **logical namespace** from ESB with **environment** from the pipeline into a **resolved context key**, e.g. **`<namespace>-<environment>`** → `acme-staging`, `acme-production`. That key selects the binding that **materializes** the logical reference into **concrete** targets for this deploy. The joiner (`-`), ordering (`acme-staging` vs `staging-acme`), and escaping (if segments contain hyphens) are **platform conventions** fixed in the catalog—pick one system and apply it consistently.

**Convention:** Treat the ESB **namespace** value as the **base segment only** (do not bake `staging` or `prod` into the ESB string). That avoids ambiguous double-suffix bugs and keeps “one ESB, many environments” literal.

**`depends_on` + environment:** If `depends_on` is present, those names follow the **same environment-scoped binding rules** as above unless the catalog marks a name **global**—see **Dependencies (`depends_on`)**.

**Logic check:** If environment were **only** implicit (nowhere in pipeline or vault), deploys would be undefined. The ESB stays clean by moving that directive to **invocation**, not by deleting it from the system.

**Inference chain** (current stack direction):

```text
ESB + environment (from invocation, not in ESB file)  →  expanded / normalized spec (machine-readable)  →  state realizer (e.g. Ansible)  →  concrete actions (Docker pull/run, files, systemd, APIs, …)  →  measured end state
```

- **Ansible** (or successor) is the **deterministic state-realizing** layer today: idempotent tasks that close the gap between “what we declared” and “what is running.” It is an **implementation choice**, not the eternal definition—another engine could replace it if it honors the same **expanded spec** contract.
- **Docker, bash, cloud APIs** sit **below** that layer: **means** to converge the host, not the vocabulary the human masters first.

**Relationship to named artifacts:** ESB **references** base **namespace** and component **`id`** (and **optional** `version` / channel); **`depends_on`** is **optional and usually empty**. The pipeline supplies **environment**; **bindings** keyed by **resolved context** supply transport, stores, channels, profile, and wiring implied by the namespace; the realizer **consumes** the expanded result.

**Non-goal:** A single giant schema on day one. Goal is **grammar discipline**—small surface, clear inference, audit trail from ESB through materialized artifacts.

**Exercise:** A **protocol stress-test** model lives under **`esb-model/`** ([`esb-model/README.md`](../esb-model/README.md))—**lean YAML only** (no narrative inside the artifact). It evaluates the grammar, not any specific product.

### Representative ESB examples (illustrative — for review)

Not a frozen schema. **Shape:** `esb` + **namespace** + **component `id`**; **`depends_on` omitted** unless you are demonstrating a **rare** implied exception. **Environment** is never in-file. **`version`** optional.

**Example A — single component, default version implied by binding**

```yaml
esb: "0.1"
component:
  id: payux
namespace: acme
```

**Example B — explicit pin when needed**

```yaml
esb: "0.1"
component:
  id: open-ergo-worker
  version: "0.9.1"
namespace: xyz
```

**Example C — channel-style version; no `depends_on` (ledger and bus implied by namespace + binding)**

```yaml
esb: "0.1"
component:
  id: payux
  version: latest
namespace: acme
```

**Example D — stack slice (mixed: default vs pinned)**

```yaml
esb: "0.1"
namespace: acme
components:
  - { id: payux-api }
  - { id: payux-worker, version: "1.2.3" }
```

**Example E — exceptional `depends_on` (only when catalog rules cannot imply the edge)**

```yaml
esb: "0.1"
component:
  id: edge-router
  namespace: acme
  depends_on:
    - foreign-audit-sink
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
