---
name: context7-json-config
description: Guide for writing, reviewing, or validating a context7.json file at the root of a GitHub repository, which tells Context7 how to index a library's docs for AI coding agents. Use when the user asks to create, review, tune, or debug a context7.json config, wants to control what Context7 indexes for their repo, or asks about Context7 folders/excludeFolders/excludeFiles/rules/previousVersions/branchVersions fields.
version: 1.0.0
author: fsans at ntwk.es
license: MIT
date: 2026-09-28
---

# Context7 Configuration (context7.json)

`context7.json` is a repo-root config (like `robots.txt`) that controls **where** Context7 looks
for documentation in a GitHub repo — not what file types it reads. Context7 already indexes
markdown/doc sites automatically and falls back to source code if needed.

## Workflow

1. **Analyze the repo first.** Before writing anything, check: where docs live (`/docs`,
   `/guides`, root README), whether it's a monorepo with multiple packages, whether there are
   archived/deprecated/versioned docs, and whether non-English `i18n` folders exist.

2. **Start minimal.** Default to just `$schema`, `projectTitle`, `description`. Only add more
   fields if the repo's defaults (full-repo scan, standard exclusions) don't fit.

   ```json
   {
     "$schema": "https://context7.com/schema/context7.json",
     "projectTitle": "Your Project Name",
     "description": "Brief description of what your project does"
   }
   ```

3. **Add `folders` only to restrict scanning** to specific directories (large repo, monorepo,
   docs confined to a few paths). If omitted/empty, Context7 scans everything (root markdown
   files are always included regardless).

4. **Add `excludeFolders` / `excludeFiles` to filter out noise** — `node_modules`, `.git`,
   `build`/`dist`, test dirs, archived docs, `CHANGELOG.md`, `LICENSE.md`, etc. Note: Context7
   already excludes common changelog/license files and non-English `i18n` folders by default —
   only list these explicitly if overriding or being extra explicit. `excludeFolders` always wins
   over `folders` — never write contradictory rules.

5. **Add `rules`** (optional but recommended): short, actionable, library-specific dos/don'ts
   that steer agents using the library (e.g. "Use v2 API only; v1 is unsupported"). Not generic
   best practices.

6. **Add version fields only if actively needed**: `previousVersions` (git tags) or
   `branchVersions` (branches) — only for versions still in active use. Use `branch` if docs live
   on a non-default branch.

7. **Validate before finalizing**: valid JSON, `$schema` present, no `folders`/`excludeFolders`
   contradictions, rules are concrete and actionable, only actively-maintained versions listed.

For exclusion pattern tables, full worked examples (web framework, SDK, monorepo), and
troubleshooting recipes (too much noise / missing docs / outdated docs surfacing / agents citing
old APIs / non-English interference), see `reference.md`.

## Output

Produce the final `context7.json` directly (valid JSON, no comments). If context about the repo
is missing, ask targeted questions rather than guessing (e.g. "Where do your docs live — a
`/docs` folder, or scattered markdown across the repo?").
