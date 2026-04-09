---
name: onboard
description: "Onboard a Claude session on the current codebase with smart caching and delta updates. Optimized for git worktree workflows — onboard once, share context across all worktrees."
allowed-tools: "Bash Read Write Glob Grep"
argument-hint: "[--read-all] [--refresh]"
---

# Onboard

Onboard this Claude session on the current codebase. Produces a cached summary stored in the git directory, shared across all worktrees. Subsequent sessions load the cache instantly and only update what changed.

## When to Use

- The user says "onboard", "onboard this project", or runs `/onboard`
- The user starts a new session on an unfamiliar codebase
- The user switches to a worktree and needs project context

## Flags

- **No flags (default):** Key files mode — reads manifests, configs, entry points, READMEs, and a sample of source files. Fast, low token cost.
- **`--read-all`:** Full scan — reads all text-based source files. Thorough, higher token cost. Use for initial onboarding on a new project or when key files mode missed important context.
- **`--refresh`:** Forces a full rescan even if a cached summary exists. Combines with `--read-all` if both are passed.

## Setup

Determine the shared git directory:

```bash
git rev-parse --git-common-dir
```

The onboard summary lives at `<git-common-dir>/claude-onboard.md`. Using `--git-common-dir` (not `--git-dir`) ensures the summary is shared across all worktrees.

If this is not a git repository, tell the user:
> "This directory isn't a git repository. Onboard requires git to track changes and share context across worktrees."

Stop here.

## Behavior

Check whether `<git-common-dir>/claude-onboard.md` exists and read it if so.

### Case 1: No existing summary OR `--refresh` flag

Perform a scan based on the mode:

**File exclusions (always applied):**
Skip these directories: `.git`, `node_modules`, `vendor`, `.DS_Store`, `dist`, `build`, `.cache`, `__pycache__`, `.tox`, `.venv`, `.next`, `.nuxt`, `coverage`, `.nyc_output`, `.terraform`, `.serverless`.
Skip these file types: images (png, jpg, gif, svg, ico, webp), fonts (woff, woff2, ttf, eot, otf), compiled assets (min.js, min.css, map), archives (zip, tar, gz), binaries, lock files (package-lock.json, composer.lock, yarn.lock, Gemfile.lock).

**Key files mode (default):**

1. Read manifest and config files first:
   - READMEs (README.md, README.txt, readme.txt)
   - CLAUDE.md, AGENTS.md if they exist
   - Package manifests (package.json, composer.json, Package.swift, requirements.txt, Gemfile, go.mod, Cargo.toml, pyproject.toml)
   - Build/CI configs (Makefile, Dockerfile, .github/workflows/*.yml, webpack.config.*, vite.config.*, tsconfig.json, phpunit.xml, .eslintrc.*, .phpcs.xml)
   - Environment templates (.env.example, .env.sample)

2. Read entry point files:
   - Look for main files referenced in manifests (e.g., `main` in package.json, bootstrap/entry files)
   - Read the top-level source files in the primary source directory
   - Read up to 5 representative source files from subdirectories to understand patterns and conventions

3. Read the directory structure (file listing) to understand the full layout without reading every file.

**Full scan mode (`--read-all`):**

1. Read manifest and config files first (same as key files mode).
2. List all files recursively (respecting exclusions above).
3. Read all remaining text-based source files.

**Write the summary:**

Generate the summary following the structure in `references/summary-template.md`. Write it to `<git-common-dir>/claude-onboard.md`.

The frontmatter `mode` field should be `key-files` or `read-all` depending on which mode was used.

Print the summary to the conversation and include:

> Onboard complete from `<sha>` on `<branch>` at `<timestamp>` (mode: <mode>). If you commit, merge, or switch branches, run `/onboard` again to update.

### Case 2: Summary exists and stored SHA matches current HEAD

Load the existing summary into the conversation and print:

> Loaded cached onboard from `<sha>` on `<branch>` at `<timestamp>` (mode: <mode>). Run `/onboard --refresh` to force a full rescan.

### Case 3: Summary exists but stored SHA differs from current HEAD

Perform a delta update:

1. Run `git diff --name-status <stored-sha> HEAD` to get added, modified, and deleted files.
2. Read ONLY the added and modified files (respecting exclusions and the mode of the original scan — if the original was `key-files`, only read added/modified files that qualify as key files; if `read-all`, read all added/modified text files).
3. Note deleted files.
4. Load the existing summary.
5. Update the summary to reflect changes: incorporate new/modified file understanding, remove references to deleted files, adjust architecture notes if warranted.
6. Update frontmatter (sha, branch, timestamp). Preserve the original mode.
7. Write the updated summary to `<git-common-dir>/claude-onboard.md`.
8. Print the updated summary and include:

> Onboard updated from `<old-sha>` to `<new-sha>` on `<branch>` at `<timestamp>` (<N> files changed). Run `/onboard --refresh` to force a full rescan.

## Important

- Always use `git rev-parse --git-common-dir` (not `--git-dir`) so the summary is shared across worktrees.
- Only read text-based source files. Skip images, fonts, compiled binaries, archives, lock files, and similar.
- Keep the summary concise but complete enough to orient a session that has zero context on the codebase.
- When doing delta updates, preserve the overall structure of the summary — don't rewrite sections that weren't affected by the changes.
- The summary is a compression, not a transcript. Summarize patterns and architecture, don't paste entire files.
