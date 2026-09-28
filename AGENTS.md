# AGENTS.md

Instructions for coding agents (Claude Code, Codex, Devin, Cursor, Windsurf, etc.) working in
this repository.

## What this repo is

A single Agent Skill, `context7-json-config/`, that teaches an agent how to author and tune
`context7.json` files (the config Context7 uses to decide what part of a GitHub repo to index
for documentation). The repo is the skill's source of truth — there is no application code here.

## Structure

- `context7-json-config/SKILL.md` — the skill entry point: YAML frontmatter (`name`,
  `description`, `version`, `author`, `license`, `date`) plus a condensed workflow. This is what
  gets loaded into an agent's context when the skill triggers.
- `context7-json-config/reference.md` — detailed field tables, full worked examples, and
  troubleshooting recipes. Loaded on demand, not automatically — keep it out of `SKILL.md` to
  keep the entry point cheap to load.
- `context7-json-config/LICENSE` — MIT.
- `context7-json-config-skill.md` — the original spec document this skill was distilled from.
  Treat it as historical/reference input, not a file to keep in sync going forward; the skill
  bundle is the canonical artifact.

## Working in this repo

- If asked to update the skill's behavior, edit `context7-json-config/SKILL.md` and
  `reference.md` directly — don't touch `context7-json-config-skill.md`.
- Keep `SKILL.md` short: core decision workflow only. Anything that reads like a lookup table,
  a long list of examples, or troubleshooting detail belongs in `reference.md`.
- The `description` field in `SKILL.md` frontmatter is what triggers the skill — if you change
  what the skill covers, update `description` so it still matches the right prompts.
- Bump `version` in the frontmatter when you change the skill's behavior or scope; update `date`
  to the day of the change.
- This skill emits JSON (`context7.json` files) as its output artifact — when validating changes
  to the skill's guidance, sanity-check that any example JSON in `SKILL.md`/`reference.md` is
  syntactically valid.

## Installing/testing the skill locally

See `README.md` for per-agent install instructions (Claude Code, Claude Desktop, Cursor,
Windsurf, Codex, Devin). For quick local testing with Claude Code:

```bash
mkdir -p ~/.claude/skills
cp -r context7-json-config ~/.claude/skills/
```

Then ask an agent to "write a context7.json for this repo" and confirm the skill triggers and
produces valid JSON following the workflow in `SKILL.md`.

## Conventions

- No code, no build/test tooling in this repo — everything is Markdown/JSON documentation.
- Don't add a build system, CI, or package.json unless explicitly asked; this is a plain
  skill-distribution repo.
