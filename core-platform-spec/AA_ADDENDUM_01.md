# Homelab in the Core Platform Reference Frame

---
Homelab current implementation/deployment:
    LOCATION: ~/nirvana/dev/infra/README
---

## What “homelab” means here
Homelab is the **current, running reference implementation** of the platform’s infrastructure and operational posture on a single private node. It is not a separate architecture. It is an **operating profile** optimized for:

- privacy-first operation
- self-sufficiency (no cloud dependency required)
- reproducibility via infrastructure automation
- rapid iteration with minimal external surface area

Homelab exists to prove that the platform can be built, operated, and evolved **entirely in-house** as a worst-case baseline. If the system works under this constraint, it can be lifted into broader topologies later.

## Implementation maturity and convergence approach
Homelab is **strategically sound**: the roles, contracts, and operating profile align with the platform. The **implementation**, however, is messy in places. The plan is to **slowly converge on correctness** and allow a **slow migration**—incremental cleanup and refactor, not a big-bang rewrite. New work (e.g. core-platform, core-ergo) should follow the spec and contracts from the start; homelab infra and existing services can be brought into line over time as they are touched or as dedicated migration steps are taken.

## How homelab fits into the larger reference frame
The Core Platform Architecture & Engineering Specification defines:
- the **roles** in the system (BFF, event log, router, projections, workers, etc.)
- the **contracts** between them (envelopes, idempotency, scoping, sync patterns, error handling)
- the **cross-cutting aspects** (deployment, observability, identity/provisioning/entitlements)
- the **iteration plan** to converge toward the long-term platform

Homelab is simply one concrete instantiation of those roles and aspects:
- it runs real infrastructure components now
- it implements the deployment aspect concretely (Ansible-first, compose-based)
- it demonstrates that the system’s “copper wire” (infrastructure substrates) can be treated as replaceable commodities

Homelab should be treated as the **baseline operating profile** against which iteration work can be validated.

## How homelab aligns with the platform strategy
Homelab aligns with the platform strategy at the principle level:

1) Abstraction over substrates
Homelab treats infrastructure components as **replaceable implementations** behind stable contracts. This matches the platform strategy where equity lives in:
- contracts
- envelopes
- idempotency and convergence logic
- isolation of concerns

2) Isolation of concerns
Homelab separates:
- provisioning/deployment (Ansible + compose)
- persistence (explicit mounts and durable volumes)
- event flow (durable log + routing layer + bridges)
- application runtime (services and workers)
This mirrors the platform doctrine: clean seams, stable boundaries, minimal coupling.

3) Self-sufficiency as proof of portability
Homelab proves the platform can operate without:
- cloud IAM
- cloud messaging
- cloud object storage
- hosted CI/CD or hosted registries

This strengthens the platform’s portability claim: if needed, any component can later be lifted and shifted to cloud equivalents without changing the core contracts.

4) Deployment and observability as pervasive aspects
Homelab’s emphasis on:
- reproducible infra
- backups/DR
- monitoring/logging
is consistent with the platform plan that treats deployment + observability as always-on aspects from the start.

## How homelab diverges from the platform strategy (and why that’s acceptable)
On the surface, homelab may look different because it optimizes for privacy and self-sufficiency. The divergences are mostly *implementation choices*, not *architectural contradictions*:

1) Substrate selection (example: JetStream vs Kafka)
Homelab may use NATS JetStream where the long-term plan may consider Kafka. This is not a strategy conflict if both are treated as implementations of the same role:
- Canonical Event Log (event SOR)
The platform strategy should be expressed in terms of roles and contracts so that swapping the substrate is a configuration change, not a rewrite.

2) Connectivity profile (LAN-only baseline)
Homelab is intentionally LAN-only to:
- reduce attack surface
- keep IP internal
- ensure no cloud dependencies
This diverges from a future public-facing deployment (e.g., Hot Potato viral product) but does not contradict the platform. It simply represents a different operating profile. The platform must support both:
- LAN_ONLY profile (homelab)
- PUBLIC_EDGE profile (future)
without changing the internal contracts.

3) MVP security posture and pinning
Homelab may defer:
- TLS
- Vault/secret hardening
- strict image pinning
These are tactical MVP decisions, not principle changes. They should be tracked as temporary tensions, not merged into the strategy as permanent norms.

## How homelab should converge toward a unified strategy
Unification does not mean “make homelab identical to the ideal architecture.” It means:

- strategy documents describe **roles + contracts**
- homelab provides one concrete **profile-specific implementation** of those roles
- other environments (cloud, enterprise, hybrid) provide different implementations of the same roles
- all profiles remain compatible because the **contracts do not change**

The convergence path is:

1) Rename implementations to roles in the strategy
Use role-based terms in the spec (e.g., “Canonical Event Log”) and list substrates (JetStream/Kafka/etc.) as interchangeable implementations.

2) Treat connectivity as a first-class profile dimension
Document connectivity profiles explicitly. Homelab remains LAN_ONLY. Public deployment becomes a separate PUBLIC_EDGE profile that adds an edge adapter layer without changing internal contracts.

3) Keep enforcement and hardening as staged maturity
Homelab can operate with soft enforcement and MVP hardening while still validating the architecture. As maturity increases, hardening becomes profile-gated (especially PUBLIC_EDGE).

4) Maintain a stable “lift-and-shift contract”
Any infra component should be liftable because applications and workflows bind to:
- BFF contracts
- gateway contracts
- event envelopes
- projection read models
not to specific vendors or toolchains.

## Practical guidance for the agent
When evaluating iteration progress, treat homelab as:
- the baseline environment for proving portability and self-sufficiency
- the implementation profile that validates role separation and contract clarity

When you see divergence, classify it correctly:
- If it’s substrate-level (JetStream vs Kafka), it is an implementation swap: acceptable.
- If it changes contracts/envelopes, it is strategy-impacting: requires an explicit amendment.
- If it changes connectivity/security posture, it is a profile difference: track as a profile requirement (PUBLIC_EDGE vs LAN_ONLY).

## Summary
Homelab and the Core Platform Architecture are chasing the same goals:
- abstraction
- isolation of concerns
- convergence on a single source of truth
- reproducible operations
Homelab biases for privacy and self-sufficiency, but remains compatible with the platform strategy because the architecture is defined by contracts, not tools. The unified strategy is achieved by documenting roles and profiles clearly, and treating homelab as a first-class operating profile that proves the platform can function without external dependencies.




# Core Platform Architecture & Engineering Specification
## Amendments + Baseline Strategy (Homelab as Ground Truth)

This section treats the existing homelab implementation as the **current baseline** and records **amendments** to the platform strategy as explicit change data, without rewriting the original strategy content.

---

## A) Baseline Interpretation Strategy (How to Use the Homelab as Ground Truth)

### A.1 Baseline Definition
- The homelab stack is the **reference implementation baseline** for:
  - deployment profile: LAN-only, single-node, Ansible-first
  - event substrate: NATS JetStream + RabbitMQ
  - infra conventions: mounts, compose layout, backup/DR, internal registry, internal pypi, local git

### A.2 Baseline vs Strategy Relationship
- The strategy doc remains the **idealized target architecture**.
- The homelab baseline is the **current operating profile** that proves:
  - no cloud dependency
  - substrate swap-ability is real
  - reproducible infra is feasible
- Conflicts are tracked as **tensions**, not resolved by rewriting either side.
- Resolution happens only when:
  - a new operating profile is introduced (e.g., “Public Edge”), or
  - the baseline evolves (e.g., Kafka replacing JetStream), or
  - the strategy is updated by amendment.

### A.3 Naming Convention (Avoid Tool Lock-In in the Docs)
- When a component is a commodity substrate, refer to it by **role**, not vendor/tool name:
  - “Canonical Event Log (CEL)” instead of “Kafka”
  - “Ephemeral Router” instead of “RabbitMQ”
  - “Object Store” instead of “S3/MinIO/Wasabi”
- Baseline can list concrete implementations as: “CEL implementation = NATS JetStream (homelab)”.

### A.4 Operating Profiles
The baseline is explicitly one operating profile:
- Profile: `LAN_ONLY_SINGLE_NODE_HOME_LAB`
Future profiles are allowed without changing contracts:
- `PUBLIC_EDGE`
- `HYBRID`
- `ENTERPRISE_ISOLATED`

---

## B) Amendments to the Strategy Document (Change Data)

### B.1 Event Source of Record (SOR) Language
CHANGE:
- “Kafka is the single source of truth / SOR for events.”

TO:
- “The system defines a **Canonical Event Log (CEL)** role as the event SOR. CEL implementations are interchangeable commodities (e.g., NATS JetStream for homelab, Kafka for scale). The equity is in the contracts and envelopes, not the substrate.”

RATIONALE:
- Current baseline uses NATS JetStream as the durable log and proves the abstraction.

---

### B.2 RabbitMQ Role Language
CHANGE:
- “RabbitMQ is an ephemeral router.”

TO:
- “RabbitMQ is an **Ephemeral Router** role implementation. The system may reconstitute streams from the CEL into an ephemeral router when needed. Router implementations are swappable (RabbitMQ today; others possible later).”

RATIONALE:
- Baseline already implements CEL → Rabbit reconstitution via bridge.

---

### B.3 Ergo Subsystem Framing
CHANGE:
- “Kafka -> rabbitmq -> python -> ... is the lifecycle.”

TO:
- “Ergo is a black-box subsystem that runs over **pluggable substrates**. The lifecycle is described in terms of roles (CEL, router, worker runtime, stdio bridge). Concrete substrate choices are operating-profile-specific.”

RATIONALE:
- You explicitly treat copper wire as commodity; ergo SDK abstracts substrates.

---

### B.4 Connectivity Assumption (LAN-only vs Public)
CHANGE:
- “No WAN exposure” as an implicit universal posture.

TO:
- “Connectivity is an explicit **operating profile dimension**. LAN-only is a first-class profile used to prove self-sufficiency. Public exposure is supported via a distinct `PUBLIC_EDGE` profile with an edge adapter layer (TLS termination, ingress controls, webhook ingress, rate limiting).”

RATIONALE:
- Baseline is LAN-only by design; Hot Potato will eventually require a public profile.

---

### B.5 Observability Tooling Assumption
CHANGE:
- “OpenTelemetry everywhere” as a default.

TO:
- “The platform defines an internal **Observability Contract** (correlation, structured logging fields, audit conventions, minimal metrics API). Tooling/exporters are adapters. OpenTelemetry is optional and not canonical.”

RATIONALE:
- OTel is explicitly rejected as canonical; you prefer a minimal home-grown API.

---

### B.6 Deployment Aspect (Ansible-first) Promotion
CHANGE:
- Deployment described as a general goal.

TO:
- “Infrastructure automation is the primary deployment interface (Ansible-first). Orchestrators are optional adapters. The homelab baseline is the reference implementation of this aspect.”

RATIONALE:
- Baseline already operationalizes this.

---

### B.7 “Local Endpoints” Terminology
CHANGE:
- “Some BFF endpoints may be local.”

TO:
- “Local vs remote is a **substrate concern**, not an API concern. APIs may resolve via direct reads or via the Ergo subsystem, but all code should be written against contracts that remain valid across substrates.”

RATIONALE:
- You’ve defined multiple local execution modes under the same ergo SDK.

---

## C) Tensions to Track (Not Yet Resolved)

These are acknowledged deltas between baseline and target strategy, intentionally deferred:

- Image pinning vs :latest usage (baseline uses latest in places; strategy favors pinned immutable builds).
- Plaintext secrets + no TLS (baseline MVP) vs production posture (future).
- LAN-only posture vs Hot Potato public viral posture (requires `PUBLIC_EDGE` profile later).
- Enforcement (soft conventions now) vs hard enforcement later (Schrödinger subtrees).

---

## D) Next Practical Step

1) Add this section as `Amendments & Baseline` in the spec.
2) In the original strategy text, do not edit the old statements; instead add footnote-style links to the relevant amendment IDs (B.1–B.7).
3) Maintain a short “Baseline Inventory” appendix that lists the currently running homelab services by role + implementation (e.g., CEL=NATS JetStream).

---


