# Hot Potato Formal Specification — Tenets, Principles, and Philosophy

This addendum distils the **invariant core** of Hot Potato from ~15 years of implementations (TheSystem 2012, HotPotato, HotPistachio, and related variants). It complements the **Hot Potato iteration plan (HP-1 through HP-9)** in REQUIREMENTS and the **Convergence Outcome** by making explicit the tenets, principles, and philosophy that any Hot Potato–compliant product must preserve.

**Reference implementations analysed:** `~/project_base/ref_impl/HPX` (thesystem, HotPotato_02, hotpotato, hotpotato_03, hotpotato_04, hotpistachio, more_thesystem, more_hotpotato, git_hp).

---

## 1. Philosophy (Why Hot Potato Exists)

- **Event-driven teams.** Work is coordinated via **tasks**; events (UI actions, messages, ingestion, delegation) drive updates. Teams can be human-only or human-plus-AI. The system is “event driven” in the sense that state changes flow from events, not from ad-hoc direct writes.
- **Task as the unit of work.** The primary abstraction is the **task**: a first-class entity with identity, name/label, hierarchy (parent/children), ordering (priority/sequence), status (e.g. active/complete), and optional assignment or delegation. Everything that “happens” in the product is anchored to tasks.
- **Focus and flow.** The product supports “one thing at a time”: a **next-best** focus, keyboard-driven navigation, and minimal friction to add, complete, reorder, or delegate. The tactical view and prioritization (HP-6) are the natural evolution of this—deterministic “next best task” and delegation affecting priority.
- **Delegation and assignment.** Tasks can be **assigned** (owner/primary) or **delegated** (history, created_by, assignee). When AI agents are present, they delegate to humans for physical actions; humans delegate to each other or to agents via a structured protocol. Email-driven delegation (REQUIREMENTS) extends this into a viral, consumer-facing engine.
- **Structured communication when agents are involved.** If the system includes AI or multi-actor agents, communication follows an explicit **protocol** (e.g. from/to/message JSON). Redundant acknowledgements and cyclical conversation are minimized; silence can mean “no further response needed.”
- **Active user by default; low barrier to entry.** Hot Potato treats any living, breathing person as an **active user** in the system. It biases for usability **without** requiring “sign up” or “create a profile.” Barrier to entry is reduced by allowing engagement through a **temporarily authenticated session** from a click-through in email. This is a critical element of the viral campaign: recipients can act on delegated work without creating an account first.
- **One thing at a time; always an action to remove the task.** Users are more likely to make progress when they have clarity on priorities and are faced with **one task at a time**. Hot Potato does the work to determine priority and present a **sequential stream** of tasks—one by one. The user, faced with a single task, must take an action that **removes that task from view**. Hot Potato ensures there is **always** such an action. The five actions (the **five D’s**) are: **Do** (complete the task), **Delegate** (to someone else), **Divide** (identify subtasks—if the task can’t be completed due to size or complexity, the next task may be the first subtask), **Defer** (identify a blocker or reason to defer, e.g. “I’m not at my desk,” “this isn’t a priority”), **Delete** (abandon—the task is no longer valid).
- **No estimates; tree-based work units.** Hot Potato does **not** collect or track “estimates” like other task-management systems. Forcing users to add estimates (or due dates only) invites noisy data—e.g. engineers filling fields to satisfy forms. Those fields exist so systems can be sold to executives who like reports; sales teams put nice estimates in for the demo. Estimates are not sustainable. Instead, Hot Potato uses the **complexity of the task tree** to infer work: if a task has **no child tasks**, it is **one unit of work**; if it isn’t one unit, the user divides until children (or children’s children) are one unit. **Counting the nodes under a task** gives total work units; everything rolls up. Early “estimates” are inaccurate but **converge quickly**.
- **Projected delivery date, not due dates.** Rather than artificial due dates, Hot Potato **projects a delivery date** given the task-tree complexity under the node and the **competition among concurrent work streams**. Everything is dynamic: as concurrency increases, projections extend; pulling any string affects the whole system, which forces business leaders to make tough decisions with **real-time impact**. An **artificial due date** may be entered to **lock** a dimension of the fabric (forcing other variables to move)—following the classic time vs. cost vs. quality trade-off; locking one unlocks another. This is **overt** so people can make informed decisions.
- **Parent–child: 1-to-many today, many-to-many ideal.** Current implementations model parent–child as **1-to-many**. The ideal implementation models parent–child as **many-to-many**, with **guardrails** to preserve consistency and usability.
- **Events on the Ergo backbone.** Hot Potato events **should flow on the Ergo backbone** (message-driven, substrate-agnostic execution and routing).
- **Data architecture evolution.** The Hot Potato MVP runs with a **central DB** or **microservice DB pattern** with distributed databases. The **ultimate goal** is **decentralized datastores** and **payload-based data** that synchronizes with multiple clients (analogous to a shared ledger). Use of a **blockchain ledger** for the **transactional exchange of tasks** is in scope for innovation.

These points appear across the lineage: TheSystem (task hierarchy, keyboard nav, API), HotPotato simple (queue-batched updates, task envelope), HotPotato_02 (React task tree, Connexion API, GPT agents with protocol), hotpotato_04 (Epic/Story/Task/Resource, ingestion), and REQUIREMENTS (email-driven delegation, event-driven orchestration, next best task).

---

## 2. Core Tenets (Invariants)

The following are **non-negotiable** for any implementation that claims to be Hot Potato–compliant.

### 2.1 Task as First-Class Entity

- Every unit of work is a **task** with a stable, documented shape.
- At minimum, a task has:
  - **Identity** (id)
  - **Label** (name, subject, or description)
  - **Hierarchy** (parent, children or equivalent)
  - **Ordering** (priority or sequence)
  - **Status** (e.g. active vs complete, or status_id)
- Optional but recurring: **assignment** (owner, primary_id, assignee), **delegation** (created_by, task_history), **dates** (due, created, completed), **type** (type_id, issue type).

### 2.2 Hierarchy and Ordering

- Tasks form a **tree**: each task has a parent and an ordered list of children (or siblings ordered by priority/sequence). Current implementations are **1-to-many** (one parent per task). The **ideal** implementation models parent–child as **many-to-many**, with **guardrails** to preserve consistency and usability.
- The API or persistence layer supports:
  - **Relationship operations:** relate(task, parent), unrelate(task, parent) (or equivalent).
  - **Ordering operations:** set priority/sequence, or reorder relative to a target (e.g. source/target).
- Children may be **lazy-loaded** (e.g. loaded on expand) to keep the initial payload small and support large trees.

### 2.3 Event-Driven Updates

- Changes flow as **events**: user actions (create, complete, reorder, delete, assign, delegate) are translated into API or message calls that mutate backend state.
- Persistence can be path-based (e.g. put/get by path like `database/table/id/column`) or CRUD + relationship endpoints (create_task, update_task, relate, unrelate, priority). Response shape is a **contract** (e.g. Status, Tasks or data envelope).
- **Batching** is acceptable and often desirable: e.g. enqueue UI changes and flush as a single batch (queue → PUT) to reduce round-trips and preserve consistency.

### 2.4 Focus and Next-Best

- The UI supports **focus**: one task (or one view) is “current.” Navigation is **depth-first** (next/prev in tree order) and **keyboard-first** where applicable (Enter = new sibling, Backspace = delete when empty, Shift+arrows = reorder/indent/outdent).
- **Next best task** is a derived concept: a deterministic prioritization (HP-6) or AI-assisted suggestion (HP-8) that answers “what should I do next?” Delegation status can affect that priority (e.g. delegated tasks surface for the delegatee).
- The user is presented with **one task at a time**. There is **always an action** that removes the current task from view: the **five D’s**—**Do** (complete), **Delegate**, **Divide** (add subtasks; next task may be first subtask), **Defer** (blocker or reason to defer), **Delete** (abandon). Hot Potato does not leave the user stuck with no valid action.

### 2.5 Active User by Default; Low Barrier to Entry

- Any living, breathing person is treated as an **active user**. The product biases for **usability without sign-up or profile creation**. Engagement is enabled through **temporarily authenticated sessions** (e.g. email click-through). This is critical for the viral campaign: recipients can act on delegated work without creating an account.

### 2.6 No Estimates; Tree-Based Work Units

- Hot Potato **does not collect or track user-entered estimates**. Forced estimates and due-date fields invite noisy data and exist in other systems for exec reporting and sales demos; they are not sustainable. Instead, **work is inferred from the task tree**: a task with **no children** is **one unit of work**; if it is not one unit, the user divides until leaf tasks are one unit. **Node count under a task** = total work units; everything rolls up. Early inference is inaccurate but **converges quickly**.

### 2.7 Projected Delivery Date (Not Due Dates)

- Rather than arbitrary due dates, Hot Potato **projects a delivery date** from task-tree complexity under the node and **competition among concurrent work streams**. The model is **dynamic**: more concurrency extends projections; any change propagates through the system so business leaders see **real-time impact** and can make informed trade-offs. An **artificial due date** may be used to **lock** one dimension (time/cost/quality), forcing other variables to move—all **overt** for informed decisions.

### 2.8 Events on the Ergo Backbone

- Hot Potato events **flow on the Ergo backbone**: message-driven, substrate-agnostic execution and routing as defined in the Ergo formal spec (AA_ADDENDUM_02). Writes and state changes are carried by Ergo; the app does not bypass the backbone for event flow.

### 2.9 Data Architecture Evolution

- **MVP:** Hot Potato runs with a **central DB** or **microservice DB pattern** with distributed databases.
- **Ultimate goal:** **Decentralized datastores** and **payload-based data** that synchronizes across multiple clients (analogous to a shared ledger). Use of a **blockchain ledger** for the **transactional exchange of tasks** is in scope for innovation.

### 2.10 API and Envelope Contract

- The product exposes a **stable API** for task CRUD and relationship/ordering operations. Optionally, batch or bulk endpoints are provided.
- Responses follow an **envelope** convention (e.g. Response.Status, Response.Tasks, or platform envelope with data/error). The contract is versioned and aligned with the platform (BFF, Ergo) where Hot Potato integrates.

### 2.11 Optional Team/Agent Layer

- When the product includes **agents** (AI or human roles): communication is **protocol-based** (e.g. from, to, message). Agents have identity, conversation history, and clear delegation rules (e.g. AI cannot perform physical actions; physical actions are delegated to humans).
- **Minimal redundant communication:** no unnecessary acks, no gratitude except to humans where appropriate; ending cyclical conversations by not responding is acceptable and encouraged.

---

## 3. Principles (Design Levers)

These principles appear across the lineage and should guide design of the flagship Hot Potato app and any variants.

- **Lazy-load children:** Load subtasks on expand (or on demand) rather than loading the full tree up front.
- **Batch updates where appropriate:** Queue local changes and flush in one request to reduce round-trips and preserve consistency (e.g. simple queue.js pattern).
- **Keyboard-first interaction:** Power users can add, complete, delete, reorder, and navigate without leaving the keyboard (depth-first next/prev, shortcuts).
- **Sortable, reorderable list:** The task list is reorderable (e.g. drag-and-drop or Shift+arrows); order is persisted via priority/sequence and relationship ops.
- **Collapse/expand:** Hierarchy can be collapsed or expanded; state can be persisted or derived from focus.
- **Viral/consumer product direction:** Email-driven delegation, invite flows, **minimal or no sign-up** (temporary auth from email click-through), growth instrumentation (HP-9). The product is a **delegation engine** and **event-driven task orchestration system**, not just a personal todo list.
- **Five D’s as the only exit from focus:** Every task in focus can be removed from view by one of: Do, Delegate, Divide, Defer, Delete. The UI and backend support all five; the user is never left without a valid action.
- **Work units from tree, not from estimates:** No estimate or due-date fields required; work units = node count under a task; delivery is **projected** from tree complexity and concurrency.
- **Ergo backbone for events:** All Hot Potato event flow uses the Ergo backbone; no bypass for writes or orchestration.
- **Real-time and eventual consistency:** Hot Potato aligns with real-time updates where possible and eventual consistency as a first-class assumption (REQUIREMENTS); projection convergence and delta sync (HP-7) support this.
- **Platform-native flagship:** Hot Potato does not bypass BFF, gateway, identity, or entitlements; it consumes platform capabilities and conforms to structural discipline (REQUIREMENTS).

---

## 4. Contract Invariants (Summary)

For alignment with the platform and consistency across implementations:

| Invariant | Description |
|-----------|-------------|
| **Task is the unit of work** | All work is represented as tasks with id, label, hierarchy, ordering, status; optional assignment/delegation and dates. |
| **Hierarchy and ordering are explicit** | Tree structure (1-to-many current; many-to-many ideal with guardrails) with relate/unrelate and priority/sequence or reorder operations. |
| **Updates are event-driven** | Changes flow via API or messages; batching is allowed; envelope contract is stable. **Events flow on the Ergo backbone.** |
| **Focus and next-best; five D’s** | One task at a time; there is always an action to remove the task from view (Do, Delegate, Divide, Defer, Delete). |
| **Active user; low barrier** | Any person is an active user; no sign-up required; temporary auth from email click-through for viral engagement. |
| **No estimates; tree-based work** | Work units inferred from task tree (node count); delivery **projected** from tree complexity and concurrency; no mandatory estimate/due-date fields. |
| **API contract is stable** | CRUD + relationship/ordering ops; envelope (Status, Tasks or platform envelope). |
| **Delegation and assignment are first-class when present** | Owner, delegatee, task history; when agents exist, protocol-based messaging and delegation to humans for physical actions. |
| **Platform discipline** | No direct DB access; no bypass of BFF or gateway for writes; no circumvention of identity or entitlements. Events on Ergo. |
| **Data evolution** | MVP: central or microservice DB; ultimate: decentralized, payload-based sync, optional blockchain ledger for task exchange. |

---

## 5. Relationship to Platform Spec and REQUIREMENTS

- **REQUIREMENTS** defines the Hot Potato iteration plan (HP-1–HP-9), structural discipline (no direct DB, no bypass), and convergence outcome (viral product, email-driven delegation, event-driven orchestration, real-time + eventual consistency).
- **Sections 5–7** (subsystems, data, BFF) define how Hot Potato consumes the platform (projections, Ergo, envelope, identity).

This addendum makes explicit the **tenets and principles** that the Hot Potato lineage has converged on, so that the flagship app and any variants remain aligned with the philosophy while evolving features (e.g. AI prioritization, viral loop, real-time sync).

---

*Document derived from analysis of ref_impl/HPX (TheSystem, HotPotato, HotPistachio, and related projects).*
