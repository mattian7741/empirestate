# Design — Detailed implementation handoffs

Each document in this folder is **narrowly scoped** to one component or bounded context. It is a **blueprint** so another agent or engineer can implement without re-deriving architecture from the full corpus.

**Product truth (what/why):** `empirestate/BILLING.md`, `empirestate/app-documentation/SKU-and-Cart.md`, `empirestate/app-documentation/Payux.md`.

**Agent handoff:** Pair any design doc with `GUIDE.md` for EmpireStack-wide constraints. Obey `empirestate/TENETS.md` — **1** VCS always; **2–3** scaffolding then lean; **4–5** deploy and iterate live; **7–14** when writing or changing docs.

---

## Commerce and billing stack

Three design documents cover the **white-labeled** path from catalog sellables through checkout to payment proof. The **host application** owns products, services, and entitlements (see product spec); it is not a fourth design file here—only its **integration contracts** appear in the docs below.

```text
Host application (products, services, entitlements — authoritative domain)
        │
        │  catalog sync or API (implementation-specific)
        ▼
SKU-MANAGEMENT.md — purchasable offers → SKU + price
        │
        │  price resolution at line-add / checkout refresh
        ▼
SHOPPING-CART.md — basket, balance, invoice → Payux
        │
        │  invoice (amount_due authoritative)
        ▼
PAYUX.md — payment → receipt (ledger)
        │
        ▼
SHOPPING-CART.md (continued) — receipt to host → grant entitlements + provenance
```

**Suggested reading order for a full checkout implementer:** `SKU-MANAGEMENT.md` → `SHOPPING-CART.md` → `PAYUX.md` (Payux can be read first if billing-only).

---

## Relationship to other docs

| Layer | Location | Role |
|-------|----------|------|
| Product spec | `empirestate/` | What and why; ecosystem boundaries. |
| Engineering spec | `core-platform-spec/` | Contracts, policies, formal sections. |
| **Detailed design** | `design/` | Scoped blueprint per component; implementation-ready. |

---

## Documents

| File | Component | Summary |
|------|-----------|---------|
| **PAYUX.md** | Billing (Payux) | **Invoice → payment → receipt**; centralized ledger; opaque metadata; no commerce semantics. M1 priority. |
| **SKU-MANAGEMENT.md** | SKU management | Maps **products / services / purchasable offers** (from host) to **SKUs** and **prices**; price resolution; no payment rails. |
| **SHOPPING-CART.md** | Shopping cart | **Cart** lines, **balance**, **invoice** assembly; calls Payux; handles **receipt** return and **entitlement-grant handoff** to host (with receipt/gift provenance). |

---

## Cross-cutting rules (all three)

- **`amount_due` on the invoice is authoritative** for money movement (Payux). Cart + SKU layer must keep **line totals consistent** with that figure when building the invoice.
- **Payux does not interpret** SKU, product, or entitlement meaning—it stores **metadata** for audit and downstream use only.
- **Entitlements** live in the **host application**; each grant from purchase **backreferences** the billing **receipt / transaction** (gifts: grant transaction). See `empirestate/BILLING.md`.
