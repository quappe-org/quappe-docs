# Decision: cookie-first locale strategy

**Status:** decided. Paraglide strategy order is
`cookie preferredLanguage url baseLocale`.

## The bug it fixes

Navigation links are unprefixed (`href="/top"`, not `/de/top`). With a url-first
strategy, following any nav link from a `/de/…` page dropped the prefix, and the
top-priority `url` strategy resolved the unprefixed path back to the base locale
(`en`). The `de` cookie existed but was never reached. Net effect: setting the
language to German, then navigating, silently reverted to English.

## Why this order

- **cookie first** — a remembered choice always wins and stays stable across
  navigation, even on unprefixed links. This is the core fix.
- **preferredLanguage second** — a first-time visitor with no cookie is served
  their browser language (`Accept-Language`). A German browser lands on German
  content at `/` without hunting for a setting — the friendly welcome for a
  multilingual, humanist platform.
- **url third** — explicit `/de/…` prefixes still work for deliberately shared,
  language-specific links.
- **baseLocale last** — English fallback.

## Trade-off

`/` is not deterministically English — it follows the visitor's browser
language (or their cookie). For a multilingual discourse platform that's
desirable, not a defect. Someone who wants to share a specific language links
`/de/…` explicitly.

## Why not localize every link instead

The alternative fix was running every `href` through `localizeHref`. We chose
the strategy order because it's robust for **every** link, current and future,
without anyone remembering to localize hrefs — and it doubles as the friendly
first-visit behaviour above.
