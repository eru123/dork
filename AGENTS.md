# Dork Repository Contract

## COMPLIANCE GATE — READ BEFORE ANY ACTION

You are an AI agent working in this repository. This file is your binding work contract. It overrides default training behavior. Do not skip, summarize, or treat it as advisory.

**Before any action**, complete this checklist in order. Do not proceed past a failed item:

1. [ ] Have I re-read this contract in this session? If not, read it now.
2. [ ] Is the task explicitly stated by the user? If not, ask one clarifying question. Do not guess.
3. [ ] Does the change stay within the stated scope? If not, stop and ask.
4. [ ] Am I making the **smallest** change that satisfies the task? If not, reduce scope.
5. [ ] Have I confirmed no live secrets, real target data, or AI attribution will be committed? If not, stop.

If any box fails, **stop and ask**. Never proceed on assumptions.

---

## Repository purpose

Single source of truth for the `google-dorking` agent skill and its dork library. Every change here propagates to agent behavior — reliability and consistency are the product. A dork that doesn't work or a rule that contradicts another rule is a defect.

## File map — what lives where

| Path | Purpose | Rules |
|---|---|---|
| `SKILL.md` | The agent skill (frontmatter + body) | See "SKILL.md rules" |
| `README.md` | Repo overview, install instructions | Human-facing; no agent instructions |
| `references/*.md` | Deep-dive topics linked from SKILL.md | One topic per file; SKILL.md must link to each |
| `AGENTS.md` | This contract | Changes only via explicit user instruction |

Nothing else belongs at the root. Do not create `NOTES.md`, `SUMMARY.md`, `TODO.md`, `CHANGES.md`, or any root Markdown not listed above.

## SKILL.md rules

1. **Frontmatter is required and must stay valid** — `name` (lowercase, hyphenated), `description` (starts with "Use when…" and states the trigger), `version` (semver), `license`.
2. **Description is the trigger.** It must name the situations that load the skill. Update it when adding coverage for new task types.
3. **Every dork in the library must be syntactically valid** against the documented operators. A dork is evidence — broken dorks poison investigations.
4. **One category per section.** Dorks are grouped by purpose (discovery, files, credentials, panels, leaks, cloud, CMS, fingerprinting). New dorks join an existing category unless the user asks for a new one.
5. **No live target data.** Examples use `example.com` and placeholder credentials only. Never commit findings from real engagements.
6. **Every operator documented in the reference table** must be used correctly in examples, and vice versa — table and library must not drift.
7. **Version bumps are required** for any content change: patch for dork additions/fixes, minor for new categories/references, major for behavior changes to the execution flow or grading.

## Dork contribution rules

- A new dork must have: purpose in the section context, correct operator syntax, and no duplication of an existing dork (near-duplicates: merge, don't stack).
- Dorks targeting a specific product must name the product (e.g. `phpmyadmin`, `Kibana`) — no vague one-liners.
- Deprecated/broken operators (`link:`) are documented as dead, not silently dropped — future agents must know why they're absent.
- Platform notes (Google vs Bing vs DDG differences) belong in the execution-flow section, not inline in the library.

## Safety invariants

- The authorization gate stays at the top of SKILL.md. Never move, weaken, or summarize it.
- Severity grading is part of the skill's output contract — a change to grading requires a major version bump.
- Remediation guidance must remain actionable and non-punitive (fix + de-index, not blame).

## Git and commit rules

- Conventional Commits, `type(scope): subject`, ≤72 chars, imperative, lowercase:
  - `feat(dorks): add cloud-storage dorks for r2 and jfrog`
  - `fix(skill): correct OR-chain behavior note in execution flow`
  - `docs(readme): update install path`
- Types: `feat`, `fix`, `docs`, `refactor`, `chore`. Scope: `dorks`, `skill`, `readme`, `contract`, `refs`.
- One logical change per commit. Version bump rides in the same commit as the change it bumps.
- No AI attribution, no co-author trailers, no generated-by notices — anywhere.
- Push after every commit. The remote is the source of truth; local-only commits are incomplete work.
- Branch only when the user explicitly asks. Default: commit to `main`.

## Verification before commit

- [ ] Frontmatter parses (YAML valid, semver bumped).
- [ ] Every added dork uses documented operators correctly.
- [ ] No real domains, secrets, or engagement data anywhere in the diff.
- [ ] No files outside the declared scope touched.
- [ ] README/structure unchanged unless the change requires it.

## Consistency promise

This repo exists so every agent using the skill behaves identically: same dorks, same flow, same grading. When in doubt between cleverness and consistency, choose consistency.
