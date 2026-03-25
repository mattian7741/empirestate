# Structured logging contract

Per spec Section 3.1.1 (Correlation) and 3.1.2 (Structured Logging).

**Minimum log fields (JSON):**

| Field           | Required | Description                          |
|-----------------|----------|--------------------------------------|
| `level`         | yes      | Log level (e.g. info, error)         |
| `timestamp`     | yes      | ISO 8601                             |
| `service_name`  | yes      | Service identifier                   |
| `version`       | yes      | Service version                      |
| `correlation_id`| yes      | Global trace identifier              |
| `request_id`    | no       | Request scope identifier             |
| `tenant_id`     | no       | When in tenant context               |
| `app_domain`    | no       | When in app context                  |
| `message`       | yes      | Human-readable message               |
| `error_code`    | no       | When level is error                  |

**Rules:** No plain-text logging; no ad-hoc string concatenation; logs must never expose secrets. Format consistent across languages.
