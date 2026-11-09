# Secret Pattern Reference

Patterns worth hunting for in indexed content, with context on what each represents. Used with `intext:` dorks in SKILL.md. When triaging hits, match these patterns exactly — partial matches are noise.

## Cloud provider keys

| Pattern | Provider | Meaning |
|---|---|---|
| `AKIA[0-9A-Z]{16}` | AWS | Long-lived access key ID — CRITICAL if paired with secret |
| `wJalr` style 40-char base64 | AWS secret key | Never travels alone; find the matching ID |
| `ASIA[0-9A-Z]{16}` | AWS | Temporary STS credential — expires, still urgent |
| `AIza[0-9A-Za-z_-]{35}` | Google API key | GCP/Maps/YouTube; scope varies |
| `ya29\.` | Google OAuth token | Short-lived access token |
| `ghp_[0-9A-Za-z]{36}` / `github_pat_` | GitHub | PAT — repo access scope unknown |
| `sk_live_[0-9a-zA-Z]{24,}` | Stripe | Live mode secret — CRITICAL |
| `sk-[A-Za-z0-9]{20,}` | OpenAI-style API key | High abuse value |
| `xox[baprs]-` | Slack tokens | Bot/user tokens |
| `[0-9a-f]{32}:mfa` shape | misc | Often 2FA recovery material |

## Connection strings

| Pattern | Meaning |
|---|---|
| `jdbc:mysql://user:pass@host` | Full DB credz inline |
| `mongodb+srv://user:pass@` | Atlas cluster creds |
| `postgres://user:pass@` / `postgresql://` | Postgres creds |
| `redis://:password@` | Redis with auth |
| `amqp://user:pass@` | RabbitMQ |
| `smtp://` / `SMTP_PASSWORD` | Mail relay creds |
| `ES_HOST` + `ELASTIC_PASSWORD` | Elasticsearch |

## Key material

| Pattern | Meaning |
|---|---|
| `BEGIN RSA PRIVATE KEY` | Unencrypted RSA key — CRITICAL |
| `BEGIN OPENSSH PRIVATE KEY` | OpenSSH key |
| `BEGIN PRIVATE KEY` | PKCS#8 generic |
| `BEGIN CERTIFICATE` | Cert (not secret alone, but useful context) |
| `id_rsa` as filename hit | Usually means the private key file itself |

## Service-specific tells

| Pattern | Service |
|---|---|
| `sk_live` / `pk_live` / `rk_live` | Stripe (live vs test prefix) |
| `SG.` prefix, 18+ chars | SendGrid API key |
| `hooks.slack.com/services/` | Slack webhook URL |
| `discordapp.com/api/webhooks/` | Discord webhook |
| `npm_[A-Za-z0-9]{36}` | npm token |
| `glpat-` | GitLab personal access token |
| `dop_v1_` | DigitalOcean token |
| `r8_` | Replicate token |

## Triage rules

1. **Validate shape, then liveness.** A pattern match alone is a finding; a confirmed live key is an incident.
2. **Never reproduce full secrets** in reports — first 4 chars + length, plus the URL where found.
3. **Check context.** `sk_test` keys are lower severity than `sk_live`; keys inside committed test fixtures may be intentional.
4. **AWS keys:** check against the account's published key policy if known; if not, treat as live.
5. Report immediately (CRITICAL findings don't wait for the full recon pass to finish).
