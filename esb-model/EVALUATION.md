# ESB exercise — evaluation notes

**Objective:** Stress-test the **ESB protocol**—**groups (`id`) + string `artifacts` + recursive token definitions**—not any product.

---

## Baseline (established)

1. **No “why” in ESB** — Mechanical fields only; narrative stays here or in aspect docs.
2. **Three deploy questions** — **What** = lay ESB + definitions; **how** / **where** = descriptors + bindings + invocation (**`empirestate/DEPLOYMENT.md`**).
3. **`domains` + string `artifacts` only** — No `- id:` maps under **`artifacts`**. Tokens resolve to definitions keyed by matching **`id`** (see **`definitions/`**).
4. **`namespace` (K8s / runtime)** — Not the lay group key; lay groups use **`domain` / `id`**. Runtime **namespace** may appear only in **bindings** or lower descriptors.

---

## Protocol observations (ongoing)

- **Lookup:** Token `open-ergo-auth` → definition with `id: open-ergo-auth` (sibling **`domains`** row in `ideal-system.esb.yaml`, or split file under `definitions/`—**tooling** picks the index).
- Further structural feedback goes here until folded into `DEPLOYMENT.md`.
