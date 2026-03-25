# Ergo Additional Lineage — Begroupd and Bento (Reference Only)

This addendum records the analysis of **begroupd** and **bento** in `~/project_base/ref_impl/additional_ergo`. It is for **reference and lineage continuity only**.

**Authoritative Ergo spec:** **AA_ADDENDUM_02 (Ergo Formal Spec)** is the source of truth. This document **does not amend** AA_ADDENDUM_02 or the platform strategy. If anything here conflicts with the formal spec or represents a less mature pattern, the formal spec **wins**. Do not elevate obsolete or immature implementations over recent improvements.

---

## 1. Source Material

- **begroupd@dropbox** — Node.js stack (2015–2016): service-sdk (Service, MQService, MQConnector), node.gateway (Express + Socket.IO + Rabbit.js), begroupd-server with services as submodules (service-task, service-gateway, service-db, etc.).
- **reference/reference07, reference07a** — bento-dev, bento-sdk, bento-sdk-python3, bento-lambdas (many files encrypted .bc; limited readable content).

---

## 2. What Aligns with the Current Ergo Spec

The following are **consistent** with AA_ADDENDUM_02 and reinforce the lineage; they are not new mandates.

- **Runtime owns transport and lifecycle.** Begroupd’s `MQService` owns the MQ connector, `start()`, `publish()`, and message dispatch. Handlers override `onMessage(message)` and do not open connections or ack directly. Aligns with “runtime wraps behavior” and “the runtime invokes the worker function; the worker function does not invoke the runtime.”
- **Message as first-class.** Messages have `routingKey`, `source`, `data`; handlers receive a message object and publish an output message object. Aligns with “message as first-class contract” (payload, routing, scope).
- **Routing drives dispatch.** `preMessage` routes by `message.routingKey` to `onMessage` or `onService`; handlers set `routingKey` on outbound messages. Aligns with “routing semantics as data” and “subscribe/publish expressed as data.”
- **Substrate behind an abstraction.** Begroupd uses RabbitMQ via an MQ connector; the service base class hides connection and channel details. Aligns with “substrate abstraction” (concrete transport behind a stable interface).

---

## 3. What Is Not Elevated

To avoid overwriting the current strategy with less mature patterns:

- **Class-based extension instead of function injection.** Begroupd services extend `MQService` and override `onMessage`; they do not use “load by reference” (e.g. `module:function`) or config-driven binding. The **current** spec favors injected pure functions and config-driven payload→args mapping; that remains the norm. Begroupd is an earlier, valid instantiation of “runtime wraps behavior” but does not define the preferred shape.
- **No explicit config-driven binding.** Begroupd does not map message fields to handler parameters via config (e.g. `args` or `input_bindings`). That remains a tenet of the formal spec; begroupd is not used to weaken it.
- **Bento.** Readable Bento material in ref_impl is limited (encrypted or missing). Bento is noted as part of the lineage (Python/RabbitMQ/stdio era); no Bento-specific behavior is elevated above the formal spec.
- **Gateway and transport details.** node.gateway (Express, Socket.IO, Rabbit.js) is one possible gateway pattern; it does not replace or constrain the Ergo HTTP gateway or substrate variants in the spec.

---

## 4. Use of This Addendum

- **Lineage:** Begroupd and Bento are part of the Ergo lineage (2015–2018) and confirm that “runtime owns transport,” “message as contract,” and “routing as data” have long been present.
- **Do not:** Amend AA_ADDENDUM_02, relax tenets (e.g. config-driven binding, function injection), or treat begroupd/bento patterns as canonical over the formal spec.
- **Convergence:** New work (e.g. core-ergo) follows AA_ADDENDUM_02; earlier implementations inform the narrative but do not override it.

---

*Document derived from ~/project_base/ref_impl/additional_ergo. Authoritative Ergo source: AA_ADDENDUM_02.*
