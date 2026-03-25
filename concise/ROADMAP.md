# Roadmap

**In scope:** Platform — **OpenErgo** (backbone), **Payux** (billing). Consuming applications are **out of scope** here.

**Principle:** `../empirestate/TENETS.md` **1–6** — VCS from the start, scaffolding, lean, deploy/iterate live, billing early.

| Milestone | Goal |
|-----------|------|
| **M1 Payux** | Invoice → payment → receipt; standalone REST + client; white-label; **in production**. Commerce front (cart/SKU) may be mocked. |
| **M2 Deployment** | Reproducible repo → Docker host; Payux on pipeline; disposable infra. |
| **M3 OpenErgo** | Backbone live (transport e.g. AMQP/HTTP); pipelines proven; **Payux ↔ OpenErgo**. |
| **M4 Identity** | OTP/magic-link, sessions, roles; enables participation-first flows. |
| **M5 Ledger** | Distributed claims ledger + projection + local/sync patterns per product needs. |
| **M6 Viral** | Transactional invite pattern; data minimization; abuse bounds. |
| **M7 Targets** | Web, mobile stores, Electron, optional native — same deploy principles. |
| **M8 Platform** | Hardening, ops, observability; stable base for downstream products. |

Detail: `../empirestate/ROADMAP.md`, `../empirestate/BACKLOG.md`.
