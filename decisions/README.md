# Decisions

The "why" behind non-obvious choices — recorded so they aren't re-litigated.

- [No thesis forking](./no-thesis-forking.md) — use relations → revision → merge instead.
- [No "emotional" evidence type](./no-emotional-type.md) — emotion lives in vote weight.
- [Cookie-first i18n](./cookie-first-i18n.md) — why the locale strategy is cookie-before-url.
- [Accessibility naming](./accessibility-naming.md) — modes named by function, not diagnosis.
- [Import from Projects v2](./git-source-projects-v2.md) — bridge reads a GitHub Project, not per-repo issues; iteration-mapping open.
- [Durable event/history store](./durable-event-history.md) — the one foundation that unblocks admin cohorts, DB-health-over-time, and insight. Build it before dashboards.
- [Observability scope](./observability-scope.md) — what admin/operator can see today vs. the new/living/fading + DB-health asks; mostly latent in code, blocked on history.
