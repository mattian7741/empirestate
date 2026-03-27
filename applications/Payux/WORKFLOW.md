# Payux — end-to-end workflows (text sequence)

**Purpose:** One-page, **text-only** ordering of actors and messages for the main Payux use cases. **Not** a diagram language (no Mermaid): it uses **numbered steps** plus optional `Source → Destination: message` lines to read like a sequence chart.

**Normative behavior:** [FUNCSPEC.md](FUNCSPEC.md). **APIs and persistence:** [DESIGN.md](DESIGN.md).

---

## Participants (canonical names)

Use these names consistently in steps and in code reviews so “who calls whom” stays unambiguous. Deployments may **collapse** roles into one service (e.g. MerchantHost and CartService share a process); the **logical** split still matters for security and amounts.

| Name | What it is |
|------|------------|
| **Payer** | Person (or automated delegate) who pays; uses a **browser** or **native client**. Does not set `amount_due`. |
| **MerchantHost** | Host application: catalog/conversion UX, account, entitlement **provisioning** after payment. Owns “what product X means,” not the payment rail. |
| **CartService** | **Payment integrator** backend: owns basket totals (with **SKU stack**), builds **invoice** (`amount_due`, currency, opaque metadata, return URLs). Only **CartService** (or another trusted integrator server) should call Payux to start checkout—not the Payer’s device alone. |
| **Payux** | White-labeled gateway: payment **session**, **checkout shell**, ledger **mint**, **receipt** issuance. |
| **Processor** | Pluggable rail behind Payux (**blackbox**): hosted payment UI, saved instruments, recurring charges, **webhooks** to Payux. Never exposed by name on Payux’s public integrator API. |

**Typical mapping:** `MerchantHost` serves the “Buy” CTA and product context; **CartService** finalizes pricing and talks to Payux. If your deployment uses a single backend, treat it as both **MerchantHost** and **CartService** in the same runtime.

---

## Notation

- **Numbered steps** narrate the use case in order.
- Indented lines **`A → B: …`** describe a single message or visible handoff (read left-to-right as “A sends to B”).
- **Optional branches** are described in prose (e.g. push vs pull for receipt delivery) rather than with a separate diagram syntax.

---

## How client, server, and processor integrate (sync vs async)

**Short answer:** The full “user pays → money captured → receipt exists” path is **not** one synchronous HTTP request/response. It is **several** HTTP exchanges across three zones—**integrator server**, **Payer client**, **Payux**, **Processor**—plus at least one **asynchronous** leg (**Processor → Payux** settlement notification). Exact URLs and payloads are in [DESIGN.md](DESIGN.md); this section is about **coupling in time**.

### Zones

| Zone | Typical processes | Role in the protocol |
|------|-------------------|----------------------|
| **Integrator server** | **CartService** (and often **MerchantHost** API) | Builds **invoice**; calls Payux **server-to-server** to start checkout; later **observes receipt** (poll, inbound webhook from Payux, or trust redirect parameters and then verify server-side). |
| **Payer client** | Browser or app used by **Payer** | Renders merchant UI; receives **handoff** (redirect URL or client token); loads **Processor** or Payux checkout UI; **does not** authoritatively set `amount_due`. |
| **Payux** | Payux API + checkout shell + webhook ingress | Creates **session** with **Processor**; hands off to client; **waits** for **verified** settlement from **Processor**; mints **receipt**. |
| **Processor** | Vendor APIs + hosted/embedded UI | Confirms payment method and settlement; notifies Payux **asynchronously** (webhook) when a charge succeeds or fails. |

### What is synchronous (typical OTP / initial subscription)

These steps usually complete inside **one integrator→Payux request** (or another bounded server chain), with a **single HTTP response** back to **CartService**:

1. `CartService → Payux`: HTTP **request**—e.g. create payment session from **invoice** (see `DESIGN.md` §3).
2. Inside that handling, Payux (adapter) often calls **Processor** APIs (still **synchronous** from CartService’s point of view: CartService is waiting on Payux).
3. **Payux → CartService**: HTTP **response**—returns **handoff** for the client: e.g. `redirect_url`, or `client_handoff_token` / client secret for embedded UI, plus correlation ids.

So: **“Start checkout”** is normally **one request/response** between **CartService** and **Payux**, but that response only means “session exists, here is how the **Payer client** continues”—**not** “payment is finished” and **not** “receipt exists.”

### After session creation: separate HTTP transactions for pay + settlement

After the handoff returns to **CartService**, checkout continues in **new** HTTP requests (each request/response may itself be synchronous, but they are **not** the same connection or the same integrator→Payux call). The following are always outside the single “start checkout” response:

1. **CartService → Payer client**: Merchant app sends handoff (JSON, redirect instruction, or HTML with auto-redirect).
2. **Payer client → Payux** (and/or **→ Processor**): Follow redirect chain or mount embedded **Processor** UI—**additional** requests, possibly many (assets, 3DS, wallet UI).
3. **Processor**: User completes payment; **Processor** commits settlement on its side—**no guarantee** this finishes in the same browser session timing as step 2 from Payux’s perspective beyond UX expectations.

**Settlement visibility to Payux** is **asynchronous**: **Processor** calls Payux on a **server-to-server webhook** (e.g. `POST` to Payux’s `/api/v1/webhooks/payments`). That webhook is **not** the browser and **not** the same HTTP connection as `CartService → Payux` session creation.

4. **Payux** (on webhook): Verifies signature, normalizes event, **mints ledger + receipt** (idempotent).

### When does the **receipt** exist relative to HTTP?

- The **receipt** is created **after** Payux accepts a **verified** settlement signal from **Processor** (webhook path in M1-style designs), not in the first “create session” response.
- Therefore **CartService** must **not** assume a receipt in the same response as step “start payment.” It learns success via one or more of:
  - **Payer** lands on **success_url** with **stable correlation** (`invoice_id`, etc.); **CartService** then **GET** receipt by id / invoice (`DESIGN.md` §3)—may **poll** until Payux has finished webhook processing.
  - **Payux → CartService** outbound **integrator webhook** or message (if configured)—also **async** relative to the Payer’s last click.
  - **Polling** only from server (recommended pattern with correlation from redirect).

So: **at least two logical phases**—(a) **synchronous session create**, (b) **async settlement + mint**—always separate the “handoff” response from the “receipt available” moment.

### Renewals (subscription)

- Often **no Payer client** and **no new CartService→Payux “start”** call for each renewal.
- **Processor** charges on schedule → **`Processor → Payux` webhook** → **Payux** mints **new receipt** → **CartService** notified per integration contract (**async** end-to-end from the subscriber’s point of view).

### Summary table

| Interaction | Usually sync? | Same single HTTP request as “user clicked Pay” on merchant? |
|-------------|----------------|--------------------------------------------------------------|
| `CartService → Payux` create session | Yes (one req/resp pair) | This **is** the server’s “start checkout” call—not the browser’s Pay click unless the browser only proxied to CartService. |
| `Payux → Processor` (adapter API, during above) | Yes, inline on Payux server | N/A (not Payer’s browser). |
| `Payux → CartService` handoff response | Same response as row 1 | Still **no** receipt yet. |
| Payer browser checkout / 3DS / wallet | Many requests; user-paced | **No**—separate from CartService↔Payux session creation. |
| `Processor → Payux` settlement webhook | **Async** (later, separate connection) | **No**. |
| Receipt available to **CartService** | After webhook + mint | **No**—not the same request as session create. |

---

## OpenErgo choreography (message bus, two-plane model)

**Context:** EmpireStack uses **OpenErgo** as the backend integration substrate—HTTP gateway ↔ bus, declarative components, idempotent consumers ([`empirestate/OPENERGO.md`](../../empirestate/OPENERGO.md)). This section maps Payux’s **sync/async** reality to a **bus-friendly** collaboration without putting card data on the broker and without pretending one client POST returns a **receipt**.

### What the bus is good for (and what PCI cares about)

- **PCI scope** is about **account data** (PAN, sensitive authentication data). **Queue messages** can carry **correlation ids**, **`invoice_id`**, **`receipt_id`**, **currency/amounts fixed by trusted servers**, **opaque metadata**, **entitlement ids**—and must **not** carry **PAN/CVC** or other raw card payloads.
- **Amount authority:** the browser must not be the source of truth for **`amount_due`**. Ingress payloads are **checkout intent** (e.g. cart/session/principal refs), not “here is the final charge.” **CartService** builds the **invoice** server-side before Payux sees it.

So: **message-driven Payux is compatible with PCI** when the **Processor** still hosts or tokenizes card UI and the bus only carries **business/audit** envelopes—as in [DESIGN.md](DESIGN.md) **opaque metadata** rules.

### Why one straight pipe “HTTP → bus → … → receipt in one response” still fails

Even with no card data on the bus:

1. **Checkout handoff** (redirect URL or client token) must reach the **Payer client** while the user is waiting—see **§ How client, server, and processor integrate** above. That is **not** the same lifecycle as “publish **ReceiptMinted** later.”
2. **Receipt** exists only after **Processor → Payux** settlement (typically **webhook**), i.e. **async** to “start checkout.”

Therefore choreography splits into **two planes** below.

### Plane 1 — Interactive checkout (session + handoff)

**Goal:** Trusted server obtains a **payment session** and returns **handoff** to the client; **no receipt** yet.

Illustrative OpenErgo-style flow (routing keys are examples only; **manifests define truth**):

1. **Ingress:** **Payer client** hits **MerchantHost/CartService HTTP API**, *or* OpenErgo **HTTP gateway** maps a payload to a bus message such as `cart.checkout.requested` with **only** intent fields (`correlation_id`, cart/session refs, **no** authoritative `amount_due` from browser alone).
2. **Cart worker** consumes message, loads basket, computes totals, builds **invoice** (see [FUNCSPEC.md](FUNCSPEC.md)).
3. **Session creation** (recommended as **direct server-to-server** for simplicity): `CartService → Payux` HTTP **create session** OR an OpenErgo component that performs the same call internally. Either way, Payux returns **handoff** synchronously to that call chain.
4. **Reply path to browser:** Cart/Cart API responds to the **original HTTP request** with handoff JSON, **or** a correlated **private reply**/short-lived channel—**not** by waiting on a **ReceiptMinted** queue event in that same round trip.

Optional: publish **`InvoiceSubmitted`** / **`CheckoutSessionCreated`** to the bus for **analytics, audit, or secondary consumers**—orthogonal to returning the handoff to the browser.

**Rule:** OpenErgo **excels** at fan-out and retries here, but **cannot** replace returning **`redirect_url` / client secret`** to the waiting client without **another** synchronous or push channel (HTTP response, WebSocket, etc.).

### Plane 2 — Settlement and downstream saga (bus-native)

**Goal:** After Payux **mints** a **receipt**, propagate **proof of payment** to cart and host **asynchronously**.

1. **Processor → Payux:** HTTPS **webhook** (not the Payer’s browser, not the commerce bus). Payux **verifies**, **dedupes** processor event, **mints receipt** ([DESIGN.md](DESIGN.md) §2.4–2.5).
2. **Payux → bus:** Publish **`billing.receipt.minted`** (illustrative key) with envelope: `receipt_id`, `invoice_id`, settlement amount/currency, `correlation_id`, processor-safe refs, metadata echo—**idempotent** by `receipt_id` / dedupe key.
3. **Cart integrator consumer:** Consumes **`billing.receipt.minted`**, updates order/checkout state, publishes **`commerce.entitlement.requested`** (or invokes host)—**idempotent** on `receipt_id`.
4. **MerchantHost / product consumer:** Grants **entitlement**, provisions resources; may emit **`commerce.entitlement.granted`** / **`product.service.provisioned`** for further pipelines.

**Subscription renewals** use **only Plane 2**: no browser; **Processor** webhook → **Payux** → **`billing.receipt.minted`** → same downstream consumers.

### Idempotency and OpenErgo replay

- Align with Payux **webhook dedupe** and **ledger** semantics ([DESIGN.md](DESIGN.md)): every bus consumer that grants **entitlements** or side-effects must be **replay-safe** (e.g. unique constraint on `receipt_id`, or idempotent upsert).
- OpenErgo’s **ack only on full success** model matches **at-least-once** delivery: design handlers to tolerate duplicate **`billing.receipt.minted`** messages.

### Summary

| Plane | Primary mechanism | Bus role | Receipt available? |
|-------|-------------------|----------|---------------------|
| **1** Interactive | HTTP + server **CartService→Payux** session create; browser follows handoff | Optional: intent/audit events; **not** a substitute for handoff response | **No** |
| **2** Settlement saga | **Processor→Payux** webhook → **Payux** publishes receipt event | **Primary:** orchestrate cart + entitlement + provisioning | **Yes** |

This is the **closest analog** to “HTTP ingress → cart → Payux → receipt → entitlement → provision” on a message queue: **Plane 2** matches that story end-to-end; **Plane 1** still needs a **synchronous** path for **session + handoff** to the **Payer client**.

---

## Workflow A — One-time purchase (OTP), success path

1. **Payer** completes intent to purchase (e.g. clicks pay or checkout CTA on **MerchantHost** UI; cart lines were assembled earlier per **CartService** / SKU rules—out of scope here).
2. **CartService** computes **amount_due** and currency from its domain rules and builds an **invoice** (stable `invoice_id`, payer reference, success/cancel URLs, opaque **metadata**). Payux does not re-total from metadata.
3. `CartService → Payux`: **Start payment** for that invoice (server-side, authenticated integrator call).
4. `Payux → Processor`: Create **payment session** for exactly **invoice.amount_due** (adapter blackbox).
5. `Payux → Payer` (browser): **Checkout handoff**—redirect or token to Payux/checkout shell, then into **Processor** hosted UI as configured.
6. **Payer** completes payment with **Processor** (card data stays in Processor scope; Payux orchestrates handoff only—see FUNCSPEC).
7. `Processor → Payux`: Asynchronous **settlement signal** (e.g. verified webhook).
8. **Payux** records settlement, **dedupes** by processor event, appends **ledger** mint, issues **receipt**.
9. `Payux → CartService` (pull or push per integration): **Receipt** (or stable receipt id + correlation) available to integrator.
10. `CartService → MerchantHost`: Hand off proof of payment so **MerchantHost** can **grant or extend entitlements** (each entitlement backreferences receipt/billing txn per `empirestate/BILLING.md`).
11. **Payer** sees confirmation (Payux shell and/or **MerchantHost** success URL with stable correlation parameters).

---

## Workflow B — Subscription: initial charge, success path

1. **Payer** chooses a subscription offer on **MerchantHost** (plan semantics and pricing rules live outside Payux).
2. **CartService** builds the **first** payable **invoice** (setup/first period/proration as **its** policy), optional `billing_mode=subscription_initial`, and includes an **opaque subscription handle** in **metadata** for later correlation.
3. `CartService → Payux`: **Start payment** (same integrator contract as OTP; Payux may create a session that establishes recurring mandate—**inside** Processor adapter).
4. `Payux → Processor`: Session that captures payment method and establishes **recurring** per adapter (blackbox).
5. `Payux → Payer`: Checkout handoff; **Payer** completes **Processor** UI for initial collection.
6. `Processor → Payux`: Verified **initial settlement** event.
7. **Payux** mints **receipt** for the **initial** charge (same proof semantics as OTP).
8. `Payux → CartService`: Initial **receipt** delivered; **MerchantHost** grants entitlements for the period using that receipt as provenance.

---

## Workflow C — Subscription: renewal charge, success path

*No new invoice POST from **CartService** in the typical case; Processor schedules the charge.*

1. **Processor** runs scheduled **renewal** charge using stored mandate (blackbox).
2. `Processor → Payux`: Verified **renewal settlement** webhook (success).
3. **Payux** **dedupes** event; mints **new receipt** for this settlement (new receipt id); links logically via stored **metadata** / subscription correlation from initial invoice snapshot.
4. `Payux → CartService` (webhook callback, polling, or message bus—integration contract): **Renewal receipt** or event for finance and provisioning.
5. **MerchantHost** extends **entitlement** coverage (or records renewal) using **this** receipt as provenance—access grace/downgrade policy is **host** decision, not Payux.

---

## Workflow D — Cancel / hard failure (OTP or initial subscription session)

1. **Payer** abandons **Processor** UI or payment **declines** / **fails** before settlement.
2. **Processor** signals non-success to **Payux** where applicable; otherwise Payux treats session as **no receipt**.
3. **Payux** does **not** mint receipt for that attempt.
4. `Payux → Payer`: Branded **failure/cancel** surface or redirect to **invoice.cancel_url**.
5. **MerchantHost** / **CartService** resumes UX (cart still valid, retry, support)—no entitlement grant from Payux for that attempt.

---

*For SKU/cart-side steps before the invoice exists, see `../SKU-and-Cart/SHOPPING-CART.md` and related docs.*
