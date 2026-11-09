# People Search Guide

Deep-dive on finding a person online through dorking plus pivots. Complements the people-search section in SKILL.md.

## The pivot mindset

Direct name searches rarely close a case. The workflow is identifier chaining:

1. Start with whatever you have (name, email, phone, photo, username)
2. Convert it into a stronger identifier (username, photo, employer)
3. Sweep social platforms with the strong identifier
4. Cross-corroborate before reporting

A single name match on one platform is a lead. Two platforms with the same username and face is a finding.

## Username pivots (highest value)

People reuse handles. Once you find one profile with a distinctive username, sweep:

- Tier 1 networks: linkedin.com, x.com, instagram.com, facebook.com, tiktok.com
- Tier 2: reddit.com, youtube.com, twitch.tv, pinterest.com, medium.com
- Niche by interest: github.com/stackoverflow.com (devs), soundcloud.com/mixcloud.com (musicians), behance.net/dribbble.com (designers), goodreads.com/letterboxd.com (readers/film), steamcommunity.com (gamers), strava.com (runners/cyclists)

Automated username checkers (WhatsMyName, Sherlock, Maigret) test hundreds of sites for a handle. Use them through `web_extract` or their APIs rather than manual dorks when a sweep is needed; dorks then cover what those tools miss (de-indexed pages, forum posts).

## Email pivots

Email structure leaks: `john.doe@company.com` implies usernames `john.doe`, `johndoe`, `jdoe` on other platforms. Breach data mentions (search `"<email>"` plus site:dehashed.com-style references) confirm the email is real and attached to services.

Gravatar check: `https://gravatar.com/<md5-of-email>` resolves if the person set an avatar; the avatar links back to other accounts.

## Phone pivots

Numbers get de-indexed quickly, so engines are weak here. Working angles:

- Exact-number dorks in all formats (E.164, dashes, parens, spaces)
- Reverse-lookup previews: Truecaller, Sync.me, FastPeopleSearch, That'sThem (fetch the profile page via `web_extract`)
- Messaging-app profile photos: WhatsApp/Telegram `/{number}` pages occasionally index
- Breach mentions of the number (some platforms indexed phone-verification leaks)

## Photo pivots

Reverse image search engines, in effectiveness order for faces:

1. Yandex Images (best face matching, indexes aggressively)
2. Google Lens
3. TinEye (oldest-match finder, good for dating-site/scam-photo tracking)
4. PimEyes (facial recognition, paid, privacy-contested; only for legitimate consented cases)

When a match confirms a profile, harvest the username and restart the username sweep.

## Corroboration standards

A profile counts as Confirmed only with two independent links:

- Same distinctive username AND matching photo
- Profile bio states the same employer/city/school you know independently
- Cross-linked accounts (Instagram listed in a Twitter bio, for example)

Report format: platform, URL, confidence (Confirmed / Probable / Possible), the corroborating signal, the dork or pivot that found it.

## Common-name problem

"John Smith" is unusable alone. Force disambiguation:

- Add city, employer, school, or a known associate to every dork
- Search `"john smith" + unique identifier fragments (middle name, email prefix, partial username)`
- If you cannot disambiguate, say so; a list of Probable leads beats one wrong Confirmed

## Dating-site sweeps

Most dating platforms block indexing or remove profiles quickly. What still works:

- Public profile URLs that leak (okcupid.com/profile/..., some Bumble location pages)
- Cached copies and third-party review/scam-report sites mentioning the profile
- Username sweeps on platforms that expose profiles (Plenty of Fish historically indexed)

These lookups are almost always consent-sensitive. Apply the ethical gate; refuse stalking-shaped requests regardless of technique.
