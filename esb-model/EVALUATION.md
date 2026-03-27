# ESB exercise — evaluation notes

Running log of **grammar stress** and **product/architecture questions** surfaced while modeling. Resolving them can drive updates to `empirestate/DEPLOYMENT.md` or this model.

## Circuit Leagues (v1 — auth/session)

- **Client in ESB:** `circuit-leagues-pwa` is included as a logical component so the dependency graph is complete; convergence tooling may treat it differently from JVM/containers (static hosting, CI).
- **Transport naming:** Model uses **`openergo-transport`** at the logical layer; repo history also references **Hypergo** as an Ergo-family implementation—catalog/bindings map the logical id to whatever runtime you run.
- **“2FA without first factor”:** Product language uses *second factor*; technically this is **one-factor** possession-of-channel OTP—fine for intent; security docs may tighten vocabulary later.
- **Abuse surface:** Unauthenticated “send OTP” is high-risk without rate limits / friction; not captured in ESB (operational/policy layer).
- **Email vs phone identity:** User-emergent model may need **merge/link** rules if the same human uses both—identity aspect work, not ESB.
- **Logout ordering:** Client drops session before server ack can create **brief desync**; acceptable for some products, worth explicit UX/policy later.
