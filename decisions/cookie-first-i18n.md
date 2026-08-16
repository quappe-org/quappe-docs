# Decision: cookie-first locale strategy

**Status:** decided. Paraglide strategy order is
`cookie url preferredLanguage baseLocale` (cookie before url).

## The bug it fixes

Navigation links are unprefixed (`href="/top"`, not `/de/top`). With a url-first
strategy, following any nav link from a `/de/…` page dropped the prefix, and the
top-priority `url` strategy resolved the unprefixed path back to the base locale
(`en`). The `de` cookie existed but was never reached. Net effect: setting the
language to German, then navigating, silently reverted to English.

## Why cookie-first (not localizing every link)

Two ways to fix it: run every `href` through `localizeHref`, or reorder the
strategy. We chose reorder because:

- For an anonymous app, the **remembered preference** (cookie) is the natural
  model.
- It's robust for **every** link, current and future, without anyone remembering
  to localize hrefs.
- Explicit `/de/` prefixes still work (for shareable links); no cookie → `en`
  default still holds.

The trade-off — a shared `/de/…` link opened by someone whose cookie is `fr`
shows `fr` — is rare and acceptable for an anonymous platform.
