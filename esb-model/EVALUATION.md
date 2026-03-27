# ESB exercise — evaluation notes

Running log of **grammar stress** and **product/architecture questions** surfaced while modeling. Resolving them can drive updates to `empirestate/DEPLOYMENT.md` or this model.

## CircuitLeagues (M1 — auth/session)

- **Client in ESB:** `circuitleagues-web` is a logical component so the dependency graph is complete; convergence tooling may treat it differently from containerized services (static hosting, CI).
- **Transport naming:** Model uses **`openergo-bus`** as the logical OpenErgo transport; catalog/bindings map the id to the real runtime. (Repo docs may also reference other Ergo-family names—keep one logical id in ESB.)
- **“2FA without first factor”:** Product language uses *second factor*; technically this is **one-factor** possession-of-channel OTP—fine for intent; security docs may tighten vocabulary later.
- **User-emergent provisioning:** If the pipeline **creates or enriches user rows before** successful OTP verify, watch for **spray**, **deliverability abuse**, and **identifier enumeration**. Prefer litigating **what persists on send vs on verify** in design, not only in ops.
- **Abuse surface:** Unauthenticated “send OTP” still needs rate limits / friction; not captured in ESB (operational/policy layer).
- **Email vs phone identity:** User-emergent model may need **merge/link** rules if the same human uses both—identity aspect work, not ESB.
- **Logout ordering:** Client drops session before server ack can create **brief desync**; acceptable for some products, worth explicit UX/policy later.
