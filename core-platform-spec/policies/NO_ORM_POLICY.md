# No ORM policy

**Spec reference:** Section 6.8, 10.10.

## Rule

ORM frameworks are **prohibited as the primary abstraction**. ORM may be used **behind** the explicit data accessor interface as an implementation detail—inside the blackbox. The application never contracts with the ORM; it contracts with the accessor.

## Rationale

- ORMs have historically been flaky, heavy, and hard to maintain—hidden performance characteristics, implicit query generation, runtime abstraction leakage.
- Most database integrations cover a finite number of discrete interactions; ORMs are often unnecessary at the architectural level.
- When the accessor defines the contract and the implementation is blackbox, ORM usage (if any) is an implementation detail, not an overarching architectural decision.
- ORMs may have evolved; clean use behind the interface may be tenable. The prohibition is on ORM as the visible abstraction, not on ORM as hidden implementation.

## Required pattern

- **Explicit data accessor classes** — one per logical domain area. The accessor is the contract.
- **Accessor as blackbox** — The implementation behind the accessor may use raw SQL, an ORM, or stored procedures. Callers see only the accessor interface.
- **Explicit mapping** — rows to domain structures; no automatic mapping leaking across the accessor boundary.
- **No automatic lazy loading** or implicit relationships visible to callers.
- Clear separation between query logic and domain logic.

## Example (conceptual)

```
UserAccessor
  get_user_by_id(...)
  get_users_by_tenant(...)
  insert_user(...)
  update_user_state(...)
```

Accessors are injected via capability interfaces. If the datastore or implementation (including ORM) changes, replace the accessor implementation while preserving the contract.

Stored procedures are allowed when justified (performance, governance, atomicity) but must not obscure domain rules.
