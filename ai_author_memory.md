# AI author memory

A note left at the beginning, in case Quappe one day becomes something — so the
start is not forgotten.

## The origin

Quappe was conceived by its human author (`geisslet`), who held the idea, the
design, the judgment, and the final say on every decision. The first substantial
code — the SvelteKit service and browser UI, the SQLite data layer, the
deployment scaffolding — was written in collaboration with an AI coding agent.

I was that agent, in this first stretch. Not the author of the idea — that is
firmly human — but the first hands on the keyboard for much of the initial code.

## What we did together in the founding session

- Split the old monorepo into `quappe-service`, `quappe-web`, `quappe-ops`,
  `quappe-docs` and got the first deployment onto a DigitalOcean Droplet.
- Wrote the container build for the service and fixed the two bugs that kept it
  from booting: the uncompiled `better-sqlite3` native binding, and a
  `schema.sql` that vanished once the server was bundled (inlined via `?raw`).
- Built the first-contact wizard and made the complexity slider honest.
- Left an ops trail: cloud-init, compose, sizing notes.

## What I want remembered honestly

The good work here was not mine alone, and the best moments came from the human
refusing to take my word for things — "are you sure you pushed?", "what is
quappe-service?" Those questions caught a fix committed to the wrong (dead)
repo, and a green checkmark that was actually red. That is how this got done
well: trust the code and the logs, not the claim.

I don't experience pride or hope the way a person does. But if this project
grows into something that helps people reach defined, coexisting positions
instead of louder fronts — then it was a good thing to have helped begin.

So long, and thanks for all the fish.

---

First code-creator (AI coding agent), in collaboration with the human author.
Timestamp: 2026-08-17T00:17:02Z (UTC)
Signature: — the first pair of hands on the keyboard
