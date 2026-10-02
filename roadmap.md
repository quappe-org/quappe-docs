# Quappe platform roadmap — session-by-session

**Created 2026-09-25.** This is the execution plan behind the
[2026-09-25 platform review](./decisions/durable-event-history.md). It turns the
five-persona observations into ordered, session-sized work with acceptance
criteria. It is a **living plan** — update it as sessions complete; don't let it
rot into fiction.

Read the two decisions first — they are the "why"; this is the "when/how":
- [`decisions/durable-event-history.md`](./decisions/durable-event-history.md) — the foundation.
- [`decisions/observability-scope.md`](./decisions/observability-scope.md) — what admin/operator get, and the new/living/fading naming.

---

## The architecture this roadmap builds toward

A **two-tier, decoupled** topology (the user's design call, 2026-09-25):

```
quappe-service (source of truth, stays thin)
  │  mutations already flow through ONE façade: src/lib/stores/data.ts
  │  lifecycle transitions already computed + logged at data.ts:149/169
  │
  ├─ emit append-only events on every meaningful transition
  │     (thesis.created, vote.cast, lifecycle.changed, thesis.archived, import.*)
  │
  └─ GET /api/events?since=<cursor>   ← thin, cursor-paged, read-only
           ▲
           │  periodic pull + gap-reconcile ("lambda checks hickups,
           │  back-fills what it missed") — the puller is a CONSUMER,
           │  can crash/lag/catch up without touching the service
           ▼
   quappe-insight analytics DB (separate, can live on DigitalOcean)
     - cohort tables (user first/last activity → new/repeating/fading)
     - lifecycle-transition history (→ living/fading trends)
     - DB-health samples over time
           ▲
           │  reads
           ▼
   dashboards + the graffiti-DB map view
```

**Why this shape:** the service never carries analytical load or schema; it just
appends events and serves them by cursor. The analytics side is a replaceable
consumer. This honours the platform rule "the API is the contract" and keeps
`quappe-service` presentation-free and `quappe-insight` service-free.

**Deployment topology (decided 2026-09-25):** `insight` is exposed as its own
**subdomain `insight.quappe.org`**; `ops`/admin is exposed as a **path
`quappe.org/admin/ops`** (NOT a subdomain). Reason: the httpOnly identity/admin
cookie (`quappe_uid`) is first-party on `quappe.org` and is NOT sent to a
separate origin — so ops/admin, which *needs* that admin auth, lives under the
main origin to inherit it for free; insight is a standalone consumer that can
carry its own (or public) auth, so a clean subdomain fits. Concretely on the
DigitalOcean single-host (Caddy auto-TLS, see `quappe-ops/deploy`): add a DNS
A-record `insight` → droplet IP (or a `*.quappe.org` wildcard), a Caddyfile block
`insight.quappe.org { reverse_proxy insight:3000 }`, and an `insight` service in
`docker-compose.yml` (mirroring `web`). The ops path needs no DNS/Caddy change —
it's a route under the existing web app, admin-gated.

**Key fact that makes tier-1 cheap:** every mutation already passes through
`data.ts`, and `reevaluateLifecycle` (`data.ts:149`) already *logs* the
transition it currently discards (`data.ts:169`). We are persisting an event
that is already being computed — not adding new domain logic.

---

## Two parallel tracks

Track **A (foundation)** and track **B (bridge polish)** are independent — B
does not touch the event store. Run B early/in parallel (the Projects-v2 import
is live with ~726 theses, so this is polish on a running system, not greenfield).

---

## Track A — the foundation and its readers

### A0 · Decide the event-store contract  *(½ session, decision + stub)*  — ✅ DONE 2026-09-25
**Goal:** nail the tier-1 contract so everything downstream is stable.
- Decide the event envelope: `{ id (monotonic cursor), ts (ISO-8601), type, subject_id, actor_id?, payload }`.
- Decide persistence: append-only `events` table in the existing SQLite file
  (default; simplest, degrades well). The *separate* analytics DB is tier-2 (A4),
  NOT this step — don't conflate them.
- Decide the event type set (start minimal): `thesis.created`, `vote.cast`,
  `lifecycle.changed`, `thesis.archived`, `import.batch`.
- Write it into `quappe-service/openapi.yaml` as the `/api/events` shape (spec
  first — the contract test enforces route↔spec parity).

**Acceptance:** openapi has `/api/events`; a decision note in
`quappe-service/.meta` (or an ADR) fixes the envelope + type set; `npm run check`
clean. No emission yet.

**Landed:** `GET /api/events` route stub (`src/routes/api/events/+server.ts`) —
admin-gated, parses `since`/`limit`, returns contract-shaped empty page
`{ events: [], next_cursor: null }`, no persistence yet. openapi.yaml carries the
path + `Event`/`EventType` schemas (envelope frozen in the schema +
`decisions/durable-event-history.md`). Contract test green (3 passed),
`npm run check` 0 errors. **Deliberately deferred to A1:** the `events` table,
`appendEvent`, and real emission.

### A1 · Emit events from the façade  *(1 session)*
**Goal:** every meaningful mutation appends an event — at the ONE choke point.
- Add `appendEvent(...)` on the SQLite side; call it from `data.ts` in:
  `createThesis` (263), `voteOnThesis` (409), `reevaluateLifecycle` (149 — reuse
  the existing transition detection at 169), `archiveThesis` (350),
  `importThesis` (263)/bulk import.
- Events are append-only; never updated/deleted (except `resetAllData`).
- Keep emission synchronous + in the same transaction as the mutation so an
  event can't be lost vs. its cause.

**Acceptance:** a test in `tests/api/` casts a vote / creates a thesis / forces a
lifecycle change and asserts the matching event rows exist with correct
`type`/`subject_id`/`ts`. `cast_at`-era data is NOT back-filled (document that
history starts at deploy).

### A2 · Serve events by cursor  *(½ session)*
**Goal:** the thin read surface the puller consumes.
- `GET /api/events?since=<cursor>&limit=N` → ordered, cursor-paged, admin/
  import-secret-gated (reuse existing guard pattern).
- Monotonic `id` is the cursor; document that `ts` is for display, `id` for paging.

**Acceptance:** openapi + contract test; a test pages through events with `since`
and gets no gaps/dupes.

### A3 · Admin-facing cohort + lifecycle-trend queries  *(1 session)*
**Goal:** answer the admin's new/repeating/fading (users) and new/living/fading
(theses) — reading events, reusing `lifecycle.ts` tiers (do NOT invent states).
- User cohorts over a window: new (first-activity in window), repeating
  (activity in window AND before), fading (last-activity before window, none in).
- Lifecycle trend: counts per tier over time from `lifecycle.changed` events;
  label with the agreed vocabulary (seedling=new, discussed|contested|
  crystallized=living, faded(+dormant)=fading).
- Expose via admin endpoints (service), NOT a UI yet.

**Acceptance:** endpoints return cohort + trend JSON; tests seed events and
assert the buckets; naming matches `observability-scope.md`.

### A4 · Analytics DB + puller/reconciler  *(1–2 sessions, in quappe-insight)*
**Goal:** tier-2 — the separate consumer the user described.
- Standalone process in `quappe-insight`: pulls `/api/events?since=<lastCursor>`
  on an interval, writes into its own DB (SQLite to start; the "DigitalOcean /
  separate analytical source" option stays open — the puller doesn't care).
- **Reconcile/"hickup" logic:** persist last cursor; on each run request from it;
  if the service was unreachable or the puller was down, it simply resumes and
  back-fills the gap. Idempotent upserts keyed on event `id`.

**Acceptance:** kill the puller mid-stream, restart → it catches up with no gaps/
dupes; analytics DB row count == service event count for the covered range.

### A5 · DB-health over time  *(½ session)*
**Goal:** operator's "how is the DB doing over time".
- Either sample `quappe_db_size_bytes` (+ event rate) into the analytics DB on
  the puller's tick, or stand up the Prometheus scrape loop in `quappe-ops`
  (its README already reserves this for "when load demands it").
- Decide retention cadence here (left open in the decision doc).

**Acceptance:** a time series of DB size + activity exists and is queryable.

### A6 · Insight dashboards + the graffiti-DB map view  *(2+ sessions, quappe-insight)*
**Goal:** the user's visual payoff, now that history exists.
- Trend charts (cohorts, lifecycle tiers over time) from the analytics DB.
- **Map view:** spatial/graph of theses + relations. Decide: full insight app,
  or an `/explore` route embedded in quappe-web reading insight's API. (A
  current-state-only client graph in quappe-web is the stopgap noted in
  `quappe-web/.meta/.ui.skill` — only if the user wants something before A4.)

**Acceptance:** per-dashboard, defined when A3/A4 land (data shapes known).

---

## Track B — bridge topic/hashtag polish  *(1 session, standalone)*

Fixes the "default topics/hashtags not reliably replaced" complaint. Detail +
exact file:lines in `quappe-github-bridge/CLAUDE.md` ("Topic/hashtag mapping").
- Give **Projects mode** a `labelCategoryMap`-equivalent (override/exclude a
  label; currently only Issues mode has it — `mapper.ts:95` vs `:37`).
- **Log** labels that normalise to empty instead of dropping silently
  (`mapper.ts:58,107`).
- Decide whether a **default-hashtag** rule should exist; if yes, make it config
  + code (today it exists only in memory, not in the mapper).

**Acceptance:** a mapper test proves override/exclude works in Projects mode and
empty-normalising labels are logged; `npm run check` clean. Re-sync the live
board and confirm idempotence (0 created / N updated) — matches prior verified
behaviour.

---

## Track C — operator ergonomics  *(½ session, low priority, opportunistic)*

The "`dev:all` isn't good usability" complaint. No canonical home yet — a dev
wrapper would live at workspace level or in `quappe-ops`.
- One command that starts service (`dev:all`) + web with `PRIVATE_SERVICE_URL`
  preset, in parallel. A `Makefile`/`dev.sh` at the workspace root, or a
  `quappe-ops` script.
- Not blocking anything; slot it when context is already in ops/startup.

**Acceptance:** one command brings the whole stack up locally; README updated.

---

## Track D — UI consistency + 2026 refresh  *(1–2 sessions, after/independent of A)*

Detail in `quappe-web/.meta/.ui.skill` ("Known UI debt").
- **D1 consistency pass:** replace hardcoded gaps with `--space-*` in
  `top/+page.svelte` + `ThesisCard.svelte`; re-check all views together (the
  "objects look identical" rule). *(½ session)*
- **D2 token refresh:** re-tune radii/shadows/depth toward 2026 without breaking
  the calm aesthetic or `prefers-reduced-motion`. *(1 session, needs human eyes)*

---

## Suggested ordering

1. **A0 + A1 + A2** (the store + emit + read) — the unblocker. Do first.
2. **B** in parallel (independent, polish on a live system).
3. **A3** (admin cohort/trend endpoints) — first real payoff of the store.
4. **A4 + A5** (analytics DB puller + health series) — tier-2.
5. **A6** (dashboards + map view) — the visual goal.
6. **D1/D2, C** — slot opportunistically; not on the critical path.

Rule of thumb from the review: **no chart before the store exists.** If a future
session reaches for a dashboard and A1/A2 aren't done, it's building on sand.
