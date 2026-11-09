# Content Search Guide

Deep-dive on finding movies, series, books, and files across open directories, storage sites, and drive shares. Complements the content-search section in SKILL.md.

## Open directories (OD)

Open directories are web-exposed folder listings (Apache/Nginx autoindex). They index poorly but the ones that do index are gold.

Anatomy of a good OD dork:

```
intitle:"index of" "movie name" (mp4 | mkv)
```

- `intitle:"index of"` matches the autoindex page title
- Quoted content name narrows to relevant folders
- Extension group filters the files

Refinements:
- Add `parent directory` for broader listing hits
- Add year (`"movie name 2024"`) to cut remakes
- `last modified` sorting in results is not queryable; fetch and parse the listing page instead

## Verification checklist for media hits

A link is not a find. Verify before reporting:

1. **Extension**: real films are .mkv, .mp4, .avi; .exe or .html "movies" are malware or redirect farms
2. **Size**: a feature film is 300 MB (low quality) to 25 GB (remux). A 2 MB .mkv is a stub or malware. Series episodes are 100 MB to 8 GB
3. **Directory context**: files sitting in a folder with other real releases look right; a lone file in a spammy folder does not
4. **Live check**: fetch the URL (HEAD request or `web_extract`); redirect chains to ad gates mean the OD is a trap

## Google Drive dorking

Drive files get indexed when shared "anyone with the link" and crawled:

```
site:drive.google.com "movie name"
"movie name" site:docs.google.com inurl:"/document/d"     # written lists/plans
```

Drive hits decay fast (links get revoked). Verify the link resolves before reporting. For dead links, search the same title on archive.org Wayback to find the original source page that listed it.

## S3 and cloud storage

```
site:s3.amazonaws.com "movie name"
site:s3.amazonaws.com intitle:index.of "movie"
site:blob.core.windows.net "movie name"
site:digitaloceanspaces.com "movie name"
site:storage.googleapis.com "movie name"
site:r2.cloudflarestorage.com "movie"
```

R2 buckets rarely index (Cloudflare sets noindex behavior by default); S3 remains the most common exposure. Bucket-listing pages found through search usually mean the bucket allows public LIST, which is a misconfiguration worth reporting if it is someone's own bucket.

## Mirror-site discovery

For finding which streaming mirrors host a title:

```
"movie name" inurl:watch -inurl:trailer -site:youtube.com
"watch movie name online" -site:justwatch.com -site:imdb.com
"movie name" site:fmovies.co | site:soap2day.day | site:123movies.day
```

Mirror sites churn domains constantly. When a domain dork dies, search the title plus generic watch-phrases instead of maintaining domain lists.

## Books and documents

```
"book title" ext:pdf -site:amazon.com
"book title" (site:archive.org OR site:openlibrary.org)      # legal first
intitle:"index of" "book title" pdf
"book title" site:annas-archive.org OR site:libgen.is        # shadow libraries; legality varies by country
"book title" site:scribd.com OR site:academia.edu
```

For papers, also try `site:arxiv.org` and `site:semanticscholar.org` before shadow libraries; most papers are legally free somewhere.

## General file finder

When hunting a specific known file:

```
intitle:"index of" "filename"
"exact file name" ext:ext1 | ext:ext2
"file name" (site:mediafire.com OR site:mega.nz OR site:anonfiles.com)
```

Hoster-scoped dorks decay as hosters die; prefer the `intitle:"index of"` pattern and OD aggregation sites when possible.

## Legal note

Finding publicly exposed content is searching; downloading pirated media is not legal in most jurisdictions. This skill reports where content exists. The user decides what to do with that knowledge within their own laws.
