# Resolution walkthrough — lay ESB → concrete stack (example)

**Illustrative only.** One leaf from [`ideal-system.esb.yaml`](../ideal-system.esb.yaml) — **`authcode-generator`** — traced **down** through example descriptor layers until **Docker**. Real catalogs, schemas, and module names will differ; this shows **how layers hang together**, not production truth.

## Layer chain (read in order)

| Step | Layer | Role | File |
|------|--------|------|------|
| 0 | **Lay ESB** | *What* belongs to which named system (recursive string tokens). | [`../ideal-system.esb.yaml`](../ideal-system.esb.yaml) — `authcode-generator` under `open-ergo-auth-bundle` |
| 1 | **Catalog / manifest** | Maps logical token → **deploy kind**, pointers to implementation artifacts, binding keys. | [`01-catalog/authcode-generator.resolve.example.yaml`](01-catalog/authcode-generator.resolve.example.yaml) |
| 2 | **OpenErgo component config** | *How* the worker behaves on the bus: library, function, I/O bindings, logical **network** attachment (OpenErgo sense — disambiguated from lay **domain** elsewhere). | [`02-openergo/authcode-generator.component.example.yaml`](02-openergo/authcode-generator.component.example.yaml) |
| 3 | **Expanded spec** | Machine-normalized merge of catalog + **environment** + bindings (what a realizer consumes). | [`03-expanded-spec/authcode-generator.expanded.example.yaml`](03-expanded-spec/authcode-generator.expanded.example.yaml) |
| 4 | **Ansible (example realizer)** | Idempotent tasks toward **measured** state: image present, container running, config mounted. | [`04-ansible/authcode-generator.play.example.yml`](04-ansible/authcode-generator.play.example.yml) |
| 5 | **Docker / OCI** | Immutable **runnable**: runtime + OpenErgo SDK + shipped library + **component YAML** baked in or mounted. | [`05-docker/Dockerfile.example`](05-docker/Dockerfile.example) |

## Resolution direction

```text
Lay domains/artifacts  →  catalog lookup (per token)  →  OpenErgo YAML + image ref
       →  expanded spec (+ env + bindings)  →  Ansible  →  docker pull/run  →  running worker
```

**Not shown here:** CI that builds the image, secrets injection, health checks, or non-Docker kinds (Play Store, Neon, …) — same **slot** in the chain, different `deploy_kind` and realizer.
