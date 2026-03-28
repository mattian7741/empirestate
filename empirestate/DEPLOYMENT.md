# Deployment — Target-Agnostic

Infrastructure is **disposable**, not foundational. The system is the codebase and its deployment definition—not the machine it runs on. Any host is a temporary execution target.

**Summary:** Infrastructure is interchangeable, deployment is deterministic, and movement between environments is routine rather than exceptional.

---

## Automation Strategy

**Priorities:** (1) Deployment automation can stand alone. (2) IaC in place; determinism drives deployments. Both are required.

**Layered model:** Deployment is expressed as layers. High-level intent is the top-most declarative layer (e.g. "Deploy X version a.b.c in production with dependencies Y and Z"—each with its own declarative specification). Under that: a deterministic state-realizing solution (e.g. Ansible, current choice; could be displaced under scrutiny). Under that: deploy operations. Under that: Docker containers. Under that: libraries (built and published in a similar automated way). Layers upon layers.

---

## Named artifacts and implementation binding

**Author-facing statements use names, not wiring.** At the **lay** layer, **ESB** names **domains** (systems you can point at in speech) and the **artifact tokens** that **belong** to each—**membership only**, not mechanism. Every token **resolves** to **more detail** (another **group definition** or a **terminal** handled by non-lay descriptors). **Bindings** map **resolved context** (**where** + **which environment**) + **how** to **concrete** targets.

**`domain` / `id` vs `namespace`:** The ESB **group** key is **`id`** (under **`domains[]`**, or as the **root** of a one-group document). **`namespace`** in the Kubernetes / platform sense is an **implementation** concern—materialized in **binding rows**—not the primary word for **lay** grouping (unless a lower descriptor explicitly maps a domain to that runtime concept).

**Logical vs concrete:** Domain ids and artifact tokens are **logical**. **Environment** at **invocation** selects **which binding** applies; the binding holds **concrete** references (URLs, clusters, secrets, tuning). **Same** lay corpus across promotions; **different** environment → **different concrete realization** without editing membership lists.

**Binding:** A **resolved context key** (platform convention, e.g. **``<logical-anchor>-<environment>``**) points at the **technical implementation** record for **how** and **where**. Example: `xyz-production` → *profile, broker URL, vhost, credentials, …*. Changing **how/where** edits **bindings**; lay **what** stays stable.

**Separation:**

| Layer | Says |
|-------|------|
| **Lay membership (ESB)** | This **domain `id`** **includes** these **artifact tokens** (string list only). |
| **Resolved context + binding** | Invocation **environment** + catalog rules → **concrete** endpoints, secrets, profile, images, playbooks, generated files. |
| **Execution** | Docker, Ansible, cloud APIs, etc.—**means**, not the lay definition. |

**Goal:** **Enumerate what belongs** to **named systems**; keep **how** and **where** in **descriptors and bindings** so deployments stay deterministic and environment-agnostic at the lay layer.

---

## Empire State Build (ESB)

**Empire State Build (ESB)** is the **lay** grammar for **what** (conceptually) belongs to a **named system**—**groups and members only**. **How** things are deployed (mechanism, wiring, versions) and **where** they run (environment, concrete targets) are answered by **recursive descriptor resolution** and **bindings**, not by prose in the lay file.

**Three deploy questions:**

| Question | Where it is answered |
|----------|----------------------|
| **What** | ESB + linked definitions: **domain `id`**, **artifact tokens** (strings), expanded **recursively** until terminals. |
| **How** | Lower-level descriptors, catalog, bindings—mechanism per token. |
| **Where** | **Environment** at **invocation** + **binding** rows for **resolved context** → concrete stack. |

**Principle:** A human **enumerates membership**: “this **named thing** includes these **named things**.” Each **artifact** entry is a **string token** that **uniquely identifies** another **definition** (more group structure or a terminal). **No** `id:` maps under **`artifacts`**—the list is **only** strings.

**Shape before semantics:** You can read the file **without** knowing what any token means. Expanders and catalogs **resolve** tokens to finer graphs until **how/where** apply.

**Document shape:**

- **`domains`:** a list of **groups**. Each item has **`id`** (the domain’s name in lay speech) and **`artifacts`:** a **YAML array of strings only**—each string names a member token.
- **Alternate:** **one** group per file with root **`id`** + **`artifacts`** (no enclosing `domains` list).

**Recursive resolution:**

- Every token in **`artifacts`** **must** resolve to **exactly one** definition in the corpus: typically another **ESB** document whose root **`id`** **equals** that token and whose **`artifacts`** list further enumerates members—or a **non-lay** terminal record (catalog) that stops recursion.
- **Expansion:** `x` resolves to `x`'s definition → `[a, b, c]` → each of `a,b,c` resolves again. **Cycles** are **invalid**.
- **Speech-level bundles:** listing **`auth`** once can expand to many runnable pieces; the author does **not** flatten sub-pieces at this layer.

**Schema version (`esb`):** Grammar/document version (e.g. `0.1`).

**No “why” in ESB:** No comments, summaries, or strategy fields—only mechanical structure.

**Same corpus across environments:** **Environment** is **invocation**, never a field in the lay list.

**Lay non-features:** Per-member **`id:` maps**, **`depends_on`**, **`version`**, and inline **`esb:`** file pointers on list items are **not** part of this enumerative model—linkage is **by token identity** → **definition lookup**. Edge cases belong in **lower descriptors** if they cannot be modeled as named groups.

### Environment injection (outside ESB)

**Directive:** *Which environment* **must** be supplied for a deploy—but **not** inside the lay membership lists.

**Where it lives:** Pipeline / CLI / entrypoint **invocation**.

**Resolution:** Combine a **logical anchor** (policy selects which **domain `id`** or deploy root drives the key) with **`environment`** into a **resolved context key** (convention e.g. **``<logical-anchor>-<environment>``** → `acme-staging`). That key selects **bindings** (**how** + **where** concrete).

**Convention:** **Lay `id`** and artifact tokens do **not** embed `staging` / `prod`—environment is **only** from invocation.

**Logic check:** With no environment anywhere in tooling, **where** is undefined.

**Inference chain** (current stack direction):

```text
ESB + environment (from invocation, not in ESB file)  →  expanded / normalized spec (machine-readable)  →  state realizer (e.g. Ansible)  →  concrete actions (Docker pull/run, files, systemd, APIs, …)  →  measured end state
```

- **Ansible** (or successor) is the **deterministic state-realizing** layer today: idempotent tasks that close the gap between “what we declared” and “what is running.” It is an **implementation choice**, not the eternal definition—another engine could replace it if it honors the same **expanded spec** contract.
- **Docker, bash, cloud APIs** sit **below** that layer: **means** to converge the host, not the vocabulary the human masters first.

**Relationship to named artifacts:** Lay ESB names **domains** and **string tokens**; **every** token is a **key** into **more detail**. After expansion and **environment** injection, **bindings** supply **concrete** mechanism and targets; the realizer **consumes** the expanded result.

**Non-goal:** A single giant schema on day one. Goal is **grammar discipline**—small surface, clear inference, audit trail from ESB through materialized artifacts.

**Exercise:** A **protocol stress-test** lives under **`esb-model/`** ([`esb-model/README.md`](../esb-model/README.md))—**lean YAML only**.

### Representative ESB examples (illustrative — for review)

Not a frozen schema. **`artifacts`** are **strings only**. **Environment** never in-file. Token **`auth`** implies a **separate** definition with `id: auth` (another file or sibling `domains` entry).

**Example A — multiple domains in one file**

```yaml
esb: "0.1"
domains:
  - id: my_comprehensive_product
    artifacts:
      - auth
      - logging
      - databases
      - features
  - id: auth
    artifacts:
      - code-generator
      - session-deleter
```

**Example B — one domain per file (root `id`)**

```yaml
esb: "0.1"
id: acme
artifacts:
  - payux
```

**Example C — token resolved by external definition**

Parent `ideal.esb.yaml`:

```yaml
esb: "0.1"
domains:
  - id: open-ergo
    artifacts:
      - open-ergo-http-gateway
      - open-ergo-auth
```

Child `definitions/open-ergo-auth.esb.yaml` (definition **must** use the **same** string as its root **`id`**):

```yaml
esb: "0.1"
id: open-ergo-auth
artifacts:
  - authcode-generator
  - logout-router-orchestrator
```

**Lookup:** expander finds **`open-ergo-auth`** → document with **`id: open-ergo-auth`** (convention: definitions index, path rules, or catalog—**platform** choice).

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
