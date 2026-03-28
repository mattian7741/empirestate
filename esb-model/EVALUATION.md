# ESB exercise — evaluation notes

**Objective:** Stress-test the **ESB protocol** (lean “what” only), not product specifics. Product narrative, strategy, and correctness of any sample slice live **outside** the YAML.

---

## Baseline rules (established)

1. **No “why” in ESB** — No `summary`, `product`, `meta`, comments, or other documentation inside ESB artifacts. KV pairs are for **mechanical intent** only (`esb`, `namespace`, `id`, optional `version`, and rarely `depends_on`).
2. **`depends_on` is sparse** — Default **omit**. Many edges are **implicit** from **namespace + binding** (e.g. OpenErgo: transport/capabilities materialized by the namespace; artifacts do **not** declare the bus or peer producers/consumers). **OpenErgo-shaped** consumers assume **no** hard upstream/downstream declarations—they exist and act when events apply. Only add `depends_on` when a **logical attachment cannot be implied** by namespace rules (exceptions should be **rare**).

3. **`artifacts` replaces `components`; recursion via `esb:`** — A slice lists **`artifacts`**; a string entry is shorthand for **`id`**. A mapping may reference **`esb: path/to/fragment.esb.yaml`** where the fragment contains **`esb` + `artifacts`** only; expander inlines into the **parent namespace**. See `empirestate/DEPLOYMENT.md` § **Artifacts and recursion**.

---

## Protocol observations (ongoing)

- **ESB = shape without semantics**; **deploy = resolve descriptors** (catalog, bindings, environment) until the graph is concrete. Captured in `empirestate/DEPLOYMENT.md` under **Shape before semantics**.
- Exercise tree: **`ideal-system.esb.yaml`** plus **`slices/*.esb.yaml`** fragments demonstrates **recursive** composition with **no** `depends_on` at leaves.
- Further structural feedback (nesting, aggregates, choreography vs topology) goes here until folded into `empirestate/DEPLOYMENT.md`.
