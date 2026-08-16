# Architecture

Quappe is built as several repositories under one organisation, around a single
API contract. This is what makes the org structure meaningful — and lets us add
clients (CLI, app, analytics) without touching the core.

## The repositories

| Repo | Role |
|------|------|
| **quappe-service** | The API + storage + all domain logic. The single source of truth. |
| **quappe-web** | The browser UI. Presentation-only; consumes the service API. |
| **quappe-docs** | This — the idea, mechanics, design, decisions. |
| **quappe-ops** | Operational setup (k8s, metrics, logs). Later, when load demands it. |
| **quappe-insight** | Data visualisation / meta reporting. Later. |

## The API is the contract

`quappe-service` exposes `/api/*` and publishes an **OpenAPI 3.1 spec**
(`openapi.yaml`, served at `GET /api/openapi`). Every client builds against it.
Keeping the contract explicit is what allows:

- **quappe-web** today,
- a **CLI** tomorrow (a good CLI is the best proof the API is complete),
- a mobile **app** wrapper,
- **quappe-insight** reading the same data.

## How web talks to the service

`quappe-web` runs a reverse proxy: every `fetch('/api/…')` in the UI is
forwarded to the service (`PRIVATE_SERVICE_URL`). This keeps the browser
same-origin (no CORS) and lets the anonymous JWT identity cookie stay
first-party. In production the proxy target is an internal address. The service
is the only place that mints or verifies identity.

## The data layer (in quappe-service)

- State lives in **SQLite** (`better-sqlite3`, WAL) — a single file that carries
  the platform a very long way (≈ 1 M theses in one Node process).
- **`data.ts` is a façade** (~38 exports) — the only surface consumers import.
  Internals delegate to `src/lib/server/db/*` (one module per table, prepared
  statements). Row↔domain mapping and embedding BLOBs live in `mappers.ts`.
- **Tier is derived, not stored** — hot/warm/cold is just a filter on
  `lifecycle_state`.
- Derived caches (heat, argument counts, activity) live in `data.ts` with a 30 s
  TTL, invalidated on every write.

## Why not a monorepo / why not k8s yet

We split into separate repos deliberately (the platform vision). But the
*operational* infrastructure — k8s, Prometheus, OpenSearch, DB bouncers — is an
answer to load we don't have yet. It's recorded as vision in `quappe-ops`, not
built prematurely. Build an operational capability when a real pain appears, not
in anticipation.

## Identity, budget, abuse guards

- Anonymous, server-minted **JWT** in an httpOnly cookie. Request bodies never
  carry a trusted user id — the cookie is authoritative.
- Daily **budgets** (3 stance buckets à 8 + a weight pool of 21) enforced
  server-side. Per-IP + per-user rate limits, length caps, a 32 KB body cap.
- A freshly-minted identity can't cast weighted votes for its first minute
  (Sybil dampener).

## Embeddings

Semantic search uses `@huggingface/transformers` (server-side, model
`Xenova/multilingual-e5-small`, 384-dim, q8-quantised). Text only — no image
processing. Embeddings never leave the machine.
