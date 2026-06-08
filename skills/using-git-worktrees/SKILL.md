---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - ensures an isolated workspace exists via native tools or git worktree fallback
---

# Using Git Worktrees

## Overview

Ensure work happens in an isolated workspace. Prefer your platform's native worktree tools. Fall back to manual git worktrees only when no native tool is available.

**Core principle:** Detect existing isolation first. Then use native tools. Then fall back to git. Never fight the harness.

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Step 0: Detect Existing Isolation

**Before creating anything, check if you are already in an isolated workspace.**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**Submodule guard:** `GIT_DIR != GIT_COMMON` is also true inside git submodules. Before concluding "already in a worktree," verify you are not in a submodule:

```bash
# If this returns a path, you're in a submodule, not a worktree — treat as normal repo
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**If `GIT_DIR != GIT_COMMON` (and not a submodule):** You are already in a linked worktree. Skip to Step 2 (Project Setup). Do NOT create another worktree.

Report with branch state:
- On a branch: "Already in isolated workspace at `<path>` on branch `<name>`."
- Detached HEAD: "Already in isolated workspace at `<path>` (detached HEAD, externally managed). Branch creation needed at finish time."

**If `GIT_DIR == GIT_COMMON` (or in a submodule):** You are in a normal repo checkout.

Has the user already indicated their worktree preference in your instructions? If not, ask for consent before creating a worktree:

> "Would you like me to set up an isolated worktree? It protects your current branch from changes."

Honor any existing declared preference without asking. If the user declines consent, work in place and skip to Step 2.

## Step 0.5: Verify the Base Branch Is Current

**Runs after Step 0 confirms a worktree is needed, before EITHER creation mechanism (1a native tool or 1b git).** A worktree forked from a stale base silently bakes outdated code (or omits unpushed work) into the feature branch. What the worktree forks *from* depends on the mechanism:

- **Native tool, default `worktree.baseRef: fresh`** → forks from **`origin/<default-branch>`** (e.g. `origin/dev`). The tool does *not* fetch, so this ref can be stale — a plain `git fetch` is the whole fix.
- **Native tool `worktree.baseRef: head`, or the git fallback** (`git worktree add -b`) → forks from your **current local HEAD**, which must itself be made current.

### Always: refresh, then warn on unpushed work

```bash
git fetch origin --quiet 2>/dev/null      # refreshes origin/<default-branch> + all upstreams — fixes a stale `fresh` fork

DEFAULT=$(git rev-parse --abbrev-ref origin/HEAD 2>/dev/null); DEFAULT=${DEFAULT#origin/}
# the LOCAL branch that TRACKS origin/<default-branch> (usually same name, e.g. local `dev`) — NOT necessarily the checked-out one
TRACKING=$(git for-each-ref --format='%(refname:short) %(upstream:short)' refs/heads \
  | awk -v u="origin/$DEFAULT" '$2==u{print $1; exit}')
AHEAD=$(git rev-list --count "origin/$DEFAULT..$TRACKING" 2>/dev/null)   # unpushed commits on the tracking branch
```

A `fresh` fork forks from `origin/<default-branch>`, not from any local branch — so after the fetch there is **nothing to fast-forward**.

**If `AHEAD > 0`** — the local branch tracking `origin/<default-branch>` (e.g. local `dev`) has unpushed commits a `fresh` fork will **exclude** — **ask**:

> "Local `<tracking>` is N commits ahead of `origin/<default-branch>` (unpushed). A fresh worktree forks from `origin/<default-branch>` and won't include them. Push first, or fork without them?"

### Only when forking from local HEAD (git fallback, or `baseRef: head`)

The base is your current branch, so make *it* current — fast-forward to its upstream:

```bash
BASE=$(git branch --show-current)         # empty when detached HEAD
if [ -n "$BASE" ] && git rev-parse --verify --quiet "origin/$BASE" >/dev/null; then
  LOCAL=$(git rev-parse "$BASE"); REMOTE=$(git rev-parse "origin/$BASE")
  BASE_MB=$(git merge-base "$BASE" "origin/$BASE")
  # LOCAL == REMOTE              -> current, proceed
  # behind (LOCAL == BASE_MB)    -> fast-forward (ask only if uncommitted changes block it)
  # ahead / diverged             -> ask before forking
fi
# detached HEAD or no origin/$BASE upstream -> skip, report externally-managed
```

**Behind** — a fast-forward is always possible (local is an ancestor); **just do it**, even with uncommitted changes (git keeps non-overlapping ones):

```bash
git merge --ff-only "origin/$BASE" -q
```

If the FF **fails** (uncommitted changes would be overwritten — git refuses and leaves the tree untouched), **ask**, recommending stash → FF → pop:

> "`<base>` is N commits behind `origin/<base>` but uncommitted changes block the fast-forward. Stash them, fast-forward, then restore (`git stash push -u && git merge --ff-only origin/<base> && git stash pop`)?"

**Ahead / diverged** — **ask** before forking; never silently pick a side. **Detached HEAD / no upstream** — skip, report externally-managed.

## Step 1: Create Isolated Workspace

**You have two mechanisms. Try them in this order.**

### 1a. Native Worktree Tools (preferred)

The user has asked for an isolated workspace (Step 0 consent). Do you already have a way to create a worktree? It might be a tool with a name like `EnterWorktree`, `WorktreeCreate`, a `/worktree` command, or a `--worktree` flag. If you do, use it and skip to Step 2.

Native tools handle directory placement, branch creation, and cleanup automatically. Using `git worktree add` when you have a native tool creates phantom state your harness can't see or manage.

Only proceed to Step 1b if you have no native worktree tool available.

### 1b. Git Worktree Fallback

**Only use this if Step 1a does not apply** — you have no native worktree tool available. Create a worktree manually using git.

#### Directory Selection

Follow this priority order. Explicit user preference always beats observed filesystem state.

1. **Check your instructions for a declared worktree directory preference.** If the user has already specified one, use it without asking.

2. **Check for an existing project-local worktree directory:**
   ```bash
   ls -d .worktrees 2>/dev/null     # Preferred (hidden)
   ls -d worktrees 2>/dev/null      # Alternative
   ```
   If found, use it. If both exist, `.worktrees` wins.

3. **If there is no other guidance available**, default to `.worktrees/` at the project root.

#### Safety Verification (project-local directories only)

**MUST verify directory is ignored before creating worktree:**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:** Add to .gitignore, commit the change, then proceed.

**Why critical:** Prevents accidentally committing worktree contents to repository.

#### Create the Worktree

```bash
# Determine path based on chosen location
path="$LOCATION/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**Sandbox fallback:** If `git worktree add` fails with a permission error (sandbox denial), tell the user the sandbox blocked worktree creation and you're working in the current directory instead. Then run setup and baseline tests in place.

## Step 2: Project Setup

Auto-detect and run appropriate setup:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## Step 3: Verify Clean Baseline

Run tests to ensure workspace starts clean:

```bash
# Use project-appropriate command
npm test / cargo test / pytest / go test ./...
```

**If tests fail:** Report failures, ask whether to proceed or investigate.

**If tests pass:** Report ready.

### Report

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| Already in linked worktree | Skip creation (Step 0) |
| In a submodule | Treat as normal repo (Step 0 guard) |
| Before any worktree creation | `git fetch origin` (Step 0.5) |
| Default `fresh` base | Forks from origin/<default> — fetch is the whole fix (Step 0.5) |
| Tracking branch has unpushed commits | Ask: push or fork without (Step 0.5) |
| Forking from local HEAD, behind | Fast-forward it (Step 0.5) |
| FF blocked by uncommitted changes | Ask: stash → FF → pop (Step 0.5) |
| Local HEAD ahead / diverged | Ask before forking (Step 0.5) |
| Detached HEAD or no upstream | Skip the HEAD reconcile (Step 0.5) |
| Native worktree tool available | Use it (Step 1a) |
| No native tool | Git worktree fallback (Step 1b) |
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check instruction file, then default `.worktrees/` |
| Directory not ignored | Add to .gitignore + commit |
| Permission error on create | Sandbox fallback, work in place |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

## Common Mistakes

### Fighting the harness

- **Problem:** Using `git worktree add` when the platform already provides isolation
- **Fix:** Step 0 detects existing isolation. Step 1a defers to native tools.

### Skipping detection

- **Problem:** Creating a nested worktree inside an existing one
- **Fix:** Always run Step 0 before creating anything

### Skipping ignore verification

- **Problem:** Worktree contents get tracked, pollute git status
- **Fix:** Always use `git check-ignore` before creating project-local worktree

### Assuming directory location

- **Problem:** Creates inconsistency, violates project conventions
- **Fix:** Follow priority: explicit instructions > existing project-local directory > default

### Proceeding with failing tests

- **Problem:** Can't distinguish new bugs from pre-existing issues
- **Fix:** Report failures, get explicit permission to proceed

## Red Flags

**Never:**
- Skip `git fetch` before creating the worktree — a default `fresh` fork from `origin/<default-branch>` would be stale (Step 0.5)
- Fork a `fresh` worktree while the tracking branch has unpushed commits without warning — they won't be in the fork (Step 0.5)
- Fast-forward a local branch in `fresh` mode — the fork ignores it; only fetch matters (Step 0.5)
- Force a fast-forward over uncommitted changes — if the FF is blocked, ask (stash → FF → pop)
- Create a worktree when Step 0 detects existing isolation
- Use `git worktree add` when you have a native worktree tool (e.g., `EnterWorktree`). This is the #1 mistake — if you have it, use it.
- Skip Step 1a by jumping straight to Step 1b's git commands
- Create worktree without verifying it's ignored (project-local)
- Skip baseline test verification
- Proceed with failing tests without asking

**Always:**
- Run Step 0 detection first
- Verify the base branch is current before creating the worktree (Step 0.5)
- Prefer native tools over git fallback
- Follow directory priority: explicit instructions > existing project-local directory > default
- Verify directory is ignored for project-local
- Auto-detect and run project setup
- Verify clean test baseline
