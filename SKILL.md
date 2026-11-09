---
name: google-dorking
description: "Use when the agent needs to find information through search-engine dorking: locating a person's social media profiles and public accounts by name, username, phone, or email; finding movies, documents, books, or files on open storage (S3, Google Drive, index-of directories); exposed-data discovery; and OSINT recon. Covers operator reference, curated dork library for people search / file search / exposure audits, safe execution flow, and result handling."
version: 2.0.0
author: eru123
license: MIT
metadata:
  hermes:
    tags: [osint, search, dorking, people-search, file-search]
---

# Google Dorking

Search-engine dorking finds what normal searches miss: a person's scattered accounts, files sitting in open buckets and index-of directories, leaked documents, hidden endpoints, and content mirrored across sites. This skill is the agent's operating manual for building and running dorks and turning results into answers.

## When to use

- **People search**: find every social profile, forum account, and public footprint for a person, starting from a name, username, email, phone number, or photo
- **Content search**: locate a movie, series, book, document, or file across open directories, cloud storage, and drive-sharing sites
- **Exposure audits**: check whether your own domains leak files, credentials, endpoints, or panels
- **General OSINT**: org research, domain recon, document trails

## Legal and ethical gate

Before searching for a specific person, confirm the request is legitimate: your own footprint, a consented background check, a fraud investigation, a missing-person case, or similar. Do not help build profiles for stalking, harassment, doxxing, or impersonation. Dorking is just searching, but the use case matters; refuse requests that only make sense as targeting. For exposure audits on domains you do not own, an authorized scope is still required.

## Operator reference (Google)

| Operator | Meaning | Example |
|---|---|---|
| `site:` | restrict to domain/site | `site:example.com` |
| `inurl:` | term in URL | `inurl:profile` |
| `intitle:` | term in page title | `intitle:"index of"` |
| `intext:` / `allintext:` | term in body | `intext:"joined"` |
| `ext:` / `filetype:` | file extension/type | `ext:pdf`, `filetype:xlsx` |
| `cache:` | cached copy (flaky) | `cache:example.com/page` |
| `before:` / `after:` | date bounds | `after:2024-01-01` |
| `-term` | exclude | `-site:pinterest.com` |
| `"exact"` | exact phrase | `"john smith"` |
| `OR` / `|` | either | `site:linkedin.com OR site:x.com` |
| `*` | wildcard in phrase | `"* johnson" chicago` |
| `related:` | similar sites | `related:imdb.com` |

Combination rules: operators chain left to right, `OR` needs caps, grouping with `( )` is unreliable; prefer separate queries. Google suppresses obvious dorking with an apology page; rotate query structure instead of retrying the same one.

## Search execution flow

1. **Construct** dorks from the library below, substituting the person's name, username, email, phone, or the content title.
2. **Fetch results.** Prefer `web_search` with the dork as the query (operators pass through to the backend). For engine-specific behavior use `https://www.google.com/search?q=<url-encoded dork>&num=100` via `web_extract`. Fallbacks: Bing (`https://www.bing.com/search?q=`), DuckDuckGo lite (`https://html.duckduckgo.com/html/?q=`), self-hosted SearXNG with `format=json` for volume work (see references/search-engines.md).
3. **Dedupe** result URLs and classify: confirmed profiles / probable profiles / files / noise.
4. **Verify people hits.** A name alone proves nothing. Confirm a profile belongs to the person through at least two corroborating signals: same username across sites, same photo (reverse image search), matching location or employer in the bio, linked accounts. Report confidence levels, not just matches.
5. **Verify file hits.** For movies and books, check the file size, extension, and directory context; a 2 MB "movie.mkv" is a fake. Prefer direct-download hits over link farms.
6. **Report** with the exact dork that produced each finding; dorks are reproducible evidence.

Rate-limit reality: aggressive sequential scraping of Google gets captcha-blocked quickly. Batch 3 to 5 dorks per pass, space them out, fall back to other engines. Never hammer.

## People search library

The core workflow for finding a person online. Start broad, then pivot on identifiers.

### Pivot table: which identifier to use when

| You have | Best dorks |
|---|---|
| Full name | Name + city/employer dorks, then username pivots from found profiles |
| Username/handle | Username dorks (highest hit rate), then email pattern guesses |
| Email | Email dorks, breach-mention dorks, then username from the email prefix |
| Phone | Phone-format dorks, then reverse-lookup sites, then name pivot |
| Photo | Reverse image search first (see below), then name/username pivots |

### By username (strongest pivot; people reuse handles)
```
"johndoe99" site:x.com OR site:instagram.com OR site:reddit.com
"johndoe99" site:tiktok.com OR site:twitch.tv OR site:youtube.com
"johndoe99" site:github.com OR site:stackoverflow.com OR site:medium.com
"johndoe99" (site:facebook.com OR site:linkedin.com)
intext:"johndoe99" -site:pinterest.com
```

### By full name
```
"john smith" site:linkedin.com
"john smith" site:facebook.com -site:linkedin.com
"john smith" site:twitter.com OR site:x.com
"john smith" chicago site:instagram.com                 # name + known city
"john smith" site:about.me OR site:crunchbase.com OR site:gravatar.com
"john smith" site:researchgate.net OR site:scholar.google.com   # academics
"john smith" site:gamer.com OR site:steamcommunity.com  # gamers; adapt per interest
```

### By email
```
"john.doe@gmail.com"                        # exact match across indexed pages
"john.doe@gmail.com" site:facebook.com OR site:linkedin.com
"johndoe@gmail.com" site:skype.com OR site:slack.com
intext:"johndoe@gmail.com" ext:pdf | ext:xlsx | ext:csv   # spills in docs
"john.doe@gmail.com" site:haveibeenpwned.com OR site:dehashed.com
```

### By phone number
```
"+14155551234"                            # exact E.164 format
"415-555-1234" OR "(415) 555-1234" OR "415 555 1234"   # US formats
"14155551234" site:facebook.com OR site:instagram.com
"+14155551234" site:truecaller.com OR site:sync.me
"4155551234" site:whatsapp.com OR site:telegram.org
```
Phone dorking has low hit rates on Google (numbers get de-indexed fast). Supplement with reverse-lookup services (Truecaller, Sync.me, FastPeopleSearch, That'sThem) via `web_extract` directly.

### Social site sweep (run all when nothing is known beyond a name)
```
"john smith" (site:linkedin.com OR site:x.com OR site:instagram.com OR site:facebook.com OR site:tiktok.com OR site:reddit.com OR site:youtube.com OR site:github.com OR site:medium.com OR site: pinterest.com)
"john smith" site:myfitnesspal.com OR site:goodreads.com OR site:letterboxd.com
"john smith" site:spotify.com OR site:soundcloud.com OR site:mixcloud.com
"john smith" site:behance.net OR site:dribbble.com OR site:deviantart.com
```

### Dating sites
```
"john smith" site:tinder.com OR site:bumble.com OR site:hinge.co
"john smith" site:okcupid.com OR site:pof.com OR site:match.com
"johndoe99" site:grindr.com OR site:her.co
site:okcupid.com/profile "john"
```
Note: dating profiles are mostly de-indexed or behind logins. Hit rates are low; a hit is high-signal. Respect the ethical gate above; dating-site lookups are almost always consent-sensitive.

### Forum and community footprints
```
"johndoe99" site:reddit.com inurl:user
"johndoe99" site:4chan.org OR site:8kun.top          # rarely indexed now
"john smith" forum | board | community inurl:member
"johndoe99" site:quora.com OR site:stackoverflow.com OR site:medium.com
```

### Reverse image search (when you have a photo)
Google Images is not dork-friendly for faces; use dedicated engines via `web_extract`:
- Google Lens: `https://lens.google.com/uploadbyurl?url=<image-url>`
- TinEye: `https://tineye.com/search?url=<image-url>`
- Yandex Images (best for faces): `https://yandex.com/images/search?rpt=imageview&url=<image-url>`
- PimEyes is the strongest facial-recognition option but is a paid, privacy-contested service; mention it only for legitimate cases.

Pivot any found profile photo back into username searches.

### Whitepages / public records (US-centric)
```
"john smith" "chicago" site:whitepages.com OR site:fastpeoplesearch.com OR site:thatsthem.com
"john smith" site:radaris.com OR site:spokeo.com OR site:beenverified.com
"john smith" site:publicrecords.com intitle:"profile"
```
These aggregator pages are often paywalled; extract the free preview data via `web_extract` and pivot.

## Content search library (movies, series, books, files)

For finding content across open directories, storage sites, and drive shares.

### Movies & series on open directories
```
intitle:"index of" "movie name" mkv
intitle:"index of" "movie name" (mp4 | mkv | avi)
intitle:"index of" /movies "movie name 2024"
intitle:"index of" "series name" s01
"movie name" inurl:download -site:justwatch.com -site:imdb.com
```
Verify hits: check extension (mkv/mp4), realistic size (a film is 300 MB to 20 GB), and that the link is a directory listing or direct file, not a redirect farm.

### Google Drive
```
site:drive.google.com "movie name"
site:drive.google.com "movie name 2024" inurl:"/file/d/"
site:drive.google.com "series name" intitle:"matching docs"   # drive shares get indexed
"movie name" (site:drive.google.com OR site:docs.google.com)
```
Shared Drive files are indexed only while the share link is public. Dead links are common; verify.

### S3 and open cloud storage
```
site:s3.amazonaws.com "movie name"
site:s3.amazonaws.com intitle:index.of "movie"
site:blob.core.windows.net "movie name"
site:digitaloceanspaces.com "movie name"
site:r2.cloudflarestorage.com "movie"
site:storage.googleapis.com "movie name"
```

### Streaming-mirror sites (find which mirror hosts it)
```
"movie name" (site:fmovies.co OR site:soap2day.day OR site:123movies.day)
"watch movie name online free" -site:justwatch.com -site:imdb.com -site:letterboxd.com
"movie name" inurl:watch -inurl:trailer -site:youtube.com
```

### Books & documents
```
"book title" ext:pdf -site:amazon.com
"book title" (site:archive.org OR site:openlibrary.org)
intitle:"index of" "book title" pdf
"book title" site:annas-archive.org OR site:libgen.is
"book title" site:scribd.com OR site:academia.edu
```

### General file finders
```
intitle:"index of" "filename or phrase"
"exact file name" ext:mkv | ext:mp4 | ext:pdf | ext:epub | ext:zip
site:mediafire.com "file name" | site:mega.nz "file name"
"file name" (site:4shared.com OR site:sendspace.com OR site:anonfiles.com)
```

## Exposure audit library (own infrastructure)

For checking your own domains for leaks. Same core as v1, kept for audits.

```
site:example.com ext:log | ext:txt | ext:conf | ext:cnf | ext:ini | ext:env
site:example.com ext:bak | ext:backup | ext:old | ext:swp | ext:git
site:example.com intitle:"index of" "parent directory"
site:example.com intext:"api_key" | intext:"AKIA" | intext:"BEGIN RSA PRIVATE KEY"
site:example.com inurl:admin | inurl:phpmyadmin | inurl:server-status
site:s3.amazonaws.com "example.com" | site:blob.core.windows.net "example.com"
site:drive.google.com "example.com" | site:firebaseio.com "example.com"
```

## GitHub code search (identifier pivot)

For people-search and leak-hunting on GitHub, use authenticated search:

```
"johndoe99"                                 # username across code and gists
"john.doe@gmail.com"
"example.com" password | api_key | AKIA
org:targetorg filename:.env
```
`gh api "search/code?q=johndoe99" --jq '.items[].html_url'`; requires an authenticated token, results only include repos you can see.

## Result handling and confidence grading

### People-search confidence levels
- **Confirmed**: two or more corroborating signals tie the profile to the person (username + photo, or bio details matching known facts)
- **Probable**: unique username match, or name + city + demographic alignment; single-source
- **Possible**: name-only matches; report as leads, not findings
State the confidence level for every reported profile. A wrong attribution is worse than a miss.

### Exposure-audit severity (own properties)
- **CRITICAL**: live credentials, private keys, session tokens, DB dumps
- **HIGH**: open admin panels, internal endpoints indexed, config backups
- **MEDIUM**: directory listings, stale portals, sensitive-named docs
- **LOW**: subdomain discovery, stack fingerprints

Report format per finding: severity or confidence / URL / the dork that found it / one-line impact / recommended fix or next pivot. Never paste live secrets into reports; reference them.

## Remediation playbook (exposure audits)

- Removing a file does not undo indexing; request removal via Google Search Console, then fix permissions
- Set `X-Robots-Tag: noindex` on auth pages, dashboards, session-gated content
- For buckets and doc platforms: audit public-sharing settings; index exposure usually means public ACL
- Re-run the same dorks after fixes to confirm de-indexing (takes days to weeks)

## Pitfalls

- `ext:` is more reliable than `filetype:` for raw extensions
- Google silently drops operators inside complex `OR` chains; split those into separate queries
- `site:example.com` includes all subdomains; use `site:www.example.com` to pin the exact host
- Cached results can be months stale; fetch the live URL before declaring anything current
- Common names produce garbage: always pivot on username, email, city, or employer before reporting
- Movies found in open directories are oftencam rips with wrong labels; verify size and container
- People-search results are probabilistic; the confidence levels above are not optional
