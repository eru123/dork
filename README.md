# Dork: Google Dorking Skill & Library

A curated Google-dorking skill for AI agents: people search (find a person's social profiles from a name, username, email, or phone), content search (movies, books, files across open directories, S3, Google Drive), and exposure audits for your own infrastructure.

## Structure

```
SKILL.md          The agent skill: operator reference, execution flow, dork libraries, grading
AGENTS.md         Repo contract (binding, modeled on RepoPact)
references/       Topic deep-dives the skill links to
```

## Using the skill

The SKILL.md frontmatter is Hermes-compatible. Install it:

```bash
cp SKILL.md ~/.hermes/skills/google-dorking/SKILL.md
```

Or point `skills.external_dirs` at a checkout of this repo. The agent loads it whenever a task involves dorking, OSINT people search, content discovery, or exposure audits.

## Sources

Dorks curated from the GHDB lineage and public collections, including
[aw-junaid/Hacking-Tools/Google Dorks](https://github.com/aw-junaid/Hacking-Tools/tree/master/Google%20Dorks)
(which derives from [taksec's google-dorks-bug-bounty](https://github.com/taksec/google-dorks-bug-bounty)),
[Proviesec/google-dorks](https://github.com/Proviesec/google-dorks), and
[lutfumertceylan/top25-parameter](https://github.com/lutfumertceylan/top25-parameter).

## License

MIT
