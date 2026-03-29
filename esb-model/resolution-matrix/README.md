# ESB resolution matrix — `user-data-store`

**Illustrative.** One **logical artifact** traced **top → bottom** through **ESB** (full stack: lay + interim + concrete). Field names are **examples**, not a frozen schema.

**Lay entry:** [`../ideal-system.esb.yaml`](../ideal-system.esb.yaml) lists **`user-data-store`** under **`example-product-slice`**.

**Layer table**

| Step | Layer | What gets decided (still illustrative) |
|------|--------|----------------------------------------|
| 0 | **Lay membership** | “This product includes **user-data-store**.” **No** SQL/file/P2P/vendor. |
| 1 | **Logical role / policy** | Purpose: contact/profile persistence; sensitivity (e.g. PII); **no** engine. |
| 2 | **Persistence class** | Abstract model: relational | document | kv | file | peer | … |
| 3 | **Engine family** | e.g. **postgresql**, or “managed SaaS SQL”—**vendor line** begins. |
| 4 | **Topology** | Single instance, HA replica, serverless, … |
| 5 | **Environment binding** | After **`environment`** at invocation: credential ref, host, **database name**, connection policy. |
| 6 | **Schema contract** | Primary table (**`user_account`**), related tables, migration refs, accessor contract id. |
| 7 | **Static domain data** | Supporting reference tables + seed sources (e.g. **`ref_us_state`**, **`ref_country`** from a versioned **reference pack** if the user model has those properties). |

**Machine-oriented sketch:** [`user-data-store.resolution.example.yaml`](user-data-store.resolution.example.yaml)

**Sibling example (compute path):** [`../resolution-walkthrough/README.md`](../resolution-walkthrough/README.md) (**`authcode-generator`**).
