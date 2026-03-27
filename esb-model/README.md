# ESB modeling — ideal system exercise

**Purpose:** Capture **product / system end-state intent** in [Empire State Build (ESB)](../empirestate/DEPLOYMENT.md#empire-state-build-esb) form, then **stress-test** whether that grammar is sufficient to drive future **expanded spec + convergence** (realizer not defined here).

This folder is **experimental**: YAML shape and field names follow the deployment doc **until** the exercise proves we need changes.

---

## How we work

1. **You** describe the ideal system in prose (capabilities, major pieces, dependencies, boundaries).
2. **We** translate that into ESB: logical **namespaces**, **components**, **`depends_on`** (logical artifact names—databases, buses, sibling services, etc.), reusing **one ESB** across environments (**environment** only at invocation—not in these files).
3. We **revise** `ideal-system.esb.yaml` (or split slices into extra files under `slices/` if it grows large).
4. **Gaps**—things ESB cannot say cleanly—get noted in **Evaluation** (below) and may feed back into `empirestate/DEPLOYMENT.md`.

**Rules of the exercise**

- **Logical only** in YAML: no URLs, no cloud IDs, no pins unless you explicitly want a reproducible slice documented.
- **Unique logical names** for every artifact the model references (components + `depends_on` targets).
- **`esb`**: grammar/document version (fine to keep `0.1` while iterating).
- **Component `version`**: optional; omit or use `latest` / channels unless a pin is intentional.

---

## Layout

| Path | Role |
|------|------|
| **`ideal-system.esb.yaml`** | Rolling **ideal end-state** model. **First case:** Circuit Leagues (PWA, auth/session v1). |
| **`slices/`** | Optional: one file per major namespace/stack if the single file gets unwieldy. |
| **`EVALUATION.md`** | Running notes: fit, gaps, fields we wish we had (created when first issues appear). |

---

## Grammar authority

Single source for ESB meaning: **[`empirestate/DEPLOYMENT.md`](../empirestate/DEPLOYMENT.md)** (logical vs concrete, environment injection, `depends_on`, optional component version).

---

## Non-goals (for this folder)

- **Binding catalog** (concrete resolution per `namespace` + environment)—out of scope here.
- **Ansible / Docker / execution**—out of scope; the exercise tests **intent capture**, not a runner.

---

*Next step:* Describe the ideal system (top-level capabilities and major runnable / logical pieces); the model file will be updated to match.
