# Decision: observability & admin-insight scope

**Status:** scoped (not built). Captures what an admin/operator can see today,
what they asked for, and which parts are blocked on the
[durable event/history store](./durable-event-history.md). Recorded 2026-09-25.

## The asks (admin + operator personas)

1. **Activity cohorts** — unique users split into **new / repeating / fading**.
2. **Thesis population** — **new / living / fading** counts, as a living figure.
3. **DB health / load** — is the database healthy, how is it trending (candidate
   to host internally, e.g. DigitalOcean, if it becomes a real service).

## Mapping each ask to current reality

| Ask | Exists today | Gap |
|---|---|---|
| User cohorts (new/repeating/fading) | distinct voters + daily voter/vote counts: `dbCountDistinctVoters`, `dbDailyVoterStats` (`src/lib/server/db/votes.ts:158,167`); `GET /api/admin/users?days=N` | **no cohort split.** "New/repeating/fading" needs per-user first- & last-activity bucketed over time — derivable from `votes.cast_at` but not yet computed or retained |
| Thesis new/living/fading | lifecycle engine + states in `src/lib/models/lifecycle.ts`; `dbTierStats()` → hot/warm/cold (`src/lib/server/db/theses.ts:62`); `GET /api/stats` | **vocabulary + trend.** hot/warm/cold already *is* new-ish/living/fading, just unlabelled for the admin and only as a snapshot — no trend of transitions |
| DB health/load | `GET /api/metrics` Prometheus text; `quappe_db_size_bytes` gauge; `metrics.ts` has counter+histogram infra | **not populated + not retained.** request counters/histograms exist but aren't incremented on the request path; the gauge is scrape-time only, nothing stores the series |

**Takeaway:** the admin's three asks are ~70% latent in the code already. The
missing 30% is almost entirely *history* + *presentation*, not new domain logic.

## The "new/living/fading" naming

Adopt the user's vocabulary as the **admin-facing** label for the existing
lifecycle tiers, so the admin view and the engine speak the same language:

- **new** → `seedling`
- **living** → `discussed | contested | crystallized`
- **fading** → `faded` (and `dormant` as "cold/archived" past fading)

This is a labelling decision, not a model change — do **not** add new lifecycle
states. The thresholds stay the single source of truth in
`LIFECYCLE_THRESHOLDS`.

## Where each piece lives (the split)

- **Aggregation / cohort queries** → `quappe-service` (new read endpoints over
  the event/history store; the API stays the contract).
- **Visualisation** (dashboards, the trend charts, the map view) →
  `quappe-insight`, which is blocked until the store exists.
- **Raw health scrape** → already `quappe-service`'s `/api/metrics`; retention/
  alerting is a `quappe-ops` concern "when load demands it" (per its README),
  not a service concern.

## Prerequisite

Cohorts and trends are **blocked on the event/history store** — see that
decision. DB-health *snapshot* is not blocked (the gauge works); only
*health-over-time* needs the store (or a Prometheus scrape loop in ops).

## How to apply

When building the admin-insight view: reuse `lifecycle.ts` tiers and the
existing vote/tier aggregates — don't invent parallel ones. Add the cohort/
trend math as readers over the history store. Keep visualisation out of
`quappe-service`; it belongs in `quappe-insight`.
