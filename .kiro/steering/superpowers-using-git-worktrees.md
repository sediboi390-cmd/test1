---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans
inclusion: manual
---

# Using Git Worktrees

## Overview

Ensure work happens in an isolated workspace.

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Step 0: Detect Existing Isolation

Before creating anything, check if already in an isolated workspace:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

**If `GIT_DIR != GIT_COMMON`:** Already in a linked worktree. Skip to Step 3.

**If `GIT_DIR == GIT_COMMON`:** In a normal repo. Ask for consent before creating a worktree:

> "Would you like me to set up an isolated worktree? It protects your current branch from changes."

## Step 1: Create Isolated Workspace

```bash
# Verify .worktrees is ignored
git check-ignore -q .worktrees 2>/dev/null

# If NOT ignored: add to .gitignore and commit first

# Create worktree
git worktree add .worktrees/$BRANCH_NAME -b $BRANCH_NAME
cd .worktrees/$BRANCH_NAME
```

## Step 2: Project Setup

Auto-detect and run appropriate setup:

```bash
if [ -f package.json ]; then npm install; fi
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
```

## Step 3: Verify Clean Baseline

Run tests to ensure workspace starts clean. If tests fail, report and ask whether to proceed.

## Red Flags

**Never:**
- Create a worktree when already in one
- Create worktree without verifying it's ignored (project-local)
- Skip baseline test verification
- Proceed with failing tests without asking
