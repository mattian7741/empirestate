# ESB exercise — evaluation notes

**Objective:** Stress-test the **ESB protocol** (lean “what” only), not product specifics. Product narrative, strategy, and correctness of any sample slice live **outside** the YAML.

---

## Baseline rules (established)

1. **No “why” in ESB** — No `summary`, `product`, `meta`, comments, or other documentation inside ESB artifacts. KV pairs are for **mechanical intent** only (`esb`, `namespace`, `id`, optional `version`, and rarely `depends_on`).
2. **`depends_on` is sparse** — Default **omit**. Many edges are **implicit** from **namespace + binding** (e.g. OpenErgo: transport/capabilities materialized by the namespace; components do **not** declare the bus or peer producers/consumers). **OpenErgo-shaped components** assume **no** hard upstream/downstream declarations—they exist and act when events apply. Only add `depends_on` when a **logical attachment cannot be implied** by namespace rules (exceptions should be **rare**).

---

## Protocol observations (ongoing)

- Exercise file `ideal-system.esb.yaml` is intentionally **topology-only**: namespaces and component `id`s with **no** `depends_on`, to reflect the baseline above.
- Further structural feedback (nesting, aggregates, choreography vs topology) goes here until folded into `empirestate/DEPLOYMENT.md`.
