# EmpireStack — Tenets

**Enumerated constraints.** **Order matters:** **1** is the most significant; each later tenet is secondary to all before it. Lower-numbered rules **win** when guidance seems to collide.

---

1. **Source control from the start.** All work that constitutes the product—including specification material that drives implementation—lives in **version control** from day one. **No “pre-repo” phase** that holds real delivery or drift-prone state; history, review, and reproducibility are **non-negotiable**.

2. **Architecture and strategy are scaffolding.** The documented aspects, boundaries, and long-term shape define where everything **slots**. Implementation shall **agree** with that whole—no local shortcuts that fight the stack you are building toward.

3. **Lean first.** Deliver the **smallest** scope that achieves the iteration’s stated outcome while remaining **fully consistent** with (2). Prefer minimal change over breadth; treat requirements as **provisional** until validated in use.

4. **Production-capable deployment defines completion.** Work is **unfinished** until it runs in a real, production-grade environment—not merely merged or demoable locally.

5. **Iterate from deployed systems.** Use **short loops** that repeatedly improve what is **already live**; each iteration should yield something **runnable** that can be deployed or is deployed, not a growing pile of unreleased surface.

6. **Billing early.** Establish a credible **revenue path** before the sellable catalog or product surface is complete—this stack assumes money can flow while the product is still thin.

7. **This corpus is the specification authority.** What counts as “specified” for EmpireStack lives here.

8. **Generated documentation is subordinate.** End-user and developer docs shall be **rendered views** from this corpus—not parallel, co-equal specifications.

9. **One primary home per fact.** Elsewhere **link**; the same story told twice without new signal is **documentation debt**.

10. **State facts concisely.** Expand only where omitting detail would mislead.

11. **Prefer structure over prose** for decisions: tables, invariants, explicit **does / does-not** boundaries—unless a tradeoff truly needs narrative.

12. **Aspect documents are dimensions.** Truth is **composed** from isolated aspect docs; avoid one undifferentiated mega-spec.

13. **Interpret inputs into coherent spec.** Submissions inform the record; the record is **synthesized**, not a verbatim transcript.

14. **Layer documentation by role.** `empirestate/` holds compact **product truth**. Per-application **`applications/<AppName>/FUNCSPEC.md`** holds **functional** behavior and UX (English); optional **`WORKFLOW.md`** holds **ordered** multi-actor steps (text sequence, not diagram syntax); **`DESIGN.md`** (or additional scoped design files in that folder) holds **slice-specific build contracts**. `funcspec/` and `design/` at the repo root are redirect hubs only. Do not repeat roadmap or backlog storytelling in every file unless that file adds **new** signal—**pointer + delta** only.
