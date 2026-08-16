# quappe-docs

The idea behind Quappe — the problem, the mission, the mechanics, and the
design decisions. Written for humans: contributors, curious users, researchers.

> This is the *explanatory* home for concepts that currently live as working
> notes in each code repo's `.meta/*.skill` files. Those stay next to the code
> as the engineering reference; here they become readable documentation.

## Contents (planned)

- **`idea.md`** — the problem (language is imprecise, social media breeds
  fronts) and the target (an argumentation graph toward defined, coexisting
  positions). From `.meta/.project`.
- **`mechanics.md`** — theses, arguments, fork variants, voting, the Fibonacci
  budget + complexity model, lifecycle, quality score, germination. From
  `.meta/.project` + `.meta/.data.skill`.
- **`design.md`** — the UI language (editorial/calm, top-down), themes and the
  function-not-diagnosis accessibility modes. From `.meta/.ui.skill`.
- **`architecture.md`** — the platform split (service/web/ops/insight/docs),
  the API-as-contract principle, the data layer.
- **`decisions/`** — the "why", so they aren't re-litigated: no thesis forking
  (relations → revision → merge), why "emotional" was removed, why cookie-first
  i18n, the accessibility naming decision.

## Why a separate docs repo

The org groups the platform by concern. Keeping the narrative in one place —
independent of any single service's release cycle — lets the *idea* be cited,
linked, and evolved on its own. It also makes the mission legible to people who
will never read the code.

## Platform

Part of the Quappe platform: **quappe-service** (API/DB/logic) ·
**quappe-web** (UI) · **quappe-ops** (operations) · **quappe-insight**
(analytics) · **quappe-docs** (this).

## License

PolyForm Noncommercial 1.0.0 — see [`LICENSE`](./LICENSE).
