---
name: fix-todo-fixme
description: Find all TODO and FIXME comments in codebase, plan and execute fixes using parallel subagents with minimal overlap, then remove the comments
license: MIT
compatibility: opencode
metadata:
  audience: developers
  language: multi
---

## What I do

I scan the entire codebase for `TODO` and `FIXME` comments, analyze each one, plan fixes/improvements, execute them in parallel using multiple subagents with non-overlapping work areas, and remove the comments after successful completion.

## When to use me

Use this when:
- Asked to fix all TODOs and FIXMEs in the codebase
- Requested to clean up technical debt markers
- Need to systematically resolve pending work items scattered across the code
- Want to improve code quality by addressing known issues

## Workflow Overview

This skill follows a structured 5-phase workflow:

```
Discovery -> Planning -> Parallel Execution -> Verification -> Cleanup
```

---

## Phase 1: Discovery

### 1.1 Scan for TODO and FIXME comments

Search the codebase for all occurrences of TODO and FIXME comments (case-insensitive).

Use the `grep` tool with patterns like:
- `(?i)\bTODO\b`
- `(?i)\bFIXME\b`

Exclude non-source files (binary, vendor, node_modules, .git, generated files).

### 1.2 Collect metadata for each item

For every found TODO/FIXME, record:
- **File path** and **line number**
- **Full comment text** (the description of what needs to be done)
- **Category**: fix, refactor, feature, optimization, documentation, test, cleanup
- **Affected area**: the package/module/file group it belongs to
- **Dependencies**: whether this item depends on or affects other TODO/FIXME items

### 1.3 Present discovery summary

Present all found items in a structured table:

| # | Type | File | Line | Description | Area |
|---|------|------|------|-------------|------|
| 1 | TODO | src/foo.go | 42 | Add input validation | src/foo |
| 2 | FIXME | src/bar.go | 88 | Race condition on shared map | src/bar |

---

## Phase 2: Planning

This phase is CRITICAL. Do NOT skip planning.

### 2.1 Cluster items by work area

Group TODO/FIXME items by their **area of influence**:
- Items touching the same file
- Items touching the same package/module
- Items with cross-dependencies

### 2.2 Design subagent assignments

Create subagent assignments following these rules:

1. **Minimize overlap**: Each subagent should work on a distinct set of files. Two subagents MUST NOT edit the same file.
2. **Respect dependencies**: If item B depends on item A, they MUST be in the same subagent batch or sequential.
3. **Balance workload**: Distribute items roughly evenly across subagents.
4. **Use git worktrees**: When running 3+ subagents, prefer creating git worktrees so each subagent works in isolation:
   ```bash
   git worktree add ../fix-agent-1 agent-1-branch
   git worktree add ../fix-agent-2 agent-2-branch
   ```
5. **Maximum parallelism**: Group independent items together for concurrent execution.

### 2.3 Present the execution plan

Show the user the plan before executing:

```
Execution Plan:
===============

Subagent 1 (branch: fix/agent-1, worktree: ../fix-agent-1)
  Files: internal/foo/*.go
  Items: #1, #3, #5
  
Subagent 2 (branch: fix/agent-2, worktree: ../fix-agent-2)  
  Files: internal/bar/*.go
  Items: #2, #4

Subagent 3 (branch: fix/agent-3, worktree: ../fix-agent-3)
  Files: pkg/utils/*.go, pkg/config/*.go
  Items: #6, #7, #8

No file overlaps detected. Dependencies respected.
```

### 2.4 Ask for confirmation

Use the `question` tool to confirm the plan with the user before proceeding.

---

## Phase 3: Parallel Execution

### 3.1 Create git worktrees (if applicable)

For each subagent group, create a worktree and branch:
```bash
git worktree add ../fix-agent-N fix/agent-N
```

### 3.2 Launch subagents

Use the `task` tool with `subagent_type: general` for each group.

Each subagent prompt MUST include:

1. **Exact list of files** to edit (no other files)
2. **Exact list of TODO/FIXME items** to fix, with file:line references
3. **Full comment text** for each item so the subagent knows what to do
4. **Instructions** to:
   - Read the file(s) before editing
   - Implement the fix/improvement as described in the comment
   - Remove the TODO/FIXME comment after fixing
   - Follow project code style (reference AGENTS.md if exists)
   - NOT touch any files outside its assigned scope
5. **Worktree path** as working directory if using worktrees

Example subagent prompt structure:

```
You are fixing TODO/FIXME items in the codebase.

Working directory: /path/to/worktree

Files to edit: internal/foo/handler.go, internal/foo/service.go

Items to fix:
1. internal/foo/handler.go:42 - TODO: Add input validation for user ID
   - Add validation that userID is a positive integer
   - Return 400 Bad Request if invalid
   - REMOVE the TODO comment after implementing

2. internal/foo/service.go:88 - FIXME: Race condition on cache access
   - Add sync.RWMutex to protect cache reads/writes
   - REMOVE the FIXME comment after implementing

Rules:
- ONLY edit the files listed above
- REMOVE each TODO/FIXME comment after implementing the fix
- Follow existing code patterns in the project
- Do not add new dependencies without checking go.mod first
```

### 3.3 Monitor subagent progress

Wait for all subagents to complete. Collect their results.

---

## Phase 4: Verification

### 4.1 Verify no remaining TODO/FIXME in edited files

Re-scan the edited files to confirm all targeted TODO/FIXME comments were removed:
```bash
grep -n "TODO\|FIXME" <edited_files>
```

### 4.2 Run project tests

Execute the project's test suite to ensure nothing is broken:
```bash
make test
# or
go test ./...
```

### 4.3 Run linting

Execute the project's linter:
```bash
make lint
# or
golangci-lint run
```

### 4.4 Fix any issues

If tests or lint fail, fix them before proceeding. Use additional subagents if needed for parallel fixes on different areas.

---

## Phase 5: Cleanup and Commit

### 5.1 Merge worktrees (if used)

If worktrees were used, merge each branch back:

```bash
# For each worktree
cd /main/repo
git merge fix/agent-1
git merge fix/agent-2
# ...
```

Resolve any merge conflicts.

### 5.2 Remove worktrees

```bash
git worktree remove ../fix-agent-1
git worktree remove ../fix-agent-2
# ...
git worktree prune
```

### 5.3 Final verification

Re-run tests and lint on the merged result.

### 5.4 Present summary to user

Show a summary of all changes made:

```
Summary:
========
Resolved 8 TODO/FIXME items across 12 files.

Items fixed:
- [DONE] internal/foo/handler.go:42 - Added input validation
- [DONE] internal/foo/service.go:88 - Fixed race condition
- [DONE] ...

No remaining TODO/FIXME in edited files.
All tests passing. Lint clean.
```

### 5.5 Commit (only if requested)

Only commit when the user explicitly asks. Use format:
```
[ai] Resolve N TODO/FIXME items across <areas>

- Fix 1 description
- Fix 2 description
- Removed TODO/FIXME comments after resolution
```

---

## Decision Tree: Worktrees vs Direct Edits

```
Number of subagents needed?
├── 1 subagent
│   └── Edit files directly (no worktree needed)
├── 2 subagents  
│   └── If file overlap is avoidable: edit directly in parallel
│   └── If risky: use worktrees
└── 3+ subagents
    └── ALWAYS use worktrees for isolation
```

## Conflict Prevention Rules

1. **One file, one owner**: Assign each file to exactly one subagent
2. **Shared dependencies**: If two items need the same helper function, put them in the same subagent
3. **Interface changes**: Changes to interfaces/types that multiple packages use MUST be sequential, not parallel
4. **Test files**: Include test files in the same subagent as their corresponding source files
5. **Config files**: Treat config changes as a separate sequential step after parallel fixes

## Quality Checks Per Subagent

Each subagent MUST verify before marking complete:
- [ ] All assigned TODO/FIXME comments removed
- [ ] No new TODO/FIXME introduced
- [ ] Code compiles (`go build ./...` or equivalent)
- [ ] Affected tests pass (`go test <package>` or equivalent)
- [ ] No lint errors in edited files
