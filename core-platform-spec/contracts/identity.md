# Identity model (CP-2)

Tenant-scoped identity per spec Section 3.3. Consumer-first default; principals are email-based and may be shadow (no signup yet).

---

## Partition model

Commuting identity is foundational: it enables shared infrastructure and shared identities across domains. Partitioning is achieved in two ways:

| Partition type | Description | When to use |
|----------------|--------------|-------------|
| **Logical silos** | Domain discriminator (e.g. `tenant_id`, `app_id`). Same or separate identity runtime instances. Same core libraries. | **Default.** |
| **Physical silos** | Independent datastore; same core libraries; separate runtime instances. | Regulation/policy motivated (multitenant). Opt-in. |

**Commuting across domains:** Identity can flow across domains by **white-listing domain discriminators** for specific domains. When configured, principals in domain A are recognized in domain B. Base case: identity data is not shared across tenants (strict tenant scope). White-listing relaxes this per configuration.

---

## Tenant

- `id` — stable tenant identifier (domain discriminator in logical-silo model)
- `name` — human-readable (optional)
- `created_at` — ISO timestamp
- Consumer-first: one tenant per deployment is a valid default

## Principal

- `id` — stable principal identifier (e.g. UUID or opaque id)
- `tenant_id` — required; principal belongs to exactly one tenant in the base case
- `email` — optional; used for shadow principal resolution
- `is_shadow` — true when principal is identified by email only (e.g. invitee who has not signed up)
- `created_at` — ISO timestamp

In the base case, identity data is not shared across tenants; all operations are evaluated in tenant scope. When discriminators are white-listed for commuting, principals may be recognized across the configured domains.

## Role model (2 roles)

Two roles per tenant (declarative; map to entitlements later):

| Role       | Purpose                          |
|-----------|-----------------------------------|
| `consumer` | Default for end users; can use app features within tenant |
| `admin`    | Tenant/admin scope; can provision and manage tenant |

Roles are defined per tenant. Role definitions must not encode application logic; entitlements (later) govern permissions.

## Bridge entities

Cross-tenant identity (physical-silo or cross-domain commuting) requires explicit configuration. When needed, bridging must be explicit, revocable, and auditable (spec 3.3.3).
