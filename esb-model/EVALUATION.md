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

---

## Bottom-up layering (OpenErgo and beyond)

**Source:** function + PyPI-style deps → **library artifact** in registry → **OpenErgo component YAML** (logical component: name, **OpenErgo** network/domain, library/function, I/O bindings, keys, pre/post) → **Docker image** (library + SDK + config) as **one** deployable pathway.

**Also:** non-Docker kinds (Play, App Store, Neon, CloudAMQP, …) each need **descriptor + realizer**; lay **ESB** stays **kind-agnostic**.

**Middle:** deployment **manifest** references lower configs for compound deployments.

**Top:** ESB = **groups + string tokens** only.

### Gaps to close

- **Leaf typing** in catalog when recursion stops (**deploy kind** per token).
- **Manifest schema** vs ESB expansion boundary (what lives in **expanded spec** only).
- **Disambiguation** of “namespace”: OpenErgo network field vs lay **`domains`**.

Folded into `empirestate/DEPLOYMENT.md` § **Bottom-up layers**.
