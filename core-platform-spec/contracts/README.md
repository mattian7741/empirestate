# Shared Contracts

Canonical definitions for cross-boundary contracts. Implementations:

- **TypeScript:** core-platform `packages/core` — BFF and frontend.
- **Python:** core-ergo — Ergo workers.

Both must conform to these specs.

| Contract | Spec reference | Document |
|----------|----------------|----------|
| **Envelope** | Section 7.3, 7.4 | Request/response and error envelope; types in core-platform `envelope.ts` / Python equivalents |
| **Error codes** | Section 7.11, 11.5 | [error-codes.md](error-codes.md) |
| **Logging** | Section 3.1.1, 3.1.2 | [logging-schema.md](logging-schema.md) |
| **Identity (CP-2)** | Section 3.3 | [identity.md](identity.md) |
| **Audit event** | Section 3.1.4 | [audit-event.md](audit-event.md) |
| **Projection pattern (CP-3)** | Section 6.2, 6.3 | [projection-pattern.md](projection-pattern.md) |

Schema versioning and backward-compatibility rules are defined in spec Section 11.
