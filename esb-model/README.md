# ESB modeling — protocol stress test

**Purpose:** Exercise whether **[Empire State Build (ESB)](../empirestate/DEPLOYMENT.md#empire-state-build-esb)** can express **end-state “what”** with **minimal surface**, without pretending to capture **why** or hand-waving execution. Product specifics are **out of scope** for the YAML; they inform updates only indirectly.

**Authority:** [`empirestate/DEPLOYMENT.md`](../empirestate/DEPLOYMENT.md).

---

## How we work

1. You give **protocol/grammar feedback** (and optionally a **vignette** in prose elsewhere—not in the ESB file).
2. We adjust **`ideal-system.esb.yaml`** and **`EVALUATION.md`**, then fold stable rules back into **`DEPLOYMENT.md`**.

---

## Rules for files in this folder

| Rule | Detail |
|------|---------|
| **Lean YAML** | Only mechanical fields: **`esb`**, **`slices`**, **`namespace`**, **`components`**, **`id`**, optional **`version`**, and **`depends_on`** only when justified. |
| **No “why”** | No `summary`, `product`, `meta` strategy, `#` comments, or docstrings in ESB YAML—those belong in **`EVALUATION.md`**, aspect docs, or applications. |
| **Sparse `depends_on`** | Default **none**. Namespace + binding imply transport/capabilities (OpenErgo pattern); do not list the bus per component. Add `depends_on` only for **unambiguous** logical attachments the catalog cannot infer (rare). |

---

## Layout

| Path | Role |
|------|------|
| **`ideal-system.esb.yaml`** | Minimal topology sample for grammar stress (not a product spec). |
| **`slices/`** | Optional split if the sample grows. |
| **`EVALUATION.md`** | Baseline rules + protocol gaps; **not** ESB content. |

---

## Non-goals

- Binding catalogs, concrete IDs, realizers (Ansible, etc.)—see `DEPLOYMENT.md`.
