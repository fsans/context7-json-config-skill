# context7.json Reference

Detailed patterns, full example configs, and troubleshooting — load this when the minimal
workflow in SKILL.md isn't enough.

## Field Reference

### `folders`
Restricts scanning to specific directories. Empty/omitted = scan entire repo (respecting
`excludeFolders`). Root-level markdown files (README.md, etc.) are always included regardless.

```json
{ "folders": ["docs", "guides", "api-reference"] }
```

Use when: repo is large and docs live only in specific subdirectories, you want to skip
examples/demos/internal-tools folders, or it's a monorepo where only one package should be
documented.

### `excludeFolders`
Takes priority over `folders`. Common patterns:

| Pattern | Excludes |
|---------|----------|
| `"node_modules"` | Any folder named `node_modules` anywhere in the tree |
| `".git"` | Version control metadata |
| `"build"`, `"dist"`, `"out"` | Build artifacts |
| `"src"` | Source code (if docs are separate) |
| `"test"`, `"__tests__"`, `"*.test"` | Test directories and files |
| `"docs/archive"`, `"docs/old"` | Specific deprecated documentation folders |
| `"examples"` | Example projects (if not meant as guides) |
| `"./config"` | Root-level config folder only |
| `"**/internal"` | Any folder named "internal" at any depth |

**Default i18n exclusions** (non-English folders, excluded automatically):
```
i18n/zh*, i18n/es*, i18n/fr*, i18n/de*, i18n/ja*, i18n/ko*
i18n/ru*, i18n/pt*, i18n/it*, i18n/ar*, i18n/hi*, i18n/tr*
zh-cn, zh-tw, zh-hk, es-mx, pt-br, fr-ca
```
To keep a non-English translation: set `excludeFolders` explicitly without those patterns, or
keep defaults and rely on Context7's LLM for multi-language content.

### `excludeFiles`
Excludes specific filenames across all directories.

**Default exclusions** (always applied unless overridden):
```
CHANGELOG.md, changelog.md, CHANGELOG.mdx, changelog.mdx
LICENSE.md, license.md
CODE_OF_CONDUCT.md, code_of_conduct.md
```

Common additional candidates: `HISTORY.md`, `LICENSE`, `CONTRIBUTING.md`, `SECURITY.md`.

### `rules`
Actionable, library-specific guidance for agents using the library — not generic best practices.
Keep each to one or two sentences. Example:

```json
{
  "rules": [
    "Always use async/await syntax; callbacks are deprecated",
    "Ensure rate limiting is configured before production deployment",
    "Use environment variables for API keys; never hardcode secrets"
  ]
}
```

### `previousVersions` / `branchVersions`
Only include versions still in active use, or where users need migration/debugging help across
significantly different APIs.

```json
{ "previousVersions": [{ "tag": "v1.2.1" }, { "tag": "v1.1.0" }] }
{ "branchVersions": [{ "branch": "stable" }, { "branch": "legacy" }] }
```

### `branch`
Set when docs live on a non-default branch: `{ "branch": "docs-only" }`.

## Full Example Configs

### Minimal
```json
{
  "$schema": "https://context7.com/schema/context7.json",
  "projectTitle": "My Awesome Library",
  "description": "A fast, type-safe library for building web apps"
}
```

### Standard Web Framework
```json
{
  "$schema": "https://context7.com/schema/context7.json",
  "projectTitle": "SvelteKit",
  "description": "The fastest way to build web apps with Svelte",
  "folders": ["docs", "packages/create-svelte"],
  "excludeFolders": [
    "node_modules",
    ".svelte-kit",
    "build",
    "dist",
    "**/.test.js",
    "docs/archive",
    "**/internal"
  ],
  "excludeFiles": ["CHANGELOG.md", "CONTRIBUTING.md"],
  "rules": [
    "Always use SvelteKit hooks for authentication and middleware",
    "Prefer server-side rendering for SEO-critical pages",
    "Use +page.svelte for UI and +page.server.js for server logic",
    "Keep +layout files minimal to avoid cascading re-renders"
  ]
}
```

### SDK/API Client Library
```json
{
  "$schema": "https://context7.com/schema/context7.json",
  "projectTitle": "Upstash Ratelimit",
  "description": "Ratelimiting library based on Upstash Redis",
  "folders": ["docs", "examples"],
  "excludeFolders": ["src", "test", "build", "node_modules", ".turbo", "examples/deprecated"],
  "excludeFiles": ["CHANGELOG.md", "LICENSE.md", "package.json"],
  "rules": [
    "Use Upstash Redis as the database backend",
    "Always set up single region for latency optimization",
    "Configure cache TTL explicitly to prevent stale tokens",
    "Test rate limit triggers before deploying to production"
  ],
  "previousVersions": [{ "tag": "v1.2.1" }, { "tag": "v1.1.0" }]
}
```

### Monorepo with Multiple Packages
```json
{
  "$schema": "https://context7.com/schema/context7.json",
  "projectTitle": "Design System Monorepo",
  "description": "Unified component library and design tokens",
  "folders": ["packages/components/docs", "packages/tokens/docs", "docs/guides"],
  "excludeFolders": [
    "node_modules",
    "packages/*/src",
    "packages/*/test",
    "packages/*/dist",
    "docs/archive",
    "docs/internal"
  ],
  "excludeFiles": ["CHANGELOG.md", "CONTRIBUTING.md", "package.json"],
  "rules": [
    "Import components from @design-system/components, not monorepo packages",
    "Use design tokens from @design-system/tokens for all styling",
    "Follow the component API patterns in the components documentation",
    "Use Storybook for component exploration and testing"
  ]
}
```

## Validation Checklist

- Schema included: `"$schema": "https://context7.com/schema/context7.json"`.
- Title & description set: clear, concise.
- `folders` (if used): only directories with actual documentation.
- Exclude patterns correct: mentally test against example paths.
- No circular logic: `excludeFolders` wins, avoid contradicting `folders`.
- Rules are actionable and library-specific.
- Versions listed are only actively-maintained ones.
- Valid JSON (no trailing commas, no comments).

## Troubleshooting Recipes

**Context7 indexes too much irrelevant content** → be more aggressive with `excludeFolders`:
```json
{
  "excludeFolders": [
    "node_modules", "build", "dist", ".next", ".nuxt",
    "src", "test", "__tests__", "examples", "playground", "docs/archive"
  ]
}
```

**Context7 misses important documentation** → use `folders` to explicitly include only relevant
paths: `{ "folders": ["docs", "api-reference"] }`

**Outdated or deprecated docs still show up** → exclude them explicitly:
`{ "excludeFolders": ["docs/v1", "docs/deprecated", "**/legacy"] }`

**Agents follow incorrect patterns from outdated versions** → remove old versions or clarify
rules: `{ "rules": ["Use v2 API only; v1 is no longer supported", "Migrate away from callbacks to Promises/async-await"] }`

**Non-English docs are interfering** → exclude explicitly:
`{ "excludeFolders": ["i18n/zh", "i18n/ja", "i18n/es"] }`

## Repository Workflow

**Owners**: create `context7.json` at repo root → commit/push to default branch → verify via
Context7 Dashboard if claimed → refine exclusions as needed → document in CONTRIBUTING.md if
helpful.

**Contributors submitting a PR**: explain *why* the change improves indexing, validate JSON
syntax and schema compliance, get maintainer approval before merging.

## Resources

- Context7 Docs: https://context7.com/docs/library-owners
- Adding Libraries: https://context7.com/docs/adding-libraries
- Claiming Libraries: https://context7.com/docs/howto/claiming-libraries
- Best Practices: https://context7.com/docs/tips
- GitHub Issues: https://github.com/upstash/context7/issues
