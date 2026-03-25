# 2. Operating Profiles and Deployment Philosophy

This section defines how the platform exists in the real world.  
It specifies the deployment philosophy, operating profiles, packaging model, and operational invariants.  

Deployment is treated as a first-class architectural concern.

---

## 2.1 Infrastructure-Automation-First Strategy

The platform is infrastructure-automation-first.

### Priorities

1. **Deployment automation can stand alone** — Automation is self-contained and operable without external platform lock-in.
2. **IaC and determinism** — Infrastructure as Code is in place; determinism drives deployments. Repeatable, reproducible.

### Layered Deployment Model

Deployment is expressed as layers. Top-down:

| Layer | Description | Example |
|-------|-------------|---------|
| **High-level intent** | Declarative: "Deploy X v1.2.3 in production with dependencies Y and Z." Each dependency has its own declarative spec. | Version + target + deps |
| **State-realizing solution** | Deterministic tool that realizes the desired state. | Ansible (current choice; could be displaced under scrutiny) |
| **Deploy operations** | Concrete deploy steps. | |
| **Containers** | Docker containers. | |
| **Libraries** | Built, published in similar automated way. | |

Layers upon layers. Ansible is an implementation choice that satisfies the strategy; it is not a fixed requirement.

### Core Principle

Deployment is controlled through infrastructure automation. Ansible is the current implementation choice.  
Container orchestration (e.g., Kubernetes) is optional and secondary.

### Rationale

- Self-host must be first-class.
- Bare-metal and VM deployments must be fully supported.
- The system must not depend on a specific cloud provider.
- Orchestrators may be layered on top, but never required.

### Rules

- Every service must be deployable on a single host without Kubernetes.
- Every service must expose:
  - Configuration via file + environment variables.
  - Health endpoint.
  - Version endpoint.
  - Structured logs to stdout.
- No deployment may assume managed cloud infrastructure.

Infrastructure automation must:
- Install dependencies.
- Configure services.
- Manage secrets.
- Handle upgrades and rollbacks.
- Configure networking and service discovery (static or DNS-based).

---

## 2.2 Deployment Profiles

The platform supports multiple operating profiles.  
These profiles are materially different compositions of aspects and subsystems.

Profiles share contracts, not topology.

### 2.2.1 POC / Self-Contained Mode

Purpose:
- Rapid development.
- Lightweight installs.
- Edge or laptop deployment.

Characteristics:
- Single host.
- SQLite permitted.
- Local filesystem object storage.
- Ergo may run:
  - Against remote Kafka.
  - Against local RMQ.
  - In stdio mode.
  - In single-process event loop mode.
- Minimal infrastructure services.

Constraints:
- Must obey same contracts as larger profiles.
- No shortcuts that violate architectural principles.

---

### 2.2.2 Standard Self-Host Mode

Purpose:
- Small organization deployment.
- Controlled environment.
- Privacy-focused installations.

Characteristics:
- 1–3 hosts.
- Postgres as primary datastore.
- S3-compatible object store (MinIO or equivalent).
- Kafka/RabbitMQ optional depending on enabled subsystems.
- Services may be co-located.

Isolation:
- Multi-tenant supported within one deployment.
- Resource isolation configurable.

---

### 2.2.3 SaaS Mode

Purpose:
- Multi-tenant hosted deployment.
- Horizontal scalability.

Characteristics:
- Dedicated Postgres cluster.
- Dedicated Kafka cluster.
- Dedicated object store.
- Multiple BFF instances.
- Multiple worker instances.
- Load-balanced HTTP ingress.

Rules:
- No cloud-provider-specific APIs embedded in application logic.
- Scaling achieved through horizontal replication of stateless services.

---

### 2.2.4 Enterprise-Isolated Mode

Purpose:
- Regulatory compliance.
- Performance isolation.
- Dedicated tenant environments.

Characteristics:
- Per-tenant service isolation.
- Optional per-tenant databases.
- Optional per-tenant Kafka partitions or clusters.
- Independent resource allocation.

Isolation must be achievable:
- Without rewriting code.
- Without forking the application.
- Through configuration and infrastructure automation only.

---

### 2.2.5 Hybrid Mode

Purpose:
- Mixed local/remote execution.
- Edge + centralized backbone.

Characteristics:
- Local components talking to remote Kafka/RabbitMQ.
- Local BFF interacting with remote projection store.
- Local AI services with remote model fallback.

Hybrid deployments must use the same contracts as non-hybrid deployments.

---

## 2.3 Immutable Artifact Model

All deployable components are immutable.

### Artifact Types

- OCI container images.
- Standalone binaries (where applicable).
- Versioned frontend bundles.

### Rules

- Every build produces a version-locked artifact.
- No runtime patching.
- No mutable code directories.
- Configuration is external to the artifact.
- Rollbacks occur by reverting to previous artifacts.

Versioning must be:
- Explicit.
- Traceable.
- Logged at startup.

---

## 2.4 Service Packaging Rules

Every service (BFF, worker, gateway, subsystem component) must conform to:

### 2.4.1 Configuration Contract

- Configuration file schema documented.
- Environment variables supported.
- Defaults explicit.
- No hidden configuration.

### 2.4.2 Health Contract

- `/health` endpoint.
- Distinguishes:
  - Liveness.
  - Readiness.
- Health must reflect dependency connectivity (DB, Kafka, etc.).

### 2.4.3 Version Contract

- `/version` endpoint.
- Returns:
  - Service version.
  - Build hash.
  - Schema versions supported.

### 2.4.4 Logging Contract

- Structured JSON logs.
- Must include:
  - correlation_id
  - request_id (if applicable)
  - service_name
  - version
  - timestamp

---

## 2.5 Configuration and Environment Modeling

Configuration must be:

- Explicit.
- Declarative.
- Environment-specific.
- Version-controlled.

Environment variables must not:
- Replace structured configuration files.
- Hide required settings.

Configuration layers:

1. Default config (checked into repo).
2. Environment config (deployment-specific).
3. Secrets (injected securely).
4. Runtime overrides (minimal and controlled).

---

## 2.6 Secrets Handling (High-Level)

Secrets must never be:

- Hardcoded.
- Embedded in artifacts.
- Logged.

Secrets may be sourced from:

- File-based secure templates (default).
- External secret manager (optional).
- OS-native secure storage (optional).

Secret injection must occur at deployment time.

---

## 2.7 Networking and Service Boundaries

Services communicate through:

- Explicit HTTP APIs.
- Explicit message buses.
- Explicit stdio wrappers (BYOL).

No implicit cross-service calls.

Service discovery may be:

- Static configuration.
- DNS-based.
- Infrastructure-managed.

Application code must not assume dynamic cloud discovery mechanisms.

---

## 2.8 Upgrade and Migration Strategy

Upgrades must be:

- Reproducible.
- Scriptable.
- Reversible.

Database migrations:
- Explicit.
- Versioned.
- Applied in controlled order.

No implicit schema drift.

Service startup must validate:
- Schema compatibility.
- Configuration validity.
- Dependency availability.

---

## 2.9 Local Development Parity

Local development must:

- Use the same artifacts.
- Use the same configuration model.
- Support local-only mode without remote dependencies.

Docker Compose or equivalent may be used for dev environments, but must not introduce alternate runtime logic.

---

## 2.10 Deployment as an Aspect

Deployment is not an afterthought.

Deployment considerations must be accounted for during:

- API design.
- Service boundaries.
- Observability integration.
- Resource usage decisions.
- Dependency selection.

No design decision may assume:

- Infinite horizontal scale.
- Managed cloud services.
- Externalized ops teams.

Deployment discipline enforces architectural discipline.

---

This section defines how the system is installed, scaled, isolated, upgraded, and operated.  
All future subsystem and product documentation must conform to these deployment constraints.