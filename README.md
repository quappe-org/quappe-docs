# quappe-docs

The idea behind Quappe — the problem, the mission, the mechanics, and the
design decisions. Written for humans: contributors, curious users, researchers.

> This is the *explanatory* home for concepts that currently live as working
> notes in each code repo's `.meta/*.skill` files. Those stay next to the code
> as the engineering reference; here they become readable documentation.

## Contents

- **[idea.md](./idea.md)** — the problem, the target, the rules, and the stance
  that the system should bend toward the good.
- **[mechanics.md](./mechanics.md)** — theses, arguments, fork variants, voting,
  the Fibonacci budget + complexity model, lifecycle, quality, germination.
- **[design.md](./design.md)** — the UI language (editorial/calm, top-down),
  themes, and the function-not-diagnosis accessibility modes.
- **[architecture.md](./architecture.md)** — the platform split, the
  API-as-contract principle, the data layer.
- **[decisions/](./decisions/)** — the "why", so it isn't re-litigated: no
  thesis forking, no "emotional" type, cookie-first i18n, accessibility naming.

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
