# Use-case: prioritizing a Git backlog with Quappe

**What Quappe is for, here:** turning a software backlog into a structured
discourse. Instead of prioritizing issues in comment threads, meetings, or a
single maintainer's gut feeling, a team brings the backlog into Quappe and lets
its argument-and-vote model surface *which items matter and why*.

This is one concrete application of Quappe's core idea (see
[idea.md](../idea.md)): find as much consent as possible, show disagreement
with respect, and let the best-reasoned positions — not the loudest — carry
weight.

## Why a backlog fits Quappe

A backlog item is already a proposition: *"we should do X."* That is a thesis.
The reasons for and against doing it are arguments. Whether the team should
invest in it is exactly the kind of *defined, coexisting position* Quappe is
built to reach:

- **Direction-neutral visibility** means a strongly-contested item stays as
  visible as a popular one — you see genuine disagreement, not just applause.
- **Qualify yourself:** an opinion without reasoning weighs less. Priorities
  end up backed by arguments, not just up-votes.
- **The germination boost** gives a freshly-raised concern a fair window to be
  heard before the majority view settles — a minority "this will bite us later"
  can still surface.

The output the team wants is **the top arguments** per item — the reasoning the
group actually converged on. That, filtered per topic, is the prioritization
signal.

## The end-to-end flow

1. **A GitHub Project holds the backlog.** In practice this spans *many* repos
   (e.g. one product Project over ~8 repos), with iterations, status, and
   custom fields — the real planning structure.
2. **The bridge imports it into Quappe** as theses. Git's *fixed* fields
   (repo, issue-type, status) become **categories**; its *free* fields
   (iteration, size) and labels become **hashtags** — so you filter the
   backlog along both axes. The import is idempotent and reversible per source.
   During a sprint you just **re-sync** to pull drift (new issues, relabels, an
   item moving iteration/status): the current git state overwrites the
   metadata, while the votes and arguments the team produced are kept. A full
   **reset** is only for starting a fresh iteration. See the mapping + drift
   rules in [decisions/git-source-projects-v2.md](../decisions/git-source-projects-v2.md).
3. **The team discusses and votes.** Arguments accrue; forks converge on the
   best wording; votes (direction-neutral, Fibonacci-weighted) rank the
   reasoning.
4. **The team reads off the top arguments** per item — filtered by iteration,
   category, or tag — as the prioritization result.
5. **The result flows back to Git** — *not yet defined.* Today the bridge is
   deliberately one-way (GitHub → Quappe). Whether and how a decision returns to
   GitHub (a comment, a label, a decision-issue, or a manual export) is an open
   design question, tracked in
   [decisions/git-source-projects-v2.md](../decisions/git-source-projects-v2.md).

## Scope boundary

This document is the *what-for* — the high-level use-case and mission fit. The
*how* — install, configure, run a sync, reset the database, purge a source —
lives with the tool, in the bridge repo's own README and engineering notes, not
here. Keeping the narrative independent of any tool's release cycle is the whole
point of a separate docs repo (see [README](../README.md)).
