# ESB exercise — evaluation notes

**Objective:** Stress-test **ESB** as the **full artifact-resolution stack** (see `empirestate/DEPLOYMENT.md`), with a minimal **lay** file and polished **examples** at lower layers.

---

## Baseline (established)

1. **ESB = whole stack** — Lay membership **plus** interim layers **plus** concrete binding/schema/seed data—not only `ideal-system.esb.yaml`.
2. **Three deploy questions** — **What (opaque)** in lay corpus; **what (refined)** in interim descriptors; **how/where** in bindings + realizers.
3. **Lay file** — **`domains`**, string **`artifacts`** only.
4. **Disambiguation** — “Namespace” in OpenErgo/bus/runtime docs vs **lay `id` / domain** — see `DEPLOYMENT.md` gaps.

---

## Canonical examples

| Artifact | Path | Role |
|----------|------|------|
| **`user-data-store`** | [`ideal-system.esb.yaml`](ideal-system.esb.yaml) + [`resolution-matrix/`](resolution-matrix/README.md) | **Data plane:** opaque store → relational/Postgres/schema/seeds. |
| **`authcode-generator`** | [`resolution-walkthrough/`](resolution-walkthrough/README.md) | **Compute:** OpenErgo → Ansible → Docker. |

---

## Protocol observations (ongoing)

- Further structural feedback goes here until folded into `empirestate/DEPLOYMENT.md`.
