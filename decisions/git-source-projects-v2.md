# Decision: import from GitHub Projects v2, not per-repo Issues

**Status:** decided. Projects v2 is the primary source; field mapping and the
drift model are settled (below). The way back to GitHub remains out of scope.

## Context

The [git-backlog use-case](../use-cases/git-backlog-prioritization.md) has a
team maintaining **one GitHub Project as a backlog across many repos** (the
concrete case: ~8 repos under a single product Project). The existing
`quappe-github-bridge` reads the **Issues REST API, one repo at a time**.

## Why Projects v2 is the right source

GitHub Issues and GitHub Projects are two different APIs, and only one carries
the structure this use-case needs:

- **Issues REST** (today's bridge): per-repo. Knows labels and open/closed
  state — but *nothing* about the Project: no iteration, no status column, no
  custom fields, no cross-repo ordering. Syncing 8 repos this way gives 8
  disconnected issue lists and throws away the planning structure that makes it
  a backlog.
- **Projects v2** (GraphQL): a single Project pulls items from *all* linked
  repos at once, **with** iteration, status, and custom fields. This is
  literally the team's sprint backlog.

So for backlog prioritization, Projects v2 isn't merely more convenient — it is
the only source that contains the iteration/status structure the use-case
depends on. A multi-repo issue sync (the config already supports a `repos[]`
array) was never the missing piece.

**Consequence — this is a new adapter, not a config toggle.** GraphQL, wider
auth scopes, different pagination, and draft items that have no backing repo.
The Issue-sync path becomes a fallback for repos with no Project.

**Rename:** once Projects v2 is the primary mode, the tool becomes
`quappe-git-project-bridge`. The name follows the primary source.

## Field mapping: fixed → category, free → hashtag

We do **not** add an iteration field to Quappe's contract. Instead we split
git's fields by **stability**, which lines up neatly with Quappe's two existing
axes:

- **Git *fixed* fields → categories.** Repo, issue-type, and the Project
  **status** (Todo / In Progress / Done). These are finite, known value sets —
  exactly what a curated category facet is for. This works *because* the
  backlog runs on a **dedicated instance with an empty DB**: `DEFAULT_CATEGORIES`
  is not fixed law here, so we define the category axis from the stable git
  structure. (My earlier objection — "categories are a small curated list,
  iterations churn" — is what *rules iterations out* of categories and *rules
  the fixed fields in*.)
- **Git *free* project fields → hashtags.** Iteration and size (free-form
  Projects-v2 fields) plus issue labels. Open, growing, churning — the hashtag
  axis. Since these fields are user-defined, the bridge needs a small
  field-mapping config (`projectField → category | hashtag`); that is bridge
  work, no service change.

**One required service config point:** the import handler normalizes categories
against `DEFAULT_CATEGORIES` and drops anything unknown to `other`. So the
git-fixed values chosen as categories (repos, issue-types, statuses) must be
**added to `DEFAULT_CATEGORIES`** on the instance, or they will collapse to
`other`.

## Living with git drift: re-sync, reset only per iteration

Git keeps changing during a sprint — new issues, relabels, an item moves
iteration or status. This is handled without losing the team's work, because
the import is a **snapshot upsert, not a merge**:

- Import upserts by `external_ref`; on an existing item it **overwrites**
  title / description / categories / hashtags / archived with the current git
  snapshot.
- Votes and arguments hang off the stable thesis id and are **untouched** by a
  re-sync.

So the model is: **`sync` (re-run) to pull drift mid-iteration** — the current
git state wins for metadata, the discourse survives — and a **full reset only
to start a fresh iteration**. No reset is needed just because git moved.

| Drift in git | Re-sync effect | Votes / arguments |
|---|---|---|
| New issue | new thesis created | — |
| New label / iteration value | appears as a new hashtag | kept |
| Iteration or status changed | category / hashtags overwritten (snapshot) | kept |
| Issue closed | thesis archived | kept |

## The way back (Quappe → GitHub) is out of scope for now

The bridge is intentionally one-way; the GitHub token is read-only. Returning a
result to GitHub — a comment with the top arguments, a "prioritized" label, a
decision-issue, or a manual export — is a **separate future feature** with wider
scopes, not part of this pivot.
