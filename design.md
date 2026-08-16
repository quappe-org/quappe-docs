# Design

The UI language of Quappe. Full working spec (with wireframes) lives in
`quappe-web/.meta/.ui.skill`; this is the readable summary.

## Direction: editorial / calm

Quappe is a discourse platform — text *is* the product. So the UI treats text
as a designed element (large serif headlines, generous whitespace, a centred
reading column) rather than something to hide. Principles:

- **Editorial** — serif headlines, small uppercase "eyebrow" meta lines, strong
  type contrast, lots of margin. Text as the design, not the problem.
- **Calm / progressive disclosure** — show the minimum needed to decide; details
  on demand. Chrome (budget, slider, theme, menu) hides in popovers until
  invoked.
- **Two axes of reader control** — amount (Fibonacci complexity slider) and
  density (author reading registers simple/prose/dense). See
  [mechanics](./mechanics.md).
- **Direction-neutral, humane** — support == reject for visibility; a decaying
  germination boost lets good new positions surface.

## Shell: top-down, no sidebar

A sticky top bar over a centred content column (the old dashboard sidebar is
gone). Left: brand. Centre: primary nav (Trending · Top · Mine · Updates ·
Pulse). Right: actions — a `+ New` button and transient popovers for the
complexity slider, the daily budget, the theme/accessibility panel, and an
overflow menu. Popovers close on outside click, Escape, or navigation; on mobile
they become bottom-sheets. One structure for all widths (mobile-first).

## Themes & accessibility — two orthogonal axes

Both are applied as attributes on `<html>`, both persisted, both surfaced in the
header theme popover.

1. **Aesthetic theme** — rainbow (default) · pastel · classic · unicorn ·
   grayscale. A pure palette swap.
2. **Accessibility modes** — independent toggles that combine with any theme:
   - **invert** — flip all colors (dark-ish).
   - **calm** — reduce visual noise: desaturate, flatten shadows, drop card
     hover-lift, dim decorative bands, stop the badge pulse.
   - **high contrast** — stronger text/border contrast.
   - **reduce motion** — neutralise transitions/animations (also respects the
     OS `prefers-reduced-motion`).

### Naming decision (do not "simplify" back)

Accessibility modes are named **by function, never by diagnosis or politics.**
"conservative" became **classic** (muted/editorial — a political label on a
neutral-discourse platform sends the wrong signal); "ADHS/neurodivergent" became
**calm** (naming a mode after a diagnosis reduces people to it — like calling a
ramp a "disabled mode"). "Calm" helps anyone distracted by a busy UI — ADHD,
autism, migraine, or a noisy room — without labelling them. Inclusive, not
labelling. This is *why* the modes exist. See
[decisions](./decisions/).

## Type & tokens

- Fonts: system sans (body), system serif stack (headlines — no web font),
  monospace (numbers/IDs).
- Soft, rounded edges; diffuse low-contrast shadows.
- Accent + vote colors are theme-swappable CSS variables.
- No CSS framework, no UI component library — hand-written CSS.
