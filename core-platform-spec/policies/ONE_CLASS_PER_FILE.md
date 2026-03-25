# One public class per file

**Spec reference:** Section 1.12, 10.9.

## Rule

- Each file should expose **one primary public construct** (class or module).
- Private helpers may exist but must remain internal.
- File names must match the primary class or module purpose.
- Modules must have explicit public surfaces.

## Rationale

- Preserves structural clarity.
- Reduces coupling and accidental dependencies.
- Aligns with composition-over-inheritance and organic hierarchy growth.

## Preference

- Flat structures over deeply nested folder hierarchies.
- Explicit public surfaces over implicit exports.

This rule is mandatory across TypeScript/Node, Python, and BYOL components. Deviations require architectural review.
