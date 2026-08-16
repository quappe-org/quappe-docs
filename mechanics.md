# Mechanics

How Quappe's principles (see [idea.md](./idea.md)) become concrete features.
Numbers here mirror the working reference in `quappe-service/.meta/.data.skill`
and `.meta/.project` — that's the source of truth if the two ever drift.

## Theses

A **thesis** is the central entity — a claim, question, or subject to discuss.

- If your thesis probably already exists but framed differently, start from the
  existing one and link, rather than duplicating.
- Theses can depend on / relate to other theses (graph edges).
- A thesis is never "closed" — it evolves; history is tracked.
- The author can edit their own thesis. An admin can **archive** (never delete):
  archived theses stay reachable by link but leave trending/top lists.
- Every thesis carries a **lifecycle state** and a **quality score** (below).

## Arguments and fork variants

- Arguments **support** or **reject** a thesis (stance is mandatory).
- Evidence type is **derived from the content** (URLs → study / authority /
  logical), not picked by the user. Types: `study · authority · logical ·
  experiential`. (There is no "emotional" type — see
  [decisions](./decisions/).)
- **No reply chains.** Instead of replying, you **fork**: copy an argument,
  adapt it, submit as a new *variant* in the same stance. The root plus its
  variants form one **argument group**.
- A group renders as **one card**: the leading variant is shown, and it expands
  into a radio-list picker. A user has **one vote per group** — voting a variant
  moves the vote off its siblings. Forking migrates the author's vote to the new
  variant.

## Voting

- **Support / Reject / Neutral**, for theses and arguments.
- Each anonymous user votes once; casting the same vote again retracts it.
- **Vote direction is treated neutrally for visibility** (support == reject). We
  do not assume reject = "bad" — a contra-argument can be up-voted for quality.
  The majority sorts meaning.
- Reading and base voting (weight 1) are always **free**.

## Weighting — the Fibonacci ladder

A vote's weight walks a Fibonacci ladder: **1 → 2 → 3 → 5 → 8** (then resets).
A strong opinion costs disproportionately more, which naturally dampens
weight-stacking abuse. Only the weight *beyond* 1 draws from the daily pool.

Beyond raw votes, positions carry qualitative weight from linked evidence,
coherent reasoning, and peer endorsement (vouching for structure, not
conclusion). Unqualified opinion is visible but weighs less.

## Input budget (Fibonacci-flavoured)

Participation is a currency, **enforced server-side** (the client is only an
optimistic mirror). Reset at UTC midnight:

| Bucket | Daily limit |
|--------|-------------|
| New theses | 8 |
| Support arguments | 8 |
| Reject arguments | 8 |
| Vote-weight pool (extra weight points) | 21 |

Reading, exploring and base voting are free. Editing your own content is free.
Forking spends the matching stance bucket (it's a new variant). A freshly-minted
identity may cast base votes only for its first minute (a Sybil dampener) — no
weighted votes until it matures. The budget is generous: a nudge, not a gate.
The point — input must have value. This is not a comment section.

## Lifecycle

A thesis moves through a deterministic lifecycle — a pure function of age,
votes, arguments, and last activity (no history required).

| State | Meaning | Tier |
|-------|---------|------|
| `seedling` | New (< 7 d), gathering first reactions | hot |
| `discussed` | Healthy activity, no clear consensus yet | hot |
| `contested` | Active but polarised (split ≤ 20 %) | hot |
| `crystallized` | Clear majority (≥ 65 %) + ≥ 3 arguments | hot |
| `faded` | Activity dropped, no consensus | warm |
| `dormant` | Inactive 90+ days | cold |

Key thresholds: seedling ≤ 7 d; ≥ 5 votes to leave seedling; crystallized needs
≥ 20 votes, ≥ 65 % consensus, ≥ 3 arguments; contested when the support/reject
split is within 20 %; faded after 30 d idle; dormant after 90 d.

## Quality score

`quality = 0.15·consensus + 0.35·argument_depth + 0.25·engagement + 0.25·evidence`

Deliberately, **quality is discourse depth and groundedness, not agreement.** A
controversial-but-well-argued thesis scores high; consensus is only a weak
signal, so we neither reward echo chambers nor punish honest disagreement.
`/top` includes contested theses.

## Germination — letting the good sprout

Young theses/arguments get a **decaying visibility bonus** (peak 8, ~1.5-day
half-life) on trending/top ranking. This is the structural nudge so a good *new*
minority position can rise beside an established majority — even if the current
majority is "wrong", better ideas get a fair window to grow. It rewards
freshness, not a side (direction-neutral).

## Two axes of reader control

- **Amount** — the Fibonacci complexity slider (theses 3·5·8·13·21·34·55,
  arguments 1·2·3·5·8). Discrete steps help people focus instead of drowning in
  a continuum.
- **Density** — author-provided reading registers for a thesis description:
  *simple · prose · dense* (prose = the original). The view follows the slider
  and falls back to the original. At *simple*, the vote UI also simplifies
  (support/reject only, no neutral/weight, no variant picker).

## Surfacing & signals

- **Commonality search** surfaces where theses overlap; consent is highlighted,
  disagreement shown with respect. Similarity via a local embedding model.
- **Heat** — a thesis's colored edge reflects 24h activity relative to the
  platform average (hot ≥ 1.5× · warm ≥ 0.75× · cool > 0 · cold none).
- **Activity streamgraph** — support (green) / creates (purple) / neutral (gray)
  / reject (red), 5-day rolling average over ~12 weeks.
- **Categories** — each thesis has 1..n; on the landing page they're size-scaled
  tiles for drill-down.
