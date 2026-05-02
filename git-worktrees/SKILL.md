---
name: git-worktrees
description: >
  Manage git worktrees through the project's Makefile.
  Creates isolated worktrees for tasks, merges them back,
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

- **Creation** of a new worktree from a base branch
- **Symlinking** of shared files (e.g., `.env`, `node_modules`, `vendor`)
- **Merging** the worktree branch back into the base branch
- **Cleanup** after merge (removing the worktree and optionally the branch)
- **Auto-installation** of `Makefile.worktrees` if the project doesn't have it yet

## When to use me

Use this skill when:

- The user asks to create a git worktree for a new task/feature.
- The user wants to merge a worktree branch back and clean up.
- You need to work on multiple branches in parallel without switching.
- You see `Makefile.worktrees` is missing or `make worktree` isn't available.

## Workflow

### Step 1 — Check availability

Run the following to check if `worktree` and `merge-worktree` targets exist:

```bash
make help | grep -E "worktree|merge-worktree"
```

Or dry-run each target:

```bash
make worktree --dry-run      2>/dev/null || echo "MISSING: worktree"
make merge-worktree --dry-run 2>/dev/null || echo "MISSING: merge-worktree"
```

### Step 2 — Install `Makefile.worktrees` if missing

If the targets are missing, install the bundled Makefile:

1. **Copy** the bundled file `Makefile.worktrees` (stored next to this SKILL.md)
   into the project root.
2. **Include** it in the existing `Makefile` by adding this line near the top
   (after variables declarations and before target declarations):

   ```makefile
   include Makefile.worktrees
   ```

3. **Verify** installation:

   ```bash
   make help | grep -E "worktree|merge-worktree"
   ```

> The bundled `Makefile.worktrees` supports:
> - `make worktree` — create a worktree
> - `make merge-worktree` — merge and clean up

### Step 3 — Create a worktree

```bash
make worktree BRANCH=my-new-feature
```

Options:

- `BRANCH` **(required)** — name of the new branch and worktree directory.
- `BASE_BRANCH` *(optional)* — branch to create from. Defaults to current branch.
- `LINK_FILES` *(optional)* — space-separated list of files/directories to symlink
  into the worktree (e.g., `.env`, dependencies).
- `WORKTREE_DIR` *(optional)* — directory for worktrees. Default: `.worktrees`.

Example with all options:

```bash
make worktree BRANCH=feat/auth BASE_BRANCH=main LINK_FILES=".env node_modules" WORKTREE_DIR=.wt
```

### Step 4 — Merge a worktree after finishing work

```bash
make merge-worktree BRANCH=my-new-feature
```

Options:

- `BRANCH` **(required)** — branch to merge.
- `BASE_BRANCH` *(optional)* — target branch. Defaults to current branch.

What `merge-worktree` does:

1. Finds the worktree linked to `BRANCH`.
2. Switches the main tree to `BASE_BRANCH`.
3. Pulls latest `BASE_BRANCH`.
4. Merges `BRANCH` into `BASE_BRANCH`.
5. Pushes `BASE_BRANCH`.
6. Removes the worktree.

## Best Practices

- **Always specify `BRANCH=`** — it's the only required parameter.
- Use `LINK_FILES` for large dependency folders (e.g., `node_modules`, `vendor`)
  to avoid copying.
- The `.worktrees/` directory is auto-added to `.gitignore`.
- If merge conflicts occur during `merge-worktree`, resolve them in the main repo
  and finish manually.

## Safety Checks

Both targets validate:

- `BRANCH` is provided.
- `BASE_BRANCH` is detectable (or explicitly provided).
- `BRANCH` != `BASE_BRANCH`.
- The repository is inside a git tree.
- The branch doesn't already exist locally.
- The worktree path is not already occupied.

## Files

- `Makefile.worktrees` — the bundled Makefile fragment that provides
  `worktree` and `merge-worktree` targets. **Copy this file to the project root
  and include it in the main Makefile.**

## See Also

- `man git-worktree`
- `make help`
