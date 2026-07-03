---
name: git-worktrees
description: >
  Manage git worktrees through the project's Makefile.
  Creates isolated worktrees for tasks, merges them back safely,
  and auto-installs Makefile.worktrees if missing.
license: MIT
metadata:
  category: git
  requires: make, git, bash
---

## What I do

I help manage git worktrees using the project's Makefile targets
`make worktree` and `make merge-worktree`.

Git worktrees allow you to work on multiple branches simultaneously
without stashing or switching contexts. This skill automates:

- **Creation** of a new worktree from a base branch.
- **Symlinking** of shared files, for example `.env`, `node_modules`, `vendor`.
- **Safe merging** of the worktree branch back into the base branch.
- **Pre-merge validation** to prevent losing uncommitted work.
- **Cleanup** after successful merge and push.
- **Auto-installation** of `Makefile.worktrees` if the project doesn't have it yet.

## When to use me

Use this skill when:

- The user asks to create a git worktree for a new task or feature.
- The user wants to merge a worktree branch back and clean up.
- You need to work on multiple branches in parallel without switching.
- You see `Makefile.worktrees` is missing or `make worktree` isn't available.

## Important Safety Rule

Never manually delete a worktree with `rm -rf` after `make merge-worktree` fails.

If `git worktree remove` or `make merge-worktree` reports that the worktree contains
modified or untracked files, first inspect the worktree:

```bash
git -C path/to/worktree status --short
```

Then either commit the intended changes:

```bash
git -C path/to/worktree add -A
git -C path/to/worktree commit -m "Describe changes"
```

or explicitly remove generated/unwanted files.

Only after the worktree is clean should you retry:

```bash
make merge-worktree BRANCH=my-feature BASE_BRANCH=main
```

Deleting a dirty worktree can permanently lose uncommitted changes.

## Workflow

### Step 1 — Check availability

Run the following to check if `worktree` and `merge-worktree` targets exist:

```bash
make help | grep -E "worktree|merge-worktree"
```

Or dry-run each target:

```bash
make -n worktree BRANCH=__check__ BASE_BRANCH=main 2>/dev/null || echo "MISSING: worktree"
make -n merge-worktree BRANCH=__check__ BASE_BRANCH=main 2>/dev/null || echo "MISSING: merge-worktree"
```

### Step 2 — Install `Makefile.worktrees` if missing

If the targets are missing, install the bundled Makefile:

1. **Copy** the bundled file `Makefile.worktrees`, stored next to this `SKILL.md`,
   into the project root.

2. **Include** it in the existing `Makefile` by adding this line near the top
   after variable declarations and before target declarations:

   ```makefile
   include Makefile.worktrees
   ```

3. **Verify** installation:

   ```bash
   make help | grep -E "worktree|merge-worktree"
   ```

The bundled `Makefile.worktrees` supports:

- `make worktree` — create a worktree.
- `make merge-worktree` — safely merge and clean up a worktree.

### Step 3 — Create a worktree

```bash
make worktree BRANCH=my-new-feature
```

Options:

- `BRANCH` **required** — name of the new branch and worktree directory.
- `BASE_BRANCH` optional — branch to create from. Defaults to the current branch.
- `LINK_FILES` optional — space-separated list of files/directories to symlink
  into the worktree, for example `.env`, `node_modules`, `vendor`.
- `WORKTREE_DIR` optional — directory for worktrees. Default: `.worktrees`.

Example with all options:

```bash
make worktree BRANCH=feat/auth BASE_BRANCH=main LINK_FILES=".env node_modules" WORKTREE_DIR=.wt
```

After creating the worktree, do the actual development inside the printed path:

```bash
cd .worktrees/feat/auth
```

### Step 4 — Commit changes inside the worktree

Before merging, make sure all intended changes in the worktree are committed:

```bash
git status --short
git add -A
git commit -m "Implement feature"
```

This is required because `git merge BRANCH` merges **commits**, not uncommitted
working tree changes.

If the worktree has modified or untracked files, `make merge-worktree` will abort
before doing anything destructive.

### Step 5 — Merge a worktree after finishing work

```bash
make merge-worktree BRANCH=my-new-feature BASE_BRANCH=main
```

Options:

- `BRANCH` **required** — branch to merge.
- `BASE_BRANCH` optional — target branch. Defaults to current branch.
- `PUSH` optional — whether to push `BASE_BRANCH` after merge. Default: `1`.
  Use `PUSH=0` to skip push.
- `DELETE_BRANCH` optional — whether to delete the merged local branch after
  successful cleanup. Default: `0`.

Example without push:

```bash
make merge-worktree BRANCH=my-new-feature BASE_BRANCH=main PUSH=0
```

Example with branch deletion after successful merge:

```bash
make merge-worktree BRANCH=my-new-feature BASE_BRANCH=main DELETE_BRANCH=1
```

What `merge-worktree` does:

1. Finds the worktree linked to `BRANCH`.
2. Verifies that the linked worktree is not the main worktree.
3. Verifies that the linked worktree is clean.
4. Verifies that the main worktree is clean.
5. Switches the main worktree to `BASE_BRANCH`.
6. Pulls latest `BASE_BRANCH` with `--ff-only`.
7. Merges `BRANCH` into `BASE_BRANCH`.
8. Pushes `BASE_BRANCH`, unless `PUSH=0`.
9. Removes the linked worktree.
10. Optionally deletes the local branch if `DELETE_BRANCH=1`.

## Best Practices

- Always specify `BRANCH=`.
- Prefer specifying `BASE_BRANCH=main` or another explicit target branch when merging.
- Commit all intended changes inside the worktree before running `merge-worktree`.
- Use `LINK_FILES` only for files/directories that are ignored or safe to symlink,
  for example `.env`, `node_modules`, `vendor`.
- The `.worktrees/` directory is auto-added to `.gitignore`.
- If merge conflicts occur during `merge-worktree`, resolve them in the main repo
  and finish manually.
- Do not use `rm -rf` on dirty worktrees.

## Safety Checks

`worktree` validates:

- `BRANCH` is provided.
- `BASE_BRANCH` is detectable or explicitly provided.
- `BRANCH` and `BASE_BRANCH` differ.
- The command runs inside a git repository.
- The local branch does not already exist.
- The target worktree path is not already occupied.

`merge-worktree` validates:

- `BRANCH` is provided.
- `BASE_BRANCH` is detectable or explicitly provided.
- `BRANCH` and `BASE_BRANCH` differ.
- The command runs inside a git repository.
- A worktree for `BRANCH` exists.
- The found worktree is not the main worktree.
- The branch worktree has no modified, staged, or untracked files.
- The main worktree has no modified, staged, or untracked files.
- Critical git commands succeed before moving to the next step.

## Files

- `Makefile.worktrees` — the bundled Makefile fragment that provides
  `worktree` and `merge-worktree` targets. Copy this file to the project root
  and include it in the main `Makefile`.

## See Also

- `man git-worktree`
- `make help`
