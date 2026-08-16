# Running Quappe

Quappe is two runnable apps — **quappe-service** (the API + DB) and
**quappe-web** (the UI) — plus the docs/ops/insight repos. This is the single
place that explains how to run the platform; the code repos link here.

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

## Production build (standalone Node)

Both apps use `@sveltejs/adapter-node` → a self-contained server in `build/`.

```bash
npm run build && node build      # listens on PORT (default 3000)
```

## Docker

Each repo ships a multi-stage `Dockerfile` producing a small Node runtime image.

```bash
# Service — SQLite + model cache live on a volume
docker build -t quappe-service ./quappe-service
docker run -p 3000:3000 \
  -e QUAPPE_SECRET=change-me \
  -v quappe-data:/data \
  quappe-service

# Web — point it at the service
docker build -t quappe-web ./quappe-web
docker run -p 8080:3000 \
  -e PRIVATE_SERVICE_URL=http://host.docker.internal:3000 \
  quappe-web
```

### Published images

CI (GitHub Actions) builds and pushes to Docker Hub on every push to `main`
(tag `latest` + short SHA) and on version tags `v*` (semver tags). Images:

- `DOCKERHUB_USER/quappe-service`
- `DOCKERHUB_USER/quappe-web`

The pushing repos need two secrets: `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`
(a Docker Hub access token).

## Kubernetes (later)

Manifests/Helm will live in **quappe-ops** once real load demands it. The shape:
`quappe-service` as a Deployment with a persistent volume for the SQLite file +
model cache; `quappe-web` as a stateless Deployment with `PRIVATE_SERVICE_URL`
set to the in-cluster service DNS; an Ingress terminating TLS in front of web.
Not built prematurely — see `quappe-ops`.
