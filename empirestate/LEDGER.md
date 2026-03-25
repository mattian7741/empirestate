# Ledger — Distributed Datastore

Ledger mechanics are a **core mechanic**—the ledger pattern must be implemented and applied across domains. The pattern is reused for:

| Application | Currency | Example |
|-------------|----------|---------|
| **Financial** | Tickets, monetary value | Billing: mint, redeem, refund, transfer. See [[BILLING]]. |
| **Claims** | Who claimed what, when | Factual, consensus, history. This doc focuses here. |
| **Social** | Commentary, opinions | Reputation, endorsements, reviews. |
| **Record-based** | Matches, scores, events | Subsets of OpenErgo events; application-specific ledger domains. |

All depend on the consuming application; the **ledger machinery shares common DNA**—append-only, immutable, projection-derived truth, idempotent.

**Datastore topology** is application-dependent: distributed (this document) or centralized (e.g. Kafka+Postgres). Both are valid.

---

This document describes the **distributed claims ledger**—a multi-writer, append-only ledger where entries are claims, not truth.

**Core axiom**: The system stores *who claimed what, when, and under what context*—not "what is true." Truth is derived through interpretation, not enforced at write time.

## Properties

| Property | Meaning |
|----------|---------|
| **Fully distributed** | Every client can write and hold a replica |
| **No central authority** | No single node required for ingest |
| **Cloud-optional peers** | Cloud nodes may act as durable, always-on replicas |
| **Append-only** | No edits; corrections are new events |

### Adjacent Datastores (Not Central)

Central or cloud-based datastores may be used, but they are **adjacent**, not central. They provide support when:
- Clients cannot combine to complete the entire picture
- Churn reduces participation

When the network is large enough, these supporting datastores may not be necessary. They are a fallback, not the foundation.

## Data Model

Each entry is an **immutable, signed event** representing a speech act. Types: claim, observation, challenge, evidence, resolution, retraction.

**Event structure**:
- unique id
- author identity (public key)
- timestamps (observed + recorded)
- subject (entity)
- predicate/value (what is claimed)
- references to prior events
- payload (details)
- signature

**Critical distinction**: Event validity ≠ claim correctness. A valid event means "this was said," not "this is true."

## Contradiction Model

Conflicts are **expected and preserved**. The ledger accepts all valid events; contradiction detection happens during interpretation, not at write time.

**Conflict types**:
- **Concurrent**: independent conflicting claims
- **Causal**: direct disagreement or reversal
- **Policy**: cannot both be operative under rules

**Resolution** (via new events, never mutation):
- arbitration
- supersession
- evidence weighting
- role-based authority

Losing claims remain as history. Nothing is deleted.

## Architecture Layers

### 1. Ledger (source of record)
- Append-only event log
- Replicated across peers
- Cryptographically verifiable

### 2. Projection (derived state)
Interprets the ledger using rules. Produces views such as:
- current operative state
- unresolved conflicts
- highest-authority claim
- most recent claim
- adjudicated outcomes

Different consumers may use different projections over the same ledger.

## Client Architecture: Local+Sync First

The data model and store design use a **local+sync first** approach:

- The client is effectively a **microservice with its own datastore**
- Datastore synchronization runs as a **separate, isolated process** with the network
- The **local client-side datastore is the abstraction layer** between the client and the network

The client speaks to its local store; the sync process handles network exchange independently.

## Sync Model

Sync is **application-specific**. Valid patterns:

| Pattern | Topology | Description |
|---------|----------|-------------|
| **Real-time** | Central data domain + message bus | Clients sync via central store and bus. |
| **Real-time** | Network of distributed | Clients sync to distributed peers. |
| **Disconnected/isolated** | Local → central | Write to local data domain; sync data domain to central. |
| **Disconnected/isolated** | Local → distributed | Write to local data domain; sync to network of distributed. |

Patterns are reusable—applications that use the same pattern share implementation. Common infra where patterns align; implementation diverges only where purpose diverges.

This document's default (distributed claims ledger) uses: peers exchange unseen events (gossip/anti-entropy), idempotent replay, no global total order, causal links via event references. For the layered model (real-time, delta, reconciliation), see core-platform-spec §9.

## Design Principles

- Persistence is not endorsement
- All writes are claims, not facts
- History is monotonic; corrections are new events
- Truth is a projection, not a stored value
- Arbitration is explicit and recorded

## Implications

- Offline-first; supports client churn
- Resilient to malicious or incorrect writers
- Full auditability and traceability
- Flexible policy evolution over time
- Avoids premature consensus

## Mental Model

A **distributed system of attestations**, not a shared-state database. Reality is computed, not stored.
