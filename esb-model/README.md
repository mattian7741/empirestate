# ESB modeling — protocol stress test

**Purpose:** Exercise **[Empire State Build (ESB)](../empirestate/DEPLOYMENT.md#empire-state-build-esb)** as **pure membership**: **domains** (`id`) and **artifact string lists**, with **recursive** token→definition lookup. **How** / **where** are out of scope here.

---

## How we work

1. You give **protocol feedback** (and prose **outside** YAML if needed).
2. We adjust **`ideal-system.esb.yaml`**, **`definitions/*.esb.yaml`**, and **`EVALUATION.md`**; stable rules go to **`DEPLOYMENT.md`**.

---

## Rules for YAML in this folder

| Rule | Detail |
|------|--------|
| **Domains + strings** | **`domains`**: list of `{ id, artifacts }` where **`artifacts` is only strings**. Or **one** `{ esb, id, artifacts }` per file. |
| **No “why”** | No comments, summaries, or extra keys in `.esb.yaml`. |
| **Recursion** | Each token must match a **definition**: another document with root **`id`** equal to that token (see **`definitions/`**). Lookup rules are **platform** concern. |
| **No per-member maps** | No `- id: foo` under **`artifacts`**—only `- foo`. |

---

## Layout

| Path | Role |
|------|------|
| **`ideal-system.esb.yaml`** | Top **domains** sample (protocol only). |
| **`definitions/`** | One file per **expandable** token (`id` matches filename stem by convention in docs—tooling defines exact index). |
| **`EVALUATION.md`** | Baseline + gaps; not ESB content. |

---

## Non-goals

Binding catalogs, realizers—see `empirestate/DEPLOYMENT.md`.
