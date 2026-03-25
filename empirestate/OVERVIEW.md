# EmpireStack — Overview

EmpireStack unifies a universal strategy spanning every aspect of software and application domains—implemented and envisioned—into a single comprehensive architecture. The strategy is established upfront as scaffolding: each identified aspect has a designated place in the stack, so components slot in correctly as they are built.

## Scope

The project synthesizes approaches that have appeared across many prior efforts, each touching a different subset of the strategy. EmpireStack brings them together into one coherent whole. The concrete solution and its specifics will be elaborated in later documentation. See [[APPLICATIONS]] for the host of applications that build on this stack.

## Aspects (Scaffolding)

The following concerns are the high-level dimensions of the architecture. Each will be explored in depth separately.

| Aspect | Description |
|--------|-------------|
| **Event-driven nano services** | Services built on OpenErgo—the backend substrate for communications, events, pipelines, HTTP, and inter-app integration. See [[OPENERGO]] |
| **Distributed datastore with shared ledger** | Append-only ledger of claims; truth derived by projection. See [[LEDGER]] |
| **Universal identity engine** | Participation-first, roundtrip auth, commuting identity. See [[IDENTITY]] |
| **Universal billing gateway** | Invoice → payment → receipt; app owns products/services and entitlements (backref purchase/gift); SKU manager + cart map to SKUs and checkout. See [[BILLING]] |
| **Target-agnostic deployment** | Infrastructure disposable; declarative, containerized, cloud-independent. See [[DEPLOYMENT]] |
