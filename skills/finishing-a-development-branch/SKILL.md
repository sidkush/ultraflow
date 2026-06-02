---
name: finishing-a-development-branch
description: Use when implementation is complete and the user needs merge, PR, keep, or discard handling for the current branch.
---

# Finishing A Development Branch

## Core Rule

Verify branch in place. Do not create a worktree or branch. The operator manages branches.

## Part A: Start/Resume Check

When invoked before work:

1. Run `git status --short`, `git branch --show-current`, and recent log.
2. Confirm the branch is already the intended branch.
3. Run or record the relevant baseline.
4. Dirty tree -> HALT and ASK unless the operator explicitly scoped the dirty files.

Do not run `git worktree add`, `git checkout -b`, or branch creation unless the operator explicitly asks in the current turn.

## Part B: Finish

Prerequisite: `ultraflow:verification-before-completion` has fresh evidence, or the response clearly says what was not verified.

Present exactly four choices unless the user already specified one:

1. Merge locally.
2. Push and create PR.
3. Keep branch as-is.
4. Discard work.

Discard requires typed confirmation. Never force-delete without it.

## Push/PR

Use explicit staging. In mixed worktrees, stage only intended files. Do not stage unrelated user changes.

Windows guardrails:

- Use separate commands instead of `&&`.
- Avoid `sed -i`.
- Quote paths.
- For multiline commit messages, write a UTF-8-no-BOM temp file and use `git commit -F`.

## Cleanup

Only remove worktrees you can prove this workflow created. Harness-owned or operator-owned worktrees stay in place.

## Red Flags

- Creating a branch/worktree because the old workflow said so.
- Proceeding with failing verification.
- Staging `git add -A` in a mixed tree without explicit scope.
- Deleting work without typed confirmation.
