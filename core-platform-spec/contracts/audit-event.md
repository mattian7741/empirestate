# Audit event model (CP-2)

Sensitive operations emit immutable audit events. Spec: Section 3.1.4.

## Required fields

- `actor` — who performed the action (principal id or system)
- `target` — what was affected (e.g. tenant id, principal id)
- `action` — verb (e.g. `tenant.provisioned`, `principal.created`)
- `timestamp` — ISO 8601
- `correlation_id` — trace id for the operation

## Optional

- `details` — structured payload (no secrets)
- `request_id` — request scope id

## Rules

- Audit is distinct from logging.
- Store depends on profile (stdout, file, or DB in later iterations).
- Events must be immutable; do not mutate after emit.

## Example (JSON, stdout)

```json
{
  "actor": "system",
  "target": "tenant:acme",
  "action": "tenant.provisioned",
  "timestamp": "2025-02-26T12:00:00.000Z",
  "correlation_id": "uuid-here"
}
```
