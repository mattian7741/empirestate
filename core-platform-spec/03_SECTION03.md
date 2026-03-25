# 3. Cross-Cutting Aspects

This section defines the cross-cutting aspects that apply across all products, subsystems, and deployment profiles. Aspects are pervasive and non-optional. They must be implemented consistently regardless of topology (POC, self-host, SaaS, enterprise-isolated).

Aspects define rules and contracts — not specific technologies.

---

# 3.1 Observability Aspect

Observability is mandatory from inception. It is not added later.

Observability must function in:
- Single-node POC mode
- Distributed SaaS mode
- Hybrid local/remote mode
- Enterprise-isolated deployments

## 3.1.1 Correlation Model

Every execution context must carry:

- `correlation_id` (global trace identifier)
- `request_id` (local request scope identifier)
- `tenant_id`
- `app_domain`
- `principal_id` (if applicable)
- `timestamp`

Rules:

- Correlation context must propagate across:
  - HTTP boundaries
  - Ergo messages
  - Async job transitions
  - Worker invocations
  - stdio bridges
- Context must be explicitly passed, not implicitly inferred.
- Loss of correlation context is considered a defect.

---

## 3.1.2 Structured Logging Contract

All logs must be structured (JSON).

Minimum fields:

- level
- timestamp
- service_name
- version
- correlation_id
- request_id
- tenant_id (if applicable)
- app_domain (if applicable)
- message
- error_code (if applicable)

Rules:

- No plain-text logging.
- No ad-hoc string concatenation.
- Errors must include structured metadata.
- Logs must never expose secrets.
- Log format must be consistent across languages.

---

## 3.1.3 Metrics Contract

Metrics are minimal but mandatory.

Each service must expose:

- request count
- error count
- latency (timing)
- resource usage indicators

Metrics must:

- Be cheap.
- Be profile-agnostic.
- Work without external SaaS dependencies.
- Be exportable to external systems if desired.

Metrics are not allowed to require distributed infrastructure.

---

## 3.1.4 Audit Event Model

Sensitive operations must emit audit events.

Examples:

- Identity changes
- Entitlement changes
- Billing operations
- Administrative actions
- Data deletion

Audit events must:

- Be immutable.
- Include actor, target, timestamp.
- Include correlation_id.
- Be stored durably (store depends on profile).

Audit is distinct from logging.

---

# 3.2 Provisioning Aspect

Provisioning governs the lifecycle of tenants, applications, and environments.

Provisioning must be:

- Deterministic
- Repeatable
- Scriptable
- Automatable

## 3.2.1 Tenant Provisioning

Tenant provisioning includes:

- Tenant identity creation
- Default role initialization
- Default entitlement configuration
- Storage namespace initialization
- Optional billing account association

Provisioning must:

- Be idempotent.
- Be re-runnable safely.
- Not assume centralized orchestration.

---

## 3.2.2 Application Provisioning

Application provisioning includes:

- App domain registration
- Microfrontend registration
- BFF namespace initialization
- Entitlement schema registration
- Storage schema initialization

Provisioning must:

- Not require manual database mutation.
- Be expressed declaratively.
- Support versioned upgrades.

---

# 3.3 Identity Aspect

Identity is tenant-scoped by default.

## 3.3.1 Tenant-Scoped Users

Rules:

- A user exists within exactly one tenant scope.
- Identity data must not be implicitly shared across tenants.
- All operations must be evaluated relative to tenant scope.

---

## 3.3.2 Role Model

- Roles are defined per tenant.
- Roles map to entitlements.
- Roles must not encode application logic.
- Role definitions must be declarative.

---

## 3.3.3 Bridge Entity Model

Cross-tenant identity relationships must be explicit.

Rules:

- Bridging must be intentional.
- Bridging must be revocable.
- Bridging must be auditable.
- Bridging must not collapse isolation guarantees.

Bridge entities must not be hacks or implicit merges.

---

# 3.4 Entitlements Aspect

Entitlements define what actions are permitted.

Entitlements are not equivalent to roles.

## 3.4.1 Canonical Entitlement Schema

Entitlements must be:

- Schema-defined
- Versioned
- Declarative
- Namespaced per app domain

Entitlement definitions must not live exclusively in code.

---

## 3.4.2 Runtime Evaluation Rules

Entitlement checks must:

- Be deterministic.
- Be stateless.
- Accept explicit scope and principal.
- Produce binary or structured results (allowed, denied, reason).

Checks must occur in:

- BFF layer
- UI gating (defensive)
- Server-side workflows

UI gating is convenience. Server-side gating is authoritative.

---

## 3.4.3 Gating Conventions

UI must:

- Hide or disable inaccessible features.
- Tolerate mismatches between UI state and server enforcement.

BFF must:

- Enforce entitlements regardless of UI state.
- Return standardized error envelopes on violation.

---

# 3.5 Deployment Aspect

Deployment is a pervasive aspect and not an afterthought.

## 3.5.1 Infrastructure-Automation-First

Deployment must be:

- Scriptable (e.g., Ansible-first philosophy)
- Deterministic
- Environment-driven (config, not code forks)

Kubernetes or other orchestrators are optional.

---

## 3.5.2 Immutable Artifacts

All services must:

- Be versioned.
- Be immutable.
- Be deployable independently.
- Expose version endpoints.

No mutable runtime patching.

---

## 3.5.3 Profile-Agnostic Service Contracts

Services must:

- Not assume distributed infrastructure.
- Not assume cloud provider services.
- Not require centralized control planes.

Profiles determine topology, not service behavior.

---

# 3.6 Configuration Aspect

Configuration must be:

- Explicit
- Version-aware
- Environment-specific
- Externalized from code

Rules:

- No hidden defaults that differ per environment.
- Configuration schema must be documented.
- Configuration must be validated at startup.

---

# 3.7 Storage Abstraction Aspect

Storage mechanisms (Postgres, SQLite, object store, etc.) must be accessed through defined adapters.

Rules:

- Domain logic must not depend on storage engine specifics.
- SQL is allowed and preferred over ORMs.
- Accessors define explicit data operations.
- Storage engines must be swappable per profile.

---

# 3.8 Async Runtime Aspect

All executable services must:

- Use async-native entrypoints.
- Separate IO from domain logic.
- Allow sync business logic inside async wrappers.

No domain logic may depend on event loop internals.

---

This section defines the cross-cutting aspects that apply to all components of the system. Aspects are mandatory constraints and must remain stable as products and subsystems evolve.