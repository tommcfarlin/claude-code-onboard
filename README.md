# Onboard

Onboard a Claude session on your codebase in one command. Produces a cached summary shared across all git worktrees — onboard once, every session starts with context.

> **Note:** This skill is experimental and actively being evaluated in real-world workflows. It works, but expect rough edges and changes as it matures. See [EVALUATION.md](EVALUATION.md) for the testing methodology.

## The Problem

Every time you start a new Claude session, it knows nothing about your project. It reads the directory, opens files, scans configs — the same orientation dance, every single time. If you use git worktrees, multiply that by every worktree you have open.

You burn tokens and time on the same discovery work that could be done once and cached.

## How It Works

Run `/onboard` and the skill:

1. **Reads your project** — manifests, configs, entry points, and key source files
2. **Produces a summary** — project purpose, tech stack, directory structure, architecture
3. **Caches it in git** — stored at the git common directory, shared across all worktrees
4. **Stays current** — on subsequent runs, detects what changed via `git diff` and only updates the delta

The cache lives at `<git-common-dir>/claude-onboard.md`. Every worktree shares the same file.

## Usage

Onboard with key files (fast, default):

```
/onboard
```

Onboard with a full scan of all source files:

```
/onboard --read-all
```

Force a fresh scan (ignores cache):

```
/onboard --refresh
```

Combine flags:

```
/onboard --read-all --refresh
```

## What Happens on Subsequent Runs

**Same commit (cache hit):** Loads the cached summary instantly. No file reads.

**Different commit (delta update):** Reads only the files that changed since the last onboard. Updates the summary. Fast.

**`--refresh` flag:** Ignores the cache and does a full scan from scratch.

## Worktree Workflow

This is where onboard shines. The summary is stored in the git common directory, which is shared across all worktrees:

```
# Worktree 1: onboard the project
/onboard

# Worktree 2: same repo, different branch — cache is already there
/onboard
> Loaded cached onboard from abc1234 on main at 2026-04-09T10:00:00Z
```

No redundant scanning. Each worktree picks up where the last one left off and only pays for its own changes.

## Key Files vs Read All

| Mode | What it reads | When to use |
|------|---------------|-------------|
| **Key files** (default) | Manifests, configs, entry points, READMEs, sample source files | Day-to-day sessions, quick orientation |
| **Read all** (`--read-all`) | Every text-based source file in the project | First time on a new project, deep dives, when key files mode missed something |

Key files mode is fast and cheap. Use `--read-all` when you need the full picture.

## What Gets Cached

The summary at `<git-common-dir>/claude-onboard.md` contains:

- **Project Summary** — what the project does
- **Technologies** — languages, frameworks, tools
- **Directory Structure** — tree overview of the layout
- **Key Entry Points** — main files and scripts that drive the project
- **Architecture Notes** — how the pieces fit together

## Requirements

- A git repository (the cache uses git's common directory)

## Installation

Clone the repo directly into your Claude Code skills directory:

```bash
git clone https://github.com/tommcfarlin/claude-code-onboard.git ~/.claude/skills/onboard
```

After that, `/onboard` is available in any Claude Code session.
