# Context7 Configuration Agent Skill

**Purpose**: Guide agents in writing and tuning `context7.json` files to optimize how Context7 indexes GitHub repositories for AI-assisted documentation discovery.

**When to use this skill**: When an agent needs to create, review, modify, or validate a `context7.json` configuration file for a library or framework repository.

---

## What is context7.json?

`context7.json` is a configuration file (similar to `robots.txt`) that sits at the root of a GitHub repository and tells Context7 how to parse and present your project's documentation to coding agents. It controls *where* Context7 looks for documentation, but not *what file types* it reads—Context7 automatically indexes markdown files, documentation sites, and falls back to source code if needed.

---

## Core Workflow for Writing context7.json

### 1. Analyze the Repository Structure

Before writing configuration, examine the repo to understand:
- **Documentation locations**: Where are docs stored? (e.g., `/docs`, `/documentation`, wiki files, README at root)
- **Folder organization**: Are there multiple folders that should be scanned? (e.g., `/docs`, `/api-reference`, `/guides`)
- **What to exclude**: Are there test files, deprecated docs, archived versions, node_modules, or build artifacts?
- **Multiple versions**: Does the library have versioned docs using branches or git tags?
- **Library type**: Is it a framework, SDK, library, tool, or package? This affects how you structure folders.

**Questions to ask**:
- What is the primary entry point for developers using this library?
- Are there outdated or archived docs that should be hidden?
- Does the repository have a `node_modules`, `build`, `dist`, or `.git` folder that should be ignored?
- Are there multiple language translations (`i18n`) that should be excluded (except English)?

### 2. Start with a Minimal Configuration

Begin with the simplest valid configuration and expand only if needed:

```json
{
  "$schema": "https://context7.com/schema/context7.json",
  "projectTitle": "Your Project Name",
  "description": "Brief description of what your project does"
}
```

**Why minimal?** By default, Context7 scans the entire repository and excludes known irrelevant patterns (CHANGELOG.md, LICENSE.md, archived folders, non-English i18n). Only add explicit rules if the defaults don't fit your repo.

### 3. Configure Folder Inclusion (if needed)

Use `folders` only if you want to *restrict* Context7 to specific directories:

```json
{
  "folders": ["docs", "guides", "api-reference"]
}
```

**When to use `folders`**:
- Your repo is large and docs are only in specific subdirectories.
- You want to ignore entire branches of the repo (e.g., internal tools, examples, demos in separate folders).
- You have a monorepo with multiple projects and only one should be documented.

**Important**: If `folders` is empty or omitted, Context7 scans the entire repository (respecting `excludeFolders`). Root-level markdown files (like README.md) are *always* included.

### 4. Configure Folder Exclusion

Use `excludeFolders` to filter out unwanted content. This takes priority over `folders`.

#### Common Exclusion Patterns

| Pattern | Excludes |
|---------|----------|
| `"node_modules"` | Any folder named `node_modules` anywhere in the tree |
| `".git"` | Version control metadata |
| `"build"`, `"dist"`, `"out"` | Build artifacts |
| `"src"` | Source code (if you have separate docs) |
| `"test"`, `"__tests__"`, `"*.test"` | Test directories and files |
| `"docs/archive"`, `"docs/old"` | Specific deprecated documentation folders |
| `"examples"` | Example projects (if they're not meant as guides) |
| `"./config"` | Root-level config folder only |
| `"**/internal"` | Any folder named "internal" at any depth |

#### Language & Localization Patterns

By default, Context7 excludes non-English i18n folders:
```
i18n/zh*, i18n/es*, i18n/fr*, i18n/de*, i18n/ja*, i18n/ko*
i18n/ru*, i18n/pt*, i18n/it*, i18n/ar*, i18n/hi*, i18n/tr*
zh-cn, zh-tw, zh-hk, es-mx, pt-br, fr-ca
```

**If you want to keep a non-English translation**, either:
1. Don't use the default exclusions (set `excludeFolders` explicitly without those patterns)
2. Or keep the defaults and rely on Context7's LLM to handle multi-language content

**Custom example**:
```json
{
  "excludeFolders": [
    "node_modules",
    ".git",
    "build",
    "dist",
    "src",
    "**/__tests__",
    "docs/archive"
  ]
}
```

### 5. Configure File Exclusion

Use `excludeFiles` to exclude specific filenames (applies to all directories):

```json
{
  "excludeFiles": [
    "CHANGELOG.md",
    "LICENSE.md",
    "CODE_OF_CONDUCT.md",
    "CONTRIBUTING.md",
    "SECURITY.md"
  ]
}
```

**Common candidates**:
- `CHANGELOG.md`, `HISTORY.md` – Historical version logs (not user docs)
- `LICENSE.md`, `LICENSE` – Legal text (not technical docs)
- `CODE_OF_CONDUCT.md` – Community guidelines (not library docs)
- `CONTRIBUTING.md` – Developer workflows (only relevant for contributors, not users)
- `SECURITY.md` – Security policies (rarely relevant to end users)

**Default exclusions** (always applied unless overridden):
```
CHANGELOG.md, changelog.md, CHANGELOG.mdx, changelog.mdx
LICENSE.md, license.md
CODE_OF_CONDUCT.md, code_of_conduct.md
```

### 6. Add Usage Rules (Optional but Recommended)

Define best practices or constraints that agents using your library should follow:

```json
{
  "rules": [
    "Always use async/await syntax; callbacks are deprecated",
    "Ensure rate limiting is configured before production deployment",
    "Use environment variables for API keys; never hardcode secrets",
    "Test thoroughly with mocked Upstash Redis connections"
  ]
}
```

**What makes a good rule?**
- Actionable: Agents can understand and apply it.
- Library-specific: Don't repeat general best practices.
- Safety/best-practice focused: Warn about common mistakes.
- Concise: Keep to one or two sentences each.

### 7. Handle Multiple Versions (Optional)

If your library has versioned documentation:

#### Using Git Tags (Recommended)
```json
{
  "previousVersions": [
    { "tag": "v1.2.1" },
    { "tag": "v1.1.0" },
    { "tag": "v1.0.0" }
  ]
}
```

#### Using Branches
```json
{
  "branchVersions": [
    { "branch": "stable" },
    { "branch": "legacy" }
  ]
}
```

**When to include previous versions**:
- Users are still actively using older versions.
- You want agents to help debug or migrate between versions.
- Each version has significantly different APIs.

### 8. Specify an Alternative Branch (Optional)

If documentation lives on a different branch than `main` or `master`:

```json
{
  "branch": "docs-only"
}
```

---

## Complete Example Configurations

### Minimal Configuration
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
  "excludeFiles": [
    "CHANGELOG.md",
    "CONTRIBUTING.md"
  ],
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
  "excludeFolders": [
    "src",
    "test",
    "build",
    "node_modules",
    ".turbo",
    "examples/deprecated"
  ],
  "excludeFiles": [
    "CHANGELOG.md",
    "LICENSE.md",
    "package.json"
  ],
  "rules": [
    "Use Upstash Redis as the database backend",
    "Always set up single region for latency optimization",
    "Configure cache TTL explicitly to prevent stale tokens",
    "Test rate limit triggers before deploying to production"
  ],
  "previousVersions": [
    { "tag": "v1.2.1" },
    { "tag": "v1.1.0" }
  ]
}
```

### Monorepo with Multiple Packages
```json
{
  "$schema": "https://context7.com/schema/context7.json",
  "projectTitle": "Design System Monorepo",
  "description": "Unified component library and design tokens",
  "folders": [
    "packages/components/docs",
    "packages/tokens/docs",
    "docs/guides"
  ],
  "excludeFolders": [
    "node_modules",
    "packages/*/src",
    "packages/*/test",
    "packages/*/dist",
    "docs/archive",
    "docs/internal"
  ],
  "excludeFiles": [
    "CHANGELOG.md",
    "CONTRIBUTING.md",
    "package.json"
  ],
  "rules": [
    "Import components from @design-system/components, not monorepo packages",
    "Use design tokens from @design-system/tokens for all styling",
    "Follow the component API patterns in the components documentation",
    "Use Storybook for component exploration and testing"
  ]
}
```

---

## Validation Checklist

Before finalizing `context7.json`, verify:

- ✅ **Schema included**: `"$schema": "https://context7.com/schema/context7.json"` enables editor autocomplete.
- ✅ **Title & description set**: Clear, concise project name and description.
- ✅ **Folders** (if used): Only include directories with actual documentation.
- ✅ **Exclude patterns are correct**: Test patterns mentally or verify with example paths.
- ✅ **No circular logic**: `excludeFolders` takes priority, so avoid contradictions.
- ✅ **Rules are actionable**: Each rule should guide agent behavior concretely.
- ✅ **Versions appropriate**: Only list versions still in active use.
- ✅ **Valid JSON**: No syntax errors (use a JSON validator).

---

## Tuning & Troubleshooting

### Problem: Context7 indexes too much irrelevant content

**Solution**: Be more aggressive with `excludeFolders`:
```json
{
  "excludeFolders": [
    "node_modules",
    "build",
    "dist",
    ".next",
    ".nuxt",
    "src",
    "test",
    "__tests__",
    "examples",
    "playground",
    "docs/archive"
  ]
}
```

### Problem: Context7 misses important documentation

**Solution**: Use `folders` to explicitly include only relevant paths:
```json
{
  "folders": ["docs", "api-reference"]
}
```

### Problem: Outdated or deprecated docs still show up

**Solution**: Exclude them explicitly:
```json
{
  "excludeFolders": ["docs/v1", "docs/deprecated", "**/legacy"]
}
```

### Problem: Agents follow incorrect patterns from outdated versions

**Solution**: Remove old versions or add clearer rules:
```json
{
  "rules": [
    "Use v2 API only; v1 is no longer supported",
    "Migrate away from callbacks to Promises/async-await"
  ]
}
```

### Problem: Non-English docs are interfering

**Solution**: Explicitly exclude them:
```json
{
  "excludeFolders": ["i18n/zh", "i18n/ja", "i18n/es"]
}
```

---

## Integration with Repository Workflow

### Steps for Repository Owners

1. **Create `context7.json`** at the root of your repository.
2. **Commit and push** to your default branch.
3. **Verify via Context7 Dashboard** (if you've claimed your library):
   - Check that indexed content matches your intent.
   - Refine exclusions if needed.
4. **Document configuration** in a CONTRIBUTING.md note if needed (optional).
5. **Sync with team**: If collaborating, discuss configuration in PRs.

### For Contributors

If submitting a PR to update `context7.json`:
1. Clearly explain *why* the change improves indexing.
2. Test locally (use Context7 CLI if available).
3. Validate JSON syntax and schema compliance.
4. Get approval from maintainers before merging.

---

## Key Takeaways

| Concept | Key Point |
|---------|-----------|
| **Purpose** | Control *where* Context7 scans, not *what* file types it reads. |
| **Minimal first** | Start with just `projectTitle` and `description`; add rules only when needed. |
| **Folders logic** | Use `folders` to *include* specific paths; `excludeFolders` always wins. |
| **Default exclusions** | Context7 auto-excludes changelogs, licenses, non-English translations. |
| **Rules matter** | Good rules guide agents toward library best practices. |
| **Versions** | Include only actively maintained versions. |
| **Validation** | Use editor autocomplete via `$schema` to catch mistakes early. |

---

## Resources

- **Context7 Docs**: https://context7.com/docs/library-owners
- **Adding Libraries**: https://context7.com/docs/adding-libraries
- **Claiming Libraries**: https://context7.com/docs/howto/claiming-libraries
- **Best Practices**: https://context7.com/docs/tips
- **GitHub Issues**: https://github.com/upstash/context7/issues
