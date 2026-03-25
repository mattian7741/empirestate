# Homelab Reference Implementation Notes — Extract for Consistency Only

This addendum is derived from **rough notes and previous work** in `~/project_base/ref_impl/homelab`. It is for **reference and consistency-checking only**.

**Authoritative homelab framing:** **AA_ADDENDUM_01** (Homelab in the Core Platform Reference Frame, including Amendments and Baseline Strategy) is the source of truth. The **current plan** (REQUIREMENTS, Sections 1–12) is not amended by this document. If anything here conflicts with AA_ADDENDUM_01 or the iteration plan, the addendum and plan **win**. Do not merge ref_impl material into the strategy without an explicit amendment.

**Caution:** Ref_impl content may be obsolete or less elegant than the current plan. This extract elevates only principles that **align** with AA_ADDENDUM_01 and the platform; it does not promote implementation details or desired-state notes that would dilute or contradict the baseline.

---

## 1. Source Material

- **mattian/** — MISSION.md, OVERVIEW.md, stories/ (epics 0–9, T-series): plans and docs only; automation lives in infra.
- **HomeServer/code/** — cicd_helloworld, infra.bak (README: server standards, GitOps flow, observability, backups).
- **HomeLab/** — laptop/shell customization (out of scope for platform tenets).

**Running implementation:** AA_ADDENDUM_01 states current deployment location as `~/nirvana/dev/infra/README`. Ref_impl is a separate collection of notes; paths and host names in ref_impl are **not** canonical.

---

## 2. Principles Extracted (Aligned with Current Plan)

The following are distilled from ref_impl and **reinforce** AA_ADDENDUM_01 and the platform. They are not new mandates; they restate or clarify what is already in the addendum.

- **Idempotence.** Reruns don’t drift; apply automation repeatedly with no unintended state change. (Aligns with AA_ADDENDUM_01 baseline, Ansible-first, reproducible infra.)
- **Rollback.** Uninstall/destroy leaves no trace; rollback is codified (e.g. in Ansible), not ad-hoc. (Aligns with platform “roll forward, never drift” and DR.)
- **Separation of concerns.** Plans/docs vs automation can live in separate repos or trees (e.g. “this repo = plans/docs; infra repo = automation”) to reduce blast radius and tie decisions to code. (Aligns with isolation of concerns in AA_ADDENDUM_01.)
- **GitOps.** No manual shell changes; everything lands in Git first. Pushes drive deployment; server is read-only for Git where applicable. (Aligns with “everything in Git,” deployment aspect, DR mirroring.)
- **Iterate fast.** Infra-first, but don’t block feature delivery. (Aligns with “rapid iteration” and convergence approach in AA_ADDENDUM_01.)
- **Self-sufficiency and DR.** Reproducible IaC; re-provision from code; offsite mirrors for Git and data. (Aligns with “no cloud dependency,” “reproducible operations,” and DR in AA_ADDENDUM_01.)
- **Observability and deployment as pervasive.** Monitoring, backups, and deployment are part of the baseline; not afterthoughts. (Aligns with “Deployment and observability as pervasive aspects” in AA_ADDENDUM_01.)
- **Substrates as commodities.** Messaging (e.g. RabbitMQ), databases (e.g. Postgres), ingress (e.g. Traefik), and optional nodes (e.g. Raspberry Pi) are implementation choices behind roles (Ephemeral Router, CEL, etc.). Refer to roles in the spec; list concrete implementations as profile-specific. (Aligns with B.1–B.3, role-based naming.)

---

## 3. What Is Not Elevated

To avoid amending the current plan with less mature or conflicting information:

- **Specific paths, host names, or user names** in ref_impl (e.g. `/Users/matthewhansen/srv`, `server1`) are implementation detail. They are not promoted to canonical strategy. AA_ADDENDUM_01 already defines baseline location and profile.
- **Tensions already tracked in AA_ADDENDUM_01 § C** (image pinning vs :latest, plaintext secrets vs production posture, LAN vs public, soft vs hard enforcement) are not “resolved” by ref_impl notes. Ref_impl may describe a desired state (e.g. “no plaintext secrets,” “images pinned by digest”); that remains a tension until explicitly amended.
- **Golden Rules or conventions** in infra.bak (e.g. “Everything in Git,” “Containers ephemeral; data persistent”) are consistent with the plan but are **operating conventions** for that ref_impl, not new strategy mandates. They need not be duplicated into AA_ADDENDUM_01 unless they fill a gap.
- **Future tech list** (e.g. “Postgres, RabbitMQ, Traefik, Authelia, LLM”) in OVERVIEW is scope at a point in time. Strategy is expressed as **roles** (CEL, Ephemeral Router, Object Store, etc.); ref_impl does not add new canonical tooling.

---

## 4. Use of This Addendum

- **Consistency check:** When adding or changing homelab-related work, ask whether it respects the principles in §2 and avoids elevating the items in §3.
- **Do not:** Overwrite AA_ADDENDUM_01, change REQUIREMENTS, or introduce new baseline definitions from ref_impl without an explicit amendment.
- **Convergence:** Homelab implementation should continue to converge per AA_ADDENDUM_01 (“slowly converge on correctness,” “incremental cleanup and refactor”). Ref_impl notes can inform that convergence but do not replace the amendment process.

---

*Document derived from ~/project_base/ref_impl/homelab. Authoritative homelab and amendment source: AA_ADDENDUM_01.*
