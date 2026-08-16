# Decision: no thesis forking (use relations → revision → merge)

**Status:** decided. Thesis forking is intentionally *not* built.

## Why not

Argument forks **converge**: one card, one vote per group, the best wording
wins. Thesis forks would **diverge** — a thesis is a container for a whole
discourse (votes + arguments), so forking it fragments the very consensus the
platform exists to find. 40 people agreeing across 8 near-identical thesis
variants means nobody reaches the 20-vote crystallisation threshold. That
inverts the mission.

Also: a "forked thesis" is indistinguishable from a "new, related thesis" — and
we already model that with `related_thesis_ids` (graph edges). So the answer is
a stronger relation graph, not a fork tree.

## The path instead (maturing in three stages)

1. **Relations first** — make "streams" on a topic visible. On create, when the
   similarity search finds a near-duplicate, actively offer "link as related
   thesis?". Visualise related theses as a cluster/stream. Groundwork for the
   rest.
2. **Revision (owner cycle)** — the owner revises wording in place; history
   tracked as a changelog. Convergent: the discourse stays in one container, the
   phrasing improves. (Needs changelog persistence — later phase.)
3. **Merge (active consensus work)** — merge several related theses; participants
   are notified and re-position; the merge event spikes activity so the topic
   resurfaces. Most powerful, most complex (merge semantics for votes/arguments,
   re-positioning flow, abuse potential). Later phase.

## Real-life honesty

There is no single canonical phrasing of a topic; related, coexisting
discourses are natural. We embrace that via a **network of related theses**, not
a **genealogy of copies**.
