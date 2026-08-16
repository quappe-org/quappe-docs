# Running Quappe

Quappe is two runnable apps — **quappe-service** (the API + DB) and
**quappe-web** (the UI) — plus the docs/ops/insight repos. The two ways that
matter: **local** (development) and **Kubernetes** (production). Docker is the
packaging step in between.

## Local development

Run the service first, then the web app pointed at it.

```bash
# Terminal 1 — the API on :5273 (seeds ~200 demo theses on first request)
cd quappe-service
npm install
npm run dev

# Terminal 2 — the UI on :5173, proxying /api/* to the service
cd quappe-web
npm install
PRIVATE_SERVICE_URL=http://localhost:5273 npm run dev
```

Open http://localhost:5173. For the LLM features (pulse, translation, variant
drafting) also run `ollama serve` (or `npm run dev:all` in the service).

Useful service env: `QUAPPE_SECRET` (JWT secret — set it so identities survive
restarts), `QUAPPE_DB_PATH` (default `.data/quappe.db`),
`OLLAMA_URL`/`OLLAMA_MODEL`/`OLLAMA_TIMEOUT`.

## Kubernetes (production target)

The intended production shape. Manifests/Helm live in **quappe-ops** (built out
when load demands it — not prematurely). The topology:

- **quappe-service** — a Deployment with a **PersistentVolume** for the SQLite
  file + embedding-model cache (mounted at `/data`). `QUAPPE_SECRET` from a
  Secret. Runs the image `DOCKERHUB_USER/quappe-service`.
- **quappe-web** — a stateless Deployment, `PRIVATE_SERVICE_URL` set to the
  in-cluster service DNS (e.g. `http://quappe-service:3000`). Image
  `DOCKERHUB_USER/quappe-web`.
- **Ingress** terminating TLS in front of `quappe-web`; the service is
  cluster-internal only.
- Scale `quappe-web` horizontally freely; `quappe-service` is single-writer
  today (SQLite) — scale it out only after moving to Postgres (see quappe-ops).

Images come from Docker Hub, pushed by CI (below).

## Docker (packaging)

Each repo ships a multi-stage `Dockerfile` → a small Node runtime image
(`adapter-node`, listens on `PORT`, default 3000).

```bash
# Service — SQLite + model cache on a volume
docker build -t quappe-service ./quappe-service
docker run -p 3000:3000 -e QUAPPE_SECRET=change-me -v quappe-data:/data quappe-service

# Web — pointed at the service
docker build -t quappe-web ./quappe-web
docker run -p 8080:3000 -e PRIVATE_SERVICE_URL=http://host.docker.internal:3000 quappe-web
```

### Published images (CI)

GitHub Actions builds and pushes to Docker Hub on every push to `main`
(`latest` + short SHA) and on version tags `v*` (semver). Images:
`DOCKERHUB_USER/quappe-service`, `DOCKERHUB_USER/quappe-web`.

The pushing repos need two secrets: `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`
(a Docker Hub access token).

## Plain Node (no container)

Both apps also run as a standalone Node server:

```bash
npm run build && node build      # listens on PORT (default 3000)
```

