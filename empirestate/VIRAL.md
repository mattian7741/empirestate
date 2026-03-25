# Viral — User-Initiated Transactional Invite

**Scope:** A pattern for involving a **second person** in a **single, defined activity** without requiring that person to already be a user—built on the **emergent user** model in [[IDENTITY]] (participation without sign-up).

**Relationship to identity:** Invites use **minimal disclosure** and **bounded messaging** so low-friction growth stays aligned with participation-first auth elsewhere (`IDENTITY.md`).

---

## Core model

| Step | Actor | Action |
|------|-------|--------|
| 1 | User A | Creates an activity that involves another participant. |
| 2 | User A | Supplies a contact channel for User B (email or phone), if not using link-only flow. |
| 3 | System | Sends a **single-purpose** notification to B: activity + inviter identity. |
| 4 | User B | Acts via **link or code** without creating an account. |
| 5 | System | **Follow-up messages** only for **that same activity** (remind, complete). |
| 6 | User B | May **optionally** become a full user—not required. |

**Net effect:** The product coordinates people who **already have a real-world relationship**; growth is a byproduct of **utility**, not broadcast outreach.

---

## Why this pattern is appropriate

| Principle | Meaning |
|-----------|---------|
| **Conduit, not initiator** | The system facilitates an interaction A and B already chose; it does not cold-contact. |
| **One transaction** | All comms tied to **one** clearly scoped activity. |
| **Narrow, temporary use** | Recipient data used only for that flow; **no secondary** reuse. |
| **No harvesting** | No marketing lists, profiling, or enrichment off the invite path. |

---

## Operational rules

| Rule | Requirement |
|------|-------------|
| **Data minimization** | Collect only the contact detail needed to deliver the invite; no cross-reference or reuse outside the transaction. |
| **Purpose limitation** | Messages only to initiate, coordinate, remind, and **complete** that activity—no unrelated content. |
| **User attribution** | Every message names **who initiated**; the platform must not appear to cold-contact independently. |
| **Bounded communication** | Initial invite + **small, predefined** reminder cap; no open-ended drip loops. |
| **Recipient control** | Each message: simple **stop** for **that activity**; honored immediately across channels. |
| **No persistence without consent** | If B never engages, delete contact after a **short window**. If B engages but never registers, data stays **scoped to that activity** only. |
| **No identity leakage** | Do not reveal whether an address is an existing account; block probing/enumeration. |
| **Abuse prevention** | Rate limits on invites; detect/block bulk abuse and repeated targeting. |

---

## Channels

| Channel | Use |
|---------|-----|
| **Email** | Initial invite + limited reminders; clear who invited, purpose, opt-out. |
| **SMS** | **Sparing**; strictly transactional copy; **minimal** count; clear STOP/stop mechanism. |

---

## Preferred mechanism

- **Primary:** Shareable **link or code** bound to the activity.
- **Direct contact entry:** Optional convenience layer on top of link-first flow.
- After B opens the link, prefer **in-app** or **user-controlled** channels for follow-up where possible.

---

## Roadmap note

Milestone for implementing this pattern at platform level: see [[ROADMAP]] (M6). Applications may adopt subsets earlier under the same rules.
