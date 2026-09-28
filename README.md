# context7-json-config (Agent Skill)

An Agent Skill that teaches coding agents how to write, review, tune, and validate
`context7.json` — the repo-root config file that tells [Context7](https://context7.com) how to
index a library's documentation for AI coding agents.

## What's in this repo

```
context7-json-config/
├── SKILL.md         # Entry point: frontmatter + condensed workflow (loaded by the agent)
├── reference.md      # Field reference, full examples, troubleshooting (loaded on demand)
└── LICENSE           # MIT
context7-json-config-skill.md  # Original source spec the skill was built from
AGENTS.md            # Instructions for coding agents working in this repo
```

`SKILL.md` is deliberately short — it covers the core workflow and defers detail (exclusion
pattern tables, worked examples for web frameworks/SDKs/monorepos, troubleshooting recipes) to
`reference.md`, which the agent reads only when it needs it.

## Installing the skill

The skill lives entirely in the `context7-json-config/` folder. How you "install" it depends on
which agent/tool you use — pick your section below.

### Claude Code

Project-scoped (only active in this repo):

```bash
mkdir -p .claude/skills
cp -r context7-json-config /path/to/your-project/.claude/skills/
```

Personal/global (active in every project):

```bash
mkdir -p ~/.claude/skills
cp -r context7-json-config ~/.claude/skills/
```

Claude Code auto-discovers skills by scanning their `description` frontmatter, so no further
registration step is needed. Verify with `/skills` or by asking about a `context7.json` file.

### Claude Desktop

Claude Desktop (and claude.ai) supports uploading Skills as a zip under
**Settings → Capabilities → Skills → Upload skill**.

```bash
cd context7-json-config
zip -r ../context7-json-config.zip .
```

Upload the resulting `context7-json-config.zip` in the Skills settings panel.

### Cursor

Cursor doesn't have a native "Agent Skills" format, but it does read project rules. Turn the
skill into a Cursor rule:

```bash
mkdir -p .cursor/rules
cp context7-json-config/SKILL.md .cursor/rules/context7-json-config.mdc
```

Add YAML frontmatter Cursor expects (`description`, `alwaysApply: false`) if it isn't already
recognized, so the rule is fetched only when relevant. Keep `reference.md` in the repo and
mention its path in the rule body so the agent can open it on demand (e.g. via `@reference.md`).

### Windsurf

Same idea as Cursor — drop it into Windsurf's rules directory:

```bash
mkdir -p .windsurf/rules
cp context7-json-config/SKILL.md .windsurf/rules/context7-json-config.md
```

Reference `context7-json-config/reference.md` from within the rule for the detailed tables.

### OpenAI Codex (CLI / cloud)

Codex automatically reads `AGENTS.md` at the repo root. Either:

- Copy the skill folder into the target repo and add a pointer line to that repo's `AGENTS.md`:
  `See context7-json-config/SKILL.md for how to write context7.json.`
- Or merge the relevant section of `SKILL.md` directly into `AGENTS.md` if you want it always
  in context rather than referenced on demand.

### Devin

Add the skill as a knowledge source: in the Devin dashboard, add `context7-json-config/SKILL.md`
(and `reference.md`) as a **Knowledge** entry, or copy the folder into the target repo and
reference it from that repo's `AGENTS.md` — Devin reads `AGENTS.md` as repo-level guidance.

### Any other agent

Most coding agents support *some* form of repo-level instruction file (`AGENTS.md`,
`.cursorrules`, `.windsurfrules`, custom system prompts, etc.). The safe fallback for any of them
is: copy the `context7-json-config/` folder into the target repo, then point the agent's
instruction file at `context7-json-config/SKILL.md`.

## License

MIT — see [`context7-json-config/LICENSE`](context7-json-config/LICENSE).
