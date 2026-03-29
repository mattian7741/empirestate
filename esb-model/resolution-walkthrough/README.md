# Resolution walkthrough — lay ESB → concrete stack (example)

**Illustrative only.** **`authcode-generator`** traced **down** through example layers until **Docker**. The repo’s **minimal** lay file is [`../ideal-system.esb.yaml`](../ideal-system.esb.yaml) (**`user-data-store`** only); this walkthrough keeps **`authcode-generator`** as a **fixed compute-path** example—treat it as listed under some domain in a fuller lay corpus. For a **data-path** matrix see [`../resolution-matrix/README.md`](../resolution-matrix/README.md).

## Layer chain (read in order)

| Step | Layer | Role | File |
|------|--------|------|------|
| 0 | **Lay corpus** | *What* belongs to which named system (string token only). | Hypothetical listing of **`authcode-generator`**; compare [**`user-data-store`** lay sample](../ideal-system.esb.yaml) |
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
