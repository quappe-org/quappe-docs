# Decision: no "emotional" evidence type

**Status:** decided. The `emotional` evidence type was removed.

## Why

"Emotional" was an evidence type alongside study / authority / logical /
experiential. In practice it had **no impact** — it didn't change voting,
ranking, or budget. Its only effects were a pink badge and *exclusion* from the
evidence-quality bonus. So marking an argument emotional could only **hurt** it
(no evidence credit) and gained nothing — which means nobody would ever use it.
A concept with no consequence is just cognitive load.

## What replaced the need

Emotion is already expressible through **vote weight** (the Fibonacci multi-vote
ladder). Someone who feels strongly casts more weight. That is the honest,
already-present mechanism. One concept less, a clearer "qualify yourself"
vision.

## Result

Evidence types are now `study · authority · logical · experiential`, derived
from the content (URLs), never picked by the user. Existing `emotional`
attributes were migrated to `logical`.
