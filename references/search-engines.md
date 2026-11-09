# Search Engine Notes & Fallbacks

Engine differences that matter when running dorks programmatically.

## Google

The reference engine; most operators (`site:`, `ext:`, `intitle:`, `before:/after:`) behave predictably.

Serves a "sorry" captcha page after a handful of automated queries. Mitigation: batch 3 to 5 dorks per session, vary query structure, use `num=100` to reduce round trips.

`ext:` is more reliable than `filetype:` for raw extensions; `filetype:` understands semantic types (`filetype:pdf`).

## Bing

Honors `site:`, `inurl:`, `intitle:`, `filetype:`; does NOT support `ext:`, `allintext:`, `cache:`.

More scraping-tolerant than Google; good first fallback.

`contains:` (feed URL) is Bing-only: `contains:wp-content` finds sites hosting WordPress paths.

## DuckDuckGo

HTML endpoint (`https://html.duckduckgo.com/html/?q=`) scrapes cleanly without JS.

Supports `site:`, `intitle:`, `inurl:`; no `ext:`, no `cache:`.

Backed by Bing for some results; coverage overlaps, so use it as a Bing mirror rather than a distinct source.

## SearXNG (self-hosted)

Passes operators through to upstream engines and aggregates; useful when a box runs one (this server does: `searxng.skiddph.com`), with `format=json` for scripted queries.

Aggregators dedupe results, which sometimes hides engine-specific differences you want to see.

## Operator matrix

| Operator | Google | Bing | DDG |
|---|---|---|---|
| `site:` | yes | yes | yes |
| `inurl:` | yes | yes | yes |
| `intitle:` | yes | yes | yes |
| `intext:` | yes | partial | partial |
| `ext:` | yes | no | no |
| `filetype:` | yes | yes | yes |
| `cache:` | yes (flaky) | no | no |
| `before:/after:` | yes | no | no |
| `-term` | yes | yes | yes |
| `OR` | yes (caps) | yes | yes |

## Ordering strategy

1. Google first (richest operator set)
2. On captcha or empty: Bing
3. On empty: DDG html endpoint
4. Self-hosted SearXNG when scripted volume matters

Empty results do not mean nothing is there. Engines occasionally drop operators silently; re-run the same dork minus the flaky operator before concluding.
