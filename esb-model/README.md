# ESB modeling — protocol stress test

**Purpose:** Exercise **Empire State Build (ESB)** end-to-end. In this repo, **ESB** means the **full resolution stack** (lay → interim → concrete), not only the top file—see [`empirestate/DEPLOYMENT.md`](../empirestate/DEPLOYMENT.md#empire-state-build-esb).

---

## How we work

1. You give **protocol feedback** (prose **outside** `.esb.yaml` when needed).
2. We adjust **`esb-model/`** and fold stable rules into **`empirestate/DEPLOYMENT.md`**.

---

## Rules for lay `.esb.yaml`

| Rule | Detail |
|------|--------|
| **Domains + strings** | **`domains`**: `{ id, artifacts[] }` where **`artifacts`** are **strings only**. Or one root **`id`** + **`artifacts`**. |
| **No “why”** | No comments or narrative keys in **lay** files. |
| **Recursion** | Token → matching **`id`** row (lay) or **catalog** (non-lay). |
| **No per-member maps** | No `- id: foo` under **`artifacts`**—only `- foo`. |

---

## Layout

| Path | Role |
|------|------|
| **`ideal-system.esb.yaml`** | **Minimal lay** sample: **`user-data-store`** only. |
| **`resolution-matrix/`** | **Data-plane** matrix: `user-data-store` lay → Postgres/schema/seeds ([`resolution-matrix/README.md`](resolution-matrix/README.md)). |
| **`resolution-walkthrough/`** | **Compute** E2E: OpenErgo → Ansible → Docker ([`resolution-walkthrough/README.md`](resolution-walkthrough/README.md)). |
| **`EVALUATION.md`** | Gaps and notes. |

**Lay vs lower layers:** **`.esb.yaml`** = **top of ESB** only; depth lives in **`resolution-matrix/`**, **`resolution-walkthrough/`**, and future catalog/manifest specs.

---

## Non-goals

Production binding catalogs and realizers—see `empirestate/DEPLOYMENT.md`.
