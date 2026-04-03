---
name: git-flow
description: Use when starting feature work needing isolation or when finishing a development branch. Handles worktree creation, dependency setup, branch completion with merge/PR/discard options.
---

# Git Flow

## When to Use
- **Starting work**: Before implementing a plan that should be isolated
- **Finishing work**: After all tests pass and verification confirms completion

## Part A: Starting (Worktree Setup)

### Step 1: Create Worktree
```bash
git worktree add -b feature/<name> <path> HEAD
```

**Path selection priority:**
1. Existing `worktrees/` or `.worktrees/` directory in project
2. Path specified in CLAUDE.md
3. Ask user

### Step 2: Setup Dependencies
Auto-detect and run:
- `package.json` → `npm install`
- `Cargo.toml` → `cargo build`
- `requirements.txt` → `pip install -r requirements.txt`
- `go.mod` → `go mod download`
- `pyproject.toml` → `pip install -e .` or `poetry install`

### Step 3: Baseline Test
Run the project's test suite in the worktree. If tests fail before any changes → the main branch has issues. Report to user before proceeding.

### Step 4: Verify Gitignore
If worktree is inside the project directory, ensure the worktree path is in `.gitignore`. Warn if not.

## Part B: Finishing (Branch Completion)

### Prerequisite
ALL tests must pass. Run `verification` first. Do NOT proceed with failing tests.

### Step 1: Verify Clean State
```bash
git status          # no uncommitted changes
git diff            # no unstaged changes
[test command]      # all tests pass
```

### Step 2: Present Options
Show exactly **4 options** using AskUserQuestion:

1. **Merge locally**: Merge branch into main/current branch
2. **Push + PR**: Push branch and create a pull request
3. **Keep as-is**: Leave the branch for later
4. **Discard**: Delete branch and worktree (requires typing "discard" to confirm)

### Step 3: Execute Choice
- **Merge**: `git checkout main && git merge feature/<name> --no-ff`
- **Push + PR**: `git push -u origin feature/<name>` then `gh pr create`
- **Keep**: Do nothing, report branch name and worktree path
- **Discard**: Require user to type "discard", then `git worktree remove <path> && git branch -D feature/<name>`

### Step 4: Cleanup
Remove worktree if merged or discarded. Report final state.

## Anti-Patterns
- Merging with failing tests
- Force-deleting without confirmation
- Leaving orphaned worktrees
- Creating worktrees without checking gitignore
