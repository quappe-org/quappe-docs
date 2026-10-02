# Decision: a durable event/history store is the next foundation

**Status:** decided (direction). The *shape* of the store is open; what is
settled is that this is the **one shared prerequisite** that unblocks admin
observability, DB-health-over-time, and `quappe-insight`. Build the foundation
before the dashboards. Recorded 2026-09-25.

## The observation that forced this

Reviewing the platform across personas (admin, operator, user, bridge-user,
developer), several "missing" complaints turned out to be **the same gap seen
from different angles**:

- *Admin:* "I can't tell how busy it is — new vs. repeating vs. fading users,
  new vs. living vs. fading theses."
- *Operator:* "I have no view of how the DB is doing over time."
- *User:* "The insight is still missing — I wanted a graffiti-DB map view."

None of these can be answered from a point-in-time snapshot. They are all
**questions about change over time**, and the platform keeps only current state.

## What already exists (so we don't rebuild it)

The *current-state* layer is in good shape — the gap is specifically **history**,
not instrumentation:

- **Lifecycle is modelled.** `quappe-service/src/lib/models/lifecycle.ts` —
  `LIFECYCLE_THRESHOLDS` and a deterministic transition engine produce
  `seedling | discussed | contested | crystallized | faded | dormant`. The
  user's "new / living / fading" vocabulary already maps onto these states.
- **Snapshot aggregates exist.** `dbTierStats()`
  (`src/lib/server/db/theses.ts:62`) → hot/warm/cold; `dbCountDistinctVoters()`
  and `dbDailyVoterStats()` (`src/lib/server/db/votes.ts:158,167`);
  `GET /api/stats`, `GET /api/admin/users?days=N`.
- **Prometheus endpoint exists.** `GET /api/metrics` + a hand-rolled emitter
  (`src/lib/server/metrics.ts`) with a `quappe_db_size_bytes` gauge. Request
  counters/histograms are defined but not yet populated on the request path.

What is **absent**: a durable, append-only, ISO-8601-timestamped record of
*transitions* — when a thesis changed lifecycle state, when a user first/last
voted — so that cohorts and trends can be computed after the fact. Votes carry
`cast_at`, which is the one real time-series we have; everything else is derived
live and then forgotten.

## Why this is the foundation (not a feature)

- **Admin cohorts** (new/repeating/fading users) need each user's first- and
  last-activity over time — reconstructable from `votes.cast_at` today, but only
  as a one-off query, not a durable cohort series.
- **Thesis lifecycle trends** (how many crystallize, do germinated minority
  positions actually rise) need the *history of lifecycle transitions*, which is
  never persisted — the engine recomputes current state and discards the path.
- **DB-health-over-time** needs the gauge sampled and retained, not just scraped.
- **`quappe-insight` is explicitly blocked on exactly this** (see
  `quappe-insight/README.md`: "needs *history* … neither exists as a durable
  feed yet"). The "map view" is a view *of* this data.

One store answers all four. Building any one dashboard first would mean
inventing a private slice of history that the next dashboard can't reuse.

## What is deliberately NOT decided here

- **Storage shape** — an append-only `events` table in the same SQLite file, a
  separate analytical DB, or an emitted event stream. (The insight README floats
  "possibly a separate analytical data source"; the ops README floats
  self-hosting on DigitalOcean. Both are downstream of *having* the feed.)
- **Retention / sampling cadence** for the DB-health gauge.
- **Whether insight reads the API or the store directly** — the platform rule is
  "the API is the contract," so a read-only `/api/events` or `/api/history`
  surface is the default assumption unless proven too thin.

## How to apply

When work resumes on metrics/insight/map-view, the **first** change is the
event/history store in (or beside) `quappe-service` — not a dashboard, not a
chart. Everything the admin, operator, and insight personas asked for is a
*reader* of this store. If a task proposes a chart before the store exists, it
is building on sand; push back and build the feed first.
