# ESB modeling — protocol stress test

**Purpose:** Exercise **[Empire State Build (ESB)](../empirestate/DEPLOYMENT.md#empire-state-build-esb)** as **pure membership**: **domains** (`id`) and **artifact string lists**, with **recursive** token→definition lookup. **How** / **where** are out of scope here.

---

## How we work

1. You give **protocol feedback** (and prose **outside** YAML if needed).
2. We adjust **`ideal-system.esb.yaml`**, optional **`definitions/*.esb.yaml`**, and **`EVALUATION.md`**; stable rules go to **`DEPLOYMENT.md`**.

---

## Rules for YAML in this folder

| Rule | Detail |
|------|--------|
| **Domains + strings** | **`domains`**: list of `{ id, artifacts }` where **`artifacts` is only strings**. Or **one** `{ esb, id, artifacts }` per file. |
| **No “why”** | No comments, summaries, or extra keys in `.esb.yaml`. |
| **Recursion** | Each token matches a **`domains`** row (same file or another file under **`definitions/`**) whose **`id`** equals that token. |
| **No per-member maps** | No `- id: foo` under **`artifacts`**—only `- foo`. |

---

## Layout

| Path | Role |
|------|------|
| **`ideal-system.esb.yaml`** | Layered sample: **lay root** → **application system** → **web vs store** tokens vs **OpenErgo bundles** → **worker** leaves; **`platform-capabilities`** for logical managed-style tokens. |
| **`definitions/`** | Optional: split-out **`id`** bodies when not inline as sibling **`domains`** rows (may be empty). |
| **`EVALUATION.md`** | Baseline + gaps; not ESB content. |
| **`resolution-walkthrough/`** | **End-to-end example:** one leaf (`authcode-generator`) from lay ESB → catalog → OpenErgo YAML → expanded spec → Ansible → Docker ([`resolution-walkthrough/README.md`](resolution-walkthrough/README.md)). |

**Lay vs depth:** [`ideal-system.esb.yaml`](ideal-system.esb.yaml) stays **membership-only**. Lower layers live only under **`resolution-walkthrough/`** so grammars are not mixed.

---

## Non-goals

Binding catalogs, realizers—see `empirestate/DEPLOYMENT.md`.
