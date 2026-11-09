---
name: google-dorking
description: Use when the agent needs to find exposed data, endpoints, credentials, vulnerable services, or OSINT intelligence via search-engine dorking (Google/GHDB syntax, Bing/DuckDuckGo fallbacks). Covers dork construction, operator reference, curated dork library, safe execution flow, and result handling.
version: 1.0.0
author: eru123
license: MIT
metadata:
  hermes:
    tags: [osint, security, recon, dorking, search]
---

# Google Dorking

Search-engine dorking finds pages that indexing should never have exposed: open dashboards, credential files, API keys, unlisted endpoints, misconfigured buckets. This skill is the agent's operating manual for building and running dorks safely and turning results into actionable recon.

## When to use

- Recon on a target domain (authorized pentest, bug bounty, own infrastructure audit)
- Checking whether YOUR OWN properties leak secrets (files, endpoints, keys, buckets)
- OSINT: finding documents, org charts, exposed panels, leaked code mentioning a target
- Finding attack surface: API routes, admin panels, staging sites, login portals

**Authorization gate:** before dorking a domain you do not own, confirm an explicit bug-bounty scope or written authorization exists. Dorking itself is just searching, but acting on results (requesting URLs en masse, probing endpoints) is active recon — only proceed against in-scope targets.

## Operator reference (Google)

| Operator | Meaning | Example |
|---|---|---|
| `site:` | restrict to domain/site | `site:example.com` |
| `inurl:` | term in URL | `inurl:admin` |
| `intitle:` | term in page title | `intitle:"index of"` |
| `intext:` / `allintext:` | term in body | `intext:"api_key"` |
| `ext:` / `filetype:` | file extension/type | `ext:env`, `filetype:sql` |
| `cache:` | Google's cached copy | `cache:example.com/secret` |
| `before:` / `after:` | date bounds | `before:2024-01-01` |
| `-term` | exclude | `-site:pinterest.com` |
| `"exact"` | exact phrase | `"internal use only"` |
| `OR` / `\|` | either | `inurl:api \| inurl:rest` |
| `*` | wildcard in phrase | `"password: *" @gmail.com` |
| `related:` | similar sites | `related:cnn.com` |
| `link:` (deprecated) | mostly dead — don't rely on it | |

Combination rules: operators chain left-to-right, `OR` needs caps, grouping with `( )` is unreliable — prefer separate queries. Google suppresses obvious dorking with an apology page; rotate query structure, don't just retry.

## Search execution flow

1. **Construct** the dork from the library below, substituting the target domain.
2. **Fetch results.** Prefer `web_search` with the dork as the query — it passes operators through to the backend. For Google-specific behavior use `https://www.google.com/search?q=<url-encoded dork>&num=100` via `web_extract` (parse result links from the markdown). Bing fallback: same query on `https://www.bing.com/search?q=`; DuckDuckGo: `https://html.duckduckgo.com/html/?q=` (lite endpoint, scrapes cleanly).
3. **Dedupe** result URLs, strip query strings, classify by category (files / endpoints / panels / leaks).
4. **Triage** each hit: fetch with `web_extract`; grade severity (see Result handling).
5. **Report** findings with the exact dork that produced them — dorks are reproducible evidence.

Rate-limit reality: aggressive sequential Google scraping gets captcha-blocked fast. Batch 3–5 dorks per pass, space them, fall back to Bing/DDG. Never hammer.

## Core dork library

Substitute `example.com` with the target. Curated from GHDB, taksec's bug-bounty set, Proviesec, and field experience.

### Discovery — attack surface mapping
```
site:example.com -www -shop -share -ir -mfa
site:example.com ext:php inurl:?
site:example.com inurl:api | site:*/rest | site:*/v1 | site:*/v2 | site:*/v3
site:*.*.example.com                     # sub-subdomains Google knows about
site:example.com intitle:"login" | intitle:"sign in" | intitle:"admin"
```

### Exposed files & juicy extensions
```
site:example.com ext:log | ext:txt | ext:conf | ext:cnf | ext:ini | ext:env
site:example.com ext:bak | ext:backup | ext:old | ext:swp | ext:~ | ext:orig
site:example.com ext:sql | ext:xml | ext:json | ext:yml | ext:yaml
site:example.com ext:git | ext:svn | ext:htpasswd | ext:htaccess
site:example.com intitle:"index of" "parent directory"
intitle:"Index of" site:example.com ".git"
```

### Credentials & secrets
```
site:example.com intext:"api_key" | intext:"apikey" | intext:"api_key"
site:example.com intext:"AKIA"                      # AWS access key IDs
site:example.com intext:"password" ext:env | ext:txt | ext:log
site:example.com intext:"jdbc:mysql" | intext:"jdbc:postgresql"
site:example.com intext:"BEGIN RSA PRIVATE KEY" | intext:"BEGIN PRIVATE KEY"
site:example.com intext:"sk_live" | intext:"pk_live" | intext:"sk_test"
site:example.com intext:"smtp" intext:"password" ext:ini | ext:conf
site:example.com intext:@gmail.com | intext:@outlook.com ext:txt | ext:xls
```

### Panels & dashboards
```
site:example.com inurl:admin | inurl:administrator | inurl:dashboard
site:example.com inurl:phpmyadmin | inurl:adminer | inurl:pgadmin
intitle:"Kibana" | intitle:"Grafana" site:example.com
intitle:"Jenkins" site:example.com
inurl:server-status | inurl:server-info site:example.com
site:example.com inurl:wp-admin inurl:admin-ajax.php
```

### Documents & data spills
```
site:example.com ext:doc | ext:docx | ext:xls | ext:xlsx | ext:ppt | ext:pptx | ext:pdf | ext:csv | ext:rtf
site:example.com intitle:"confidential" | intitle:"internal use"
site:example.com intext:"DO NOT DISTRIBUTE" | intext:"proprietary and confidential"
```

### Third-party leaks (no domain needed in dork, target as keyword)
```
site:pastebin.com "example.com"
site:jsfiddle.net | site:codepen.io | site:codebeautify.org "example.com"
site:groups.google.com "example.com"
site:openbugbounty.org inurl:reports intext:"example.com"
site:shodan.io "example.com"
```

### Cloud storage / buckets
```
site:s3.amazonaws.com "example.com"
site:blob.core.windows.net "example.com"
site:googleapis.com "example.com" storage
site:firebaseio.com "example.com"
site:drive.google.com "example.com"
site:docs.google.com inurl:"/d/" "example.com"
site:digitaloceanspaces.com "example.com"
site:jfrog.io "example.com"
site:r2.cloudflarestorage.com "example.com"
site:dropbox.com/s | site:box.com/s "example.com"
```

### Framework/CMS-specific quick wins
```
inurl:/wp-content/plugins/ site:example.com ext:txt | ext:log
intext:"Powered by" & intext:Drupal & inurl:user
site:*/joomla/login
inurl:/config.php.bak | inurl:/configuration.php.bak site:example.com
site:example.com inurl:node_modules | inurl:vendor/ ext:json
```

### Tech stack fingerprinting
```
site:example.com ext:package.json | ext:composer.lock | ext:pom.xml | ext:Gemfile
inurl:swagger | inurl:openapi site:example.com        # API docs exposure
site:example.com inurl:graphql | inurl:graphiql
site:example.com intitle:"actuator" | inurl:/actuator/health   # Spring Boot
```

## GitHub dorking (code-search variant, same mindset)

GitHub code search is the richest leak source; use `gh api` search or the web UI logged in:

```
"example.com" password
"example.com" api_key
"example.com" AKIA
org:targetorg filename:.env
org:targetorg filename:config.php password
"example.com" "BEGIN RSA PRIVATE KEY"
```
`gh api "search/code?q=example.com+AKIA" --jq '.items[].html_url'` — requires an authenticated token; results only include repos you can see, so leaks in private repos of your own orgs are findable.

## Result handling & severity grading

- **CRITICAL** — live credentials, private keys, session tokens, full DB dumps, cloud keys (AKIA, service-account JSON). Report immediately; do not store the secret values themselves, store the URL and a redacted excerpt.
- **HIGH** — admin panels without visible auth, internal endpoints indexed, backup files of configs, `.env`-shaped files, exposed API docs of internal services.
- **MEDIUM** — directory listings, stale login portals, sensitive-named docs (payroll, contracts), version-control metadata.
- **LOW / INFO** — subdomain discovery, stack fingerprints, third-party mentions.

Report format per finding: severity / URL / the dork that found it / one-line impact / recommended fix. Never paste live secrets into the report — reference them.

## Remediation playbook (when auditing own properties)

- Files exposed: remove + add `Disallow:` in robots.txt does NOT undo indexing — request removal via Google Search Console, then fix permissions.
- Set `X-Robots-Tag: noindex` on auth pages, dashboards, and anything session-gated.
- For buckets/docs platforms: audit public-sharing settings; index-exposure usually means public ACL.
- Re-search after fixes with the same dorks to confirm de-indexing (takes days to weeks).

## Pitfalls

- `ext:` and `filetype:` behave slightly differently; `ext:` is more reliable for extensions.
- Google ignores some operators when combined with `OR` chains — split complex ORs into separate queries.
- `site:` matches subdomains automatically; `site:example.com` includes `sub.example.com`. Use `site:www.example.com` to pin exact host.
- Cached results can be months stale — always fetch the live URL before declaring a leak current.
- Quotes inside dorks need careful escaping when passed through shells — use `web_search`/`web_extract` args, not raw shell strings.
