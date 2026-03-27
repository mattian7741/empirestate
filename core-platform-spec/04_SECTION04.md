# 4. Product Layer

This section defines products that are built on top of the platform foundation.  
Products are not aspects. They are deployable capabilities that use platform aspects and subsystems but do not redefine them.

---

# 4.1 Billing Product

The Billing Product is a first-class platform product. It is not an aspect and it is not an infrastructure concern. It is a domain system built on top of the platform.

Its purpose is to act as a **transaction broker and ledger system** that converts monetary value into internal value units ("tickets") and binds those units to entitlements across applications and domains.

**EmpireStack Payux boundary:** In the reference billing implementation (**Payux**), catalog, pricing, and cart are **separate** applications. Payux accepts an **invoice** (authoritative amount due) and returns a **receipt** backed by ledger ticket primitives; **product code** and **redemption** semantics for the host application are orchestrated **outside** Payux (cart + app), which may supply opaque metadata Payux stores but does not interpret. The host application models **products and services** and **entitlements** (not SKUs); **entitlements granted from purchase are expected to backreference** the billing receipt or transaction id (gifts: backreference the grant transaction). See `empirestate/BILLING.md` and `applications/Payux/DESIGN.md`.

The billing product must operate independently of:
- Specific deployment topology
- Specific payment provider
- Specific application domain

It must function in:
- Single-tenant deployments
- Multi-tenant SaaS
- Enterprise-isolated environments
- Hybrid environments

---

## 4.1.1 Conceptual Model

The Billing Product revolves around five primary primitives:

1. Ledger  
2. Ticket  
3. Product Code  
4. Redemption  
5. Entitlement Binding  

These primitives are stable and versioned.

---

## 4.1.2 Ledger

The Billing Ledger is the system of record for financial and value transactions. It implements the **ledger pattern**—common DNA shared with claims, social, and record-based ledgers across the platform (see empirestate/LEDGER). Currency here is financial; the machinery (append-only, immutable, idempotent) is shared.

### Properties

- Append-only.
- Immutable transaction entries.
- Strong auditability.
- Deterministic reconstruction of balance.
- Idempotent transaction handling.

### Ledger Entries Must Include

- Transaction ID (stable, unique)
- Correlation ID
- Tenant/App scope (if applicable)
- Principal (actor)
- Timestamp
- Type (mint, redeem, refund, transfer, adjustment)
- Amount (ticket quantity or monetary value)
- Source reference (payment processor, internal system, manual adjustment)
- Metadata (structured, versioned)

### Rules

- Ledger entries are never mutated.
- Corrections occur via compensating transactions.
- Ledger must be reconstructible from events.

---

## 4.1.3 Ticket Model

Tickets are internal value units minted by the billing system.

Tickets are not money. They are ledger-backed internal assets.

### Ticket Properties

- Unique ticket identifier.
- Quantity (may be fractional if allowed by policy).
- Owner (tenant-scoped identity or wallet abstraction).
- Status (available, locked, redeemed, expired, refunded).
- Expiration policy (optional).
- Product binding (if pre-bound).

Tickets may represent:
- Subscription value
- One-time purchase value
- Usage credits
- Physical goods
- Digital goods
- Cross-application value

Tickets are portable within the defined ledger universe.

---

## 4.1.4 Product Code Model

Product Codes represent redeemable targets.

A Product Code:

- Is canonical and versioned.
- Resolves to a specific entitlement schema.
- Belongs to a specific app domain.
- May define:
  - Duration
  - Feature flags
  - Quotas
  - Role upgrades
  - Resource grants

Product Codes do not implement entitlements directly.  
They map to entitlement specifications governed by the Entitlements Aspect.

---

## 4.1.5 Redemption Flow

Redemption binds value (tickets) to entitlements.

### Redemption Steps

1. Ticket validation (ownership, status, sufficiency).
2. Product code validation.
3. Ticket consumption (lock, partial spend, or full consume).
4. Entitlement issuance.
5. Ledger update.
6. Audit event emission.

Redemption must be:

- Idempotent.
- Safe under retries.
- Safe under duplication.
- Atomic at the ledger level.

### Spend Semantics

The billing product must support:

- Full consumption
- Partial consumption
- Locking pending confirmation
- Compensating reversal

Exact semantics may vary by deployment profile but must conform to ledger invariants.

---

## 4.1.6 Entitlement Binding

Redemption does not grant raw permissions.  
It grants entitlements bound to:

- App domain
- Tenant scope
- Principal

Entitlements:

- Are evaluated by the Entitlements Aspect.
- Must be versioned.
- Must be revocable.
- Must be auditable.

The Billing Product does not interpret entitlements.  
It binds value to entitlement specifications defined elsewhere.

---

## 4.1.7 Payment Processor Adapters

The Billing Product integrates with external payment rails via adapters.

Adapters must:

- Convert external payment confirmation into ledger mint transactions.
- Handle webhooks idempotently.
- Support refund flows.
- Support dispute/chargeback handling.
- Maintain reconciliation logs.

Adapters are:

- Pluggable.
- Replaceable.
- Deployment-profile dependent.

Examples:
- Stripe
- Alternative regional providers
- Crypto processors (if applicable)

Payment processors must never write directly to entitlements.  
They only mint value into the ledger.

---

## 4.1.8 Multi-Tenant Behavior

The ledger may be:

- Shared across tenants within a deployment.
- Isolated per tenant in enterprise deployments.

In all cases:

- Tenant boundaries must be enforced.
- Cross-tenant leakage is prohibited.
- Ticket ownership is explicit and auditable.

---

## 4.1.9 Isolation from Platform Internals

The Billing Product:

- Must not depend on specific frontend architecture.
- Must not depend on specific transport protocols.
- Must not depend on Ergo internals.
- Must operate via stable contracts.

It may use:

- BFF endpoints
- Ergo command workflows
- Projection models
- Storage adapters

But its domain invariants must remain transport-agnostic.

---

## 4.1.10 Observability and Audit

Billing operations must be fully observable.

Mandatory:

- Correlation IDs for every transaction.
- Structured audit logs for mint, redeem, refund.
- Explicit error codes for payment and redemption failures.
- Reconciliation reporting capability.

Financial integrity supersedes performance optimization.

---

## 4.1.11 Consistency Guarantees

The Billing Product operates under eventual consistency.

However:

- Ledger entries must be strongly durable.
- Ticket balance reconstruction must be deterministic.
- Redemption must be safe under replay.
- Read models may lag, but ledger truth must not.

---

## 4.1.12 Deployment Considerations

Billing must operate in:

- Single-node POC environments.
- Full distributed SaaS environments.
- Enterprise-isolated deployments.

The product must not assume:

- Always-available external payment providers.
- Continuous internet connectivity.
- Centralized infrastructure.

Offline reconciliation must be possible.

---

## 4.1.13 Versioning and Evolution

The Billing Product must support:

- Schema versioning of ledger entries.
- Versioned product codes.
- Backward compatibility in redemption logic.
- Controlled deprecation of legacy value models.

No breaking changes to ledger interpretation are permitted without migration strategy.

---

The Billing Product is a durable value broker.  
It must be conservative, deterministic, auditable, and portable across all deployment modes.