# Error code taxonomy

Stable, namespaced error codes per spec Section 7.11 and 11.5. Used in the canonical error envelope.

**Top-level namespaces (layer-based):**

| Namespace    | Meaning                          |
|-------------|-----------------------------------|
| `AUTH`      | Authentication / identity         |
| `ENTITLEMENT` | Permission / entitlement checks |
| `VALIDATION`  | Request validation failures     |
| `CONFLICT`    | State conflict (e.g. duplicate)  |
| `NOT_FOUND`   | Resource not found               |
| `RATE_LIMIT`  | Rate limiting                    |
| `UPSTREAM`    | Upstream/dependency failure      |
| `TIMEOUT`     | Timeout                          |
| `INTERNAL`    | Internal server error            |

Domain-specific subcodes extend these (e.g. `VALIDATION.INVALID_EMAIL`). Error codes are stable across versions; messages may change.
