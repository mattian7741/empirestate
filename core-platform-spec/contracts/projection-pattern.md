# Projection schema pattern (CP-3)

Projections are read-optimized tables populated from the event stream (or, in CP-3, direct writes). Spec: Section 6.2, 6.3.

## Rules

- One or more **migrations** define the table(s). Migrations are versioned SQL files run in order; state is tracked in `schema_migrations`.
- **Accessors** are one class per logical domain (e.g. `TaskAccessor`). They expose explicit methods, parameterized SQL only, and explicit row→domain mapping. No ORM.
- **Idempotency** — Writes that support idempotency accept an optional key; duplicate key returns existing row (no duplicate insert). Stored per-entity or in a dedicated table depending on need.
- **Tenant scope** — All projection tables and accessor calls are tenant-scoped (tenant_id in WHERE / unique constraints).

## Example (tasks)

- Table: `tasks` (id, tenant_id, title, description, status, created_at, updated_at, idempotency_key). Unique (tenant_id, idempotency_key) for idempotent create.
- Accessor: `getById(tenantId, taskId)`, `listByTenant(tenantId, status?)`, `insert(tenantId, title, description?, idempotencyKey?)`, `updateStatus(tenantId, taskId, status)`.
