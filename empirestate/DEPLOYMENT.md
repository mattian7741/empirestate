# Deployment — Target-Agnostic

Infrastructure is **disposable**, not foundational. The system is the codebase and its deployment definition—not the machine it runs on. Any host is a temporary execution target.

**Summary:** Infrastructure is interchangeable, deployment is deterministic, and movement between environments is routine rather than exceptional.

---

## Automation Strategy

**Priorities:** (1) Deployment automation can stand alone. (2) IaC in place; determinism drives deployments. Both are required.

**Layered model:** Deployment is expressed as layers. High-level intent is the top-most declarative layer (e.g. "Deploy X version a.b.c in production with dependencies Y and Z"—each with its own declarative specification). Under that: a deterministic state-realizing solution (e.g. Ansible, current choice; could be displaced under scrutiny). Under that: deploy operations. Under that: Docker containers. Under that: libraries (built and published in a similar automated way). Layers upon layers.

---

## Named artifacts and implementation binding

**Author-facing statements use names, not wiring.** **ESB** (full stack—see [Empire State Build](#empire-state-build-esb)) starts at the **lay** layer: **domains** (systems in speech) and **artifact tokens** that **belong** to each—**membership only**. Each token then **resolves** through **interim layers** to **concrete** targets; **bindings** map **resolved context** + **how** for a given **environment**.

**`domain` / `id` vs `namespace`:** The ESB **group** key is **`id`** (under **`domains[]`**, or as the **root** of a one-group document). **`namespace`** in the Kubernetes / platform sense is an **implementation** concern—materialized in **binding rows**—not the primary word for **lay** grouping (unless a lower descriptor explicitly maps a domain to that runtime concept).

**Logical vs concrete:** Domain ids and artifact tokens are **logical**. **Environment** at **invocation** selects **which binding** applies; the binding holds **concrete** references (URLs, clusters, secrets, tuning). **Same** lay corpus across promotions; **different** environment → **different concrete realization** without editing membership lists.

**Binding:** A **resolved context key** (platform convention, e.g. **``<logical-anchor>-<environment>``**) points at the **technical implementation** record for **how** and **where**. Example: `xyz-production` → *profile, broker URL, vhost, credentials, …*. Changing **how/where** edits **bindings**; lay **what** stays stable.

**Separation:**

| Layer | Says |
|-------|------|
| **Lay membership (ESB top)** | This **domain `id`** **includes** these **artifact tokens** (string list only). |
| **Resolved context + binding** | Invocation **environment** + catalog rules → **concrete** endpoints, secrets, profile, images, playbooks, generated files. |
| **Execution** | Docker, Ansible, cloud APIs, etc.—**means**, not the lay definition. |

**Goal:** **Enumerate what belongs** to **named systems**; keep **how** and **where** in **descriptors and bindings** so deployments stay deterministic and environment-agnostic at the lay layer.

---

## Empire State Build (ESB)

**ESB scope:** **Empire State Build (ESB)** is the **entire artifact-resolution system**, not only the top YAML file. It spans **all layers** that turn a **logical artifact** from an opaque, speech-level name into **concrete implementation facts**: interim steps fix **persistence class**, **engine**, **topology**, **schema**, **static reference data**, **credentials**, etc., until a **realizer** (Ansible, Terraform, provider API, …) can act. The **lay corpus** (below) is the **top** of ESB; **catalogs, manifests, bindings, and schemas** are **middle and bottom** of the **same** ESB story.

**Definition problem (data-plane example):** **`user-data-store`** at the top means “where user contact/profile fields (email, phone, …) live” **without** committing to SQL, a file, P2P, or a vendor. At the bottom it must become, e.g., a **Postgres** connection target, **named database + schema**, **`user_account` (and related) tables**, and **seed rows** for static domains (e.g. **US State** if the model has that property). The **rows of the resolution matrix** in [`esb-model/resolution-matrix/README.md`](../esb-model/resolution-matrix/README.md) show how those layers hang together for one token.

### Analogy (network stacks) and canonical layer count

**Parallels (OSI / TCP/IP):** Refinement from **“what we mean”** to **“what actually runs on the wire”** is **similar in spirit** to stacked network models: upper levels stay **abstract**; lower levels add **addressing, framing, physical delivery**. **Differences:** Internet stacks are **standards** with **tight** peer contracts at each boundary; **ESB** tiers are **your** platform contracts; deploy paths are **more heterogeneous** (relational DB, mobile binary, message worker) than packets; some **tiers may be skipped** or **defaulted** when policy implies them.

**Fixed number of layers — a reasonable goal if:** you define a **small, stable** set of **logical** tiers (on the order of **five to seven** names, not a magic OSI count) such that **every deployable kind** **maps** onto **the same slot sequence** at the **conceptual** level—while allowing **(a)** **skipped** steps with defaults, **(b)** **kind-specific** fields only in **allowed** slots (especially **terminal / expanded**), and **(c)** **polymorphic** concrete payloads (Postgres binding vs Play listing vs OCI image) **under** one shared **resolution** story.

**Unreasonable if:** you require **one identical schema** at every step for all kinds; or you freeze **seven** steps because of the OSI metaphor rather than **minimal** tiers that **actually separate concerns** for *your* stack.

### ESB lay corpus (top layer)

The **lay** grammar is **membership only**: **what** (conceptually) belongs to a **named system**—**groups and string tokens**, no mechanism. **How** and **where** are **not** in the lay file; they emerge from **lower ESB layers** (descriptors + **environment** + **bindings**).

**Three deploy questions:**

| Question | Where it is answered (inside ESB) |
|----------|-------------------------------------|
| **What (opaque)** | **Lay corpus:** **domain `id`**, **artifact tokens** (strings); optional recursive **token → group** definitions. |
| **What (refined)** | **Interim layers:** persistence model, engine family, topology, schema contracts, static-domain policies, … |
| **How / where (concrete)** | **Bindings + expanded spec** after **environment** at invocation; then **realizers**. |

**Principle (lay):** A human **enumerates membership**: “this **named thing** includes these **named things**.” Each **artifact** entry is a **string token** that **uniquely identifies** the **next** definition in the stack (another **group** in the lay corpus, or a **catalog** row that starts non-lay refinement). **No** `id:` maps under **`artifacts`**—the list is **only** strings.

**Shape before semantics:** You can read the file **without** knowing what any token means. Expanders and catalogs **resolve** tokens to finer graphs until **how/where** apply.

**Document shape:**

- **`domains`:** a list of **groups**. Each item has **`id`** (the domain’s name in lay speech) and **`artifacts`:** a **YAML array of strings only**—each string names a member token.
- **Alternate:** **one** group per file with root **`id`** + **`artifacts`** (no enclosing `domains` list).

**Recursive resolution:**

- Every token in **`artifacts`** **must** resolve to **exactly one** next step in the **ESB** stack: often another **lay** row (**`id`** match + child **`artifacts`**), or a **catalog / descriptor** entry that begins **non-lay** refinement (persistence class, engine, …).
- **Expansion:** `x` resolves to `x`'s definition → `[a, b, c]` → each of `a,b,c` resolves again. **Cycles** are **invalid**.
- **Speech-level bundles:** listing **`auth`** once can expand to many runnable pieces; the author does **not** flatten sub-pieces at this layer.

**Schema version (`esb`):** Grammar/document version (e.g. `0.1`).

**No “why” in lay corpus:** No comments, summaries, or strategy fields in **`.esb.yaml`** lay files—only mechanical structure.

**Same corpus across environments:** **Environment** is **invocation**, never a field in the lay list.

**Lay non-features:** Per-member **`id:` maps**, **`depends_on`**, **`version`**, and inline **`esb:`** file pointers on list items are **not** part of this enumerative model—linkage is **by token identity** → **definition lookup**. Edge cases belong in **lower descriptors** if they cannot be modeled as named groups.

### Environment injection (outside lay corpus)

**Directive:** *Which environment* **must** be supplied for a deploy—but **not** inside the lay membership lists.

**Where it lives:** Pipeline / CLI / entrypoint **invocation**.

**Resolution:** Combine a **logical anchor** (policy selects which **domain `id`** or deploy root drives the key) with **`environment`** into a **resolved context key** (convention e.g. **``<logical-anchor>-<environment>``** → `acme-staging`). That key selects **bindings** (**how** + **where** concrete).

**Convention:** **Lay `id`** and artifact tokens do **not** embed `staging` / `prod`—environment is **only** from invocation.

**Logic check:** With no environment anywhere in tooling, **where** is undefined.

**Inference chain** (current stack direction):

```text
Lay corpus + interim ESB layers + environment (invocation)  →  expanded spec (machine-readable)  →  state realizer (e.g. Ansible, Terraform)  →  concrete actions  →  measured end state
```

- **Ansible** (or successor) is the **deterministic state-realizing** layer today: idempotent tasks that close the gap between “what we declared” and “what is running.” It is an **implementation choice**, not the eternal definition—another engine could replace it if it honors the same **expanded spec** contract.
- **Docker, bash, cloud APIs** sit **below** that layer: **means** to converge the host, not the vocabulary the human masters first.

**Relationship to named artifacts:** The **lay corpus** names **domains** and **string tokens**; **every** token is a **key** into **deeper ESB layers** (not only more lay groups). After refinement and **environment** injection, **bindings** supply **concrete** mechanism and targets; the realizer **consumes** the expanded result.

**Non-goal:** A single giant schema on day one. Goal is **grammar discipline**—small surface, clear inference, audit trail from ESB through materialized artifacts.

**Exercise:** Under **`esb-model/`** ([`esb-model/README.md`](../esb-model/README.md)): **lay** sample [`ideal-system.esb.yaml`](../esb-model/ideal-system.esb.yaml); **data-plane resolution matrix** ( **`user-data-store`** ) [`resolution-matrix/README.md`](../esb-model/resolution-matrix/README.md); **compute E2E** (OpenErgo worker → Ansible → Docker) [`resolution-walkthrough/README.md`](../esb-model/resolution-walkthrough/README.md).

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

**Example C — recursive token: sibling `domains` entry (same file)**

`open-ergo` lists the token **`open-ergo-auth`**; another **`domains`** row defines that **`id`** and its members:

```yaml
esb: "0.1"
domains:
  - id: open-ergo
    artifacts:
      - open-ergo-http-gateway
      - open-ergo-auth
  - id: open-ergo-auth
    artifacts:
      - authcode-generator
      - logout-router-orchestrator
```

**Alternate:** the **`open-ergo-auth`** row can live in **`definitions/open-ergo-auth.esb.yaml`** (root **`id: open-ergo-auth`** + **`artifacts`**) instead—**same** lookup key, **platform** defines how files are indexed.

---

## Bottom-up layers (implementation) vs lay ESB (organization)

This section walks **up** from **code and packages** to **compound deployment** and **speech-level grouping**, so **concern boundaries** stay visible. **ESB** stays at the **top** of this stack: **membership and names**, not mechanism.

### Example path: OpenErgo component → Docker image

1. **Source** — Implementer writes code (e.g. a Python function) with **library dependencies** (e.g. packages from an index such as PyPI). That is **behavior**, not deployment vocabulary.
2. **Library artifact** — The function (and its dependency closure) is **released** to an **artifact registry** (package index or internal equivalent). That **publication** is already a form of “deployment” of the **library**, separate from runtime orchestration.
3. **OpenErgo component configuration** — A **YAML (or equivalent) descriptor** is the **logical OpenErgo component**: it names the component, ties it to the **OpenErgo logical network** (`namespace` / domain in the **OpenErgo** sense—not the lay ESB **`domains`** key), points at the **library** and **function**, and declares **input/output bindings**, **sub keys**, **default publish keys**, **pre/post operations**, etc. **Here** the injected function is **chosen behavior**; the **file** is the **component contract** for how that behavior runs **inside** the bus.
4. **Runnable image (one packaging pathway)** — A **Docker image** (or OCI artifact) is built that **bundles** at least: the **released function/library layer**, the **OpenErgo SDK/runtime**, and the **component configuration**. The image is a **static, versioned deployable** for environments that run **containers**.
5. **Other pathways** — **Docker is not the only deploy kind.** The same **logical** item might instead (or also) be described for **Google Play**, **Apple App Store**, **managed Postgres (e.g. Neon)**, **managed RabbitMQ (e.g. CloudAMQP)**, **static web**, **serverless**, etc. Each **kind** has its own **implementation descriptor** and **packaging/publish** steps; the **realizer** (Ansible or successor) closes the gap per kind.

### Middle: deployment manifest (compound)

A **deployment manifest** (or equivalent **machine-normalized** layer) lists the **logical elements** that participate in a **compound** deployment. **Names** in that manifest are **references** into **lower-level definitions**: OpenErgo component configs, image digests, store listing metadata, DB instance specs, broker subscriptions, etc. **Orchestration** (order, health, rollout) lives **at or below** this layer—not in the **lay** ESB string lists.

### Top: ESB lay corpus (speech-level grouping)

The **lay corpus** sits **above** per-kind descriptors: it only says which **named systems** (`id` / domain) **include** which **artifact tokens** (strings), **recursively**, until tokens hand off to **catalog / manifest** for refinement. It does **not** encode PyPI coordinates, SQL dialects, Docker layers, or Play Store SKUs.

### Concern separation (summary)

| Layer | Concern | Must **not** appear in lay ESB |
|-------|---------|--------------------------------|
| **ESB lay corpus** | Organization: **what belongs** to which **named system** | Per-kind mechanism |
| **Manifest / expanded spec** | **Compound** set of logical deployables; **references** to lower configs | — |
| **Implementation descriptor** | OpenErgo YAML, image metadata, store configs, **SaaS API** shapes, … | — |
| **Package / registry artifact** | Immutable **published** thing (wheel, image, bundle id, …) | — |
| **Source** | Code + deps | — |

### Gaps to close (staging)

- **Terminal typing:** When recursion stops, the expander must know each leaf token’s **deploy kind** (OpenErgo image, Neon DB, Play bundle, …)—via **catalog** or **manifest**, not via extra fields in the lay YAML.
- **One vocabulary, many realizers:** **Bindings** supply **where**; **descriptors + manifests** supply **how** per kind; **ESB** must stay **agnostic** as new kinds appear.
- **Naming collision:** **OpenErgo “namespace”** (logical bus network in component YAML) vs **lay `domains` / `id`** must stay **conceptually separate** in docs and tooling—same English word, different layers.

### Realization posture (no mandated stack)

The platform **does not** standardize on Kubernetes—or any **single** orchestrator, cloud, or runtime—as a **requirement**. **Lay ESB**, **catalog**, **bindings**, and **expanded spec** stay **portable intent**; nothing in them should **force** a K8s (or Nomad-only, or vendor-only) path.

**Concrete execution** should use **off-the-shelf tools wherever they fit**: configuration management and convergence (e.g. Ansible), resource provisioning APIs (e.g. Terraform / OpenTofu, Pulumi), **OCI/Docker** where container runtimes apply, **CI** for builds, **provider CLIs** for app stores or managed SaaS—**chosen per deploy kind**, **replaceable** under the same expanded-spec contract. Custom glue (**expansion**, validation, orchestration of manifests) is for **bridging layers**, not for **re-implementing** what mature tools already solve well.

---

## Core Principles

### Target Agnosticism from the Start
No part of the application or deployment depends on a specific provider, managed service, or proprietary runtime. If a component cannot be recreated elsewhere from first principles, it is a liability and is excluded from the core path.

### Fully Declarative
Everything required to run—services, networking, routing, configuration—is expressed in version-controlled files. No manual steps, no hidden state, no reliance on prior machine history. A fresh host must go from zero to fully operational using only the repository and a small set of external inputs (credentials, DNS, etc.).

### Containerization
**Containerized** services run in Docker (or OCI) with well-defined inputs and outputs; the image is a common **unit of portability** for that path. **Other deliverable kinds** (app stores, managed DB/broker SaaS, static web, etc.) use **different** packaging and constraints; see **Bottom-up layers** above—**ESB** does not privilege Docker as the only abstract deployable.

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
