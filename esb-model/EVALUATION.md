# ESB exercise — evaluation notes

Running log of **grammar fit**, **assumptions**, and **open product/engineering questions** while modeling CircuitLeagues.

---

## CircuitLeagues M1 — auth & session (2026-03-27)

### Captured in `ideal-system.esb.yaml`

- Two **logical namespaces**: `circuit-leagues` (PWA + consumption API), `open-ergo` (gateway, generators, senders, orchestrator, persister).
- **Logical artifacts**: shared **`circuit-leagues-auth-datastore`** (user/profile + OTP + session material at logical level—physical layout TBD), **`open-ergo-bus`**, **`email-delivery`**, **`sms-delivery`**.
- **Read vs write path** expressed implicitly via which components exist (consumption API vs bus-mediated writers).

### Challenges on the narrative (not blocking the exercise)

1. **“Second factor only”** — Operationally this is **identifier + channel OTP** (proof of reachability). That is a normal passwordless/magic-link family pattern; the risk set is **account enumeration**, **OTP brute force**, and **throttling**—uniform API responses and rate limits matter more than the naming.
2. **Logout via async bus → persister** — **Session may remain valid on the server** until the persister runs. For competitive security, pair client-side clearance with **short session TTL**, **server-side revocation list**, or a **synchronous invalidation** path if you need immediate server kill.
3. **Client session storage** — **httpOnly, secure cookies** beat **localStorage** for session tokens under typical XSS threat models; ESB does not encode this—belongs in **design** of `circuit-leagues-consumption-api` / PWA.
4. **OTP lifecycle** — Single-use, short expiry, constant-time verify, and lockout policy are **implementation** concerns; the ESB only names **`authcode-generator`** and the datastore.

### ESB grammar gaps noticed

- **No first-class “workflow step” or sequence** — The listed order (landing → gateway → generator → sender → verify) is **prose + architecture**, not encoded. Acceptable for M1 if we accept ESB as **topology**, not **choreography**.
- **No “channel” dimension** — Email vs SMS is **routing inside** `authcode-generator` / bus message types / sender subscription—could add logical artifacts later if we need to model them separately.
- **Hosting for PWA static assets** — Omitted as a dependency; can add **`circuit-leagues-web-host`** (or similar) when static hosting is its own deployable.

### Next modeling increments

- Add **web host** / CDN logical artifact if deployment splits **PWA shell** from **API** origin.
- Split **`circuit-leagues-auth-datastore`** if product truth later separates **profile** vs **session** stores at the **logical** layer.
