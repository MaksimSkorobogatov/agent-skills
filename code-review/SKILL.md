---
name: code-review
description: Review uncommitted code changes for quality, security, bugs, and compliance with project standards
license: MIT
metadata:
  category: quality
  requires: git
---

## What I do

I perform code reviews on uncommitted changes, checking for security vulnerabilities, bugs, hardcoded secrets, unintentional backdoors, and compliance with project coding standards defined in `AGENTS.md` and `CODE_STYLE.md`.

## When to use me

Use this skill when:
- Asked to review code before committing or merging
- A task is complete and needs final quality verification
- Reviewing changes in a git worktree branch before merging

## Workflow

### Phase 1: Detect Diff Strategy (parent agent)

Run lightweight git commands to determine **which diff command** the subagent should use. Do NOT read the diff output yourself.

```bash
# Are we in a worktree?
git rev-parse --is-inside-work-tree
git rev-parse --git-dir          # worktree if path contains .worktrees
git rev-parse --git-common-dir   # always points to main .git

# Branch info
git branch --show-current

# Check staged vs unstaged (names only, no content)
git diff --cached --name-only
git diff --name-only
```

Decision logic (apply in order):

1. **Worktree branch?** `.git-dir` path differs from `.git-common-dir` → diff against base branch.
   Detect base branch:
   ```bash
   git branch -r | grep -E 'origin/(main|master|develop|dev)$' | head -1
   # Fallback
   git log --oneline --ancestry-path --simplify-by-decoration HEAD | tail -1
   ```
   Diff command: `git diff <base_branch>...HEAD`

2. **Staged changes?** `git diff --cached --name-only` is non-empty → `git diff --cached`

3. **Unstaged changes?** `git diff --name-only` is non-empty → `git diff`

4. **Nothing to review** → report and exit.

### Phase 2: Delegate to Subagent

Spawn a **single subagent** using the `general` agent type. The subagent performs ALL heavy work: reading the diff, reading project docs, analyzing code.

**Subagent prompt:**

```
You are performing a code review. Your task:

1. Read project standards from AGENTS.md (and CODE_STYLE.md if it exists) in the project root.
2. Get the diff using this command: <diff_command>
3. Analyze every changed line against the checklist below.
4. Return the report in the specified output format.

Working directory: <project_root>

## Review Checklist

Analyze every changed line for ALL of the following categories:

### 1. Security
- Hardcoded secrets, tokens, passwords, API keys (even in comments or examples)
- SQL injection: raw string concatenation in queries instead of parameterized ($1, $2)
- XSS: unescaped user input in HTML/JSX, unsafe use of dangerouslySetInnerHTML
- Missing input validation on user-supplied data
- Insecure CORS configuration (wildcard * in production)
- Missing rate limiting on auth or public endpoints
- Plaintext password storage (must be hashed: bcrypt/argon2)
- JWT issues: missing exp/iss/aud checks, weak secret (<32 bytes)
- File upload: missing Content-Type/size/extension validation
- Path traversal in file operations
- Exposed error details or stack traces to clients

### 2. Unintentional Backdoors
- Debug endpoints or routes left in production code
- Overly permissive access control (e.g., admin check commented out)
- Disabled authentication or authorization checks
- Catch-all error handlers that swallow security errors
- Overly broad CORS or CSP policies
- Credentials or tokens logged to console/files
- Test/mock data or endpoints reachable in production
- Magic values that bypass validation (e.g., if id == -1 return all)
- Unsafe deserialization of user input
- Command injection via unsanitized shell execution

### 3. Bugs
- Unchecked error returns (especially in Go: _, err patterns)
- Nil/null pointer dereference without guard checks
- Resource leaks: unclosed files, connections, response bodies
- Race conditions: shared state without synchronization
- Off-by-one errors in loops or slice operations
- Missing break/return in switch/select statements
- Incorrect boolean logic (inverted conditions)
- Unreachable code after return/panic
- Missing default case in type switches
- Goroutine leaks (no cancellation mechanism)

### 4. Code Quality (per AGENTS.md)
- Go: gofmt compliance, exported names capitalized, error handling explicit
- React: components PascalCase, hooks camelCase with use prefix, props typed
- Code duplication (3+ occurrences must be extracted)
- Deprecated API usage
- Anti-patterns specific to the language/framework

### 5. Hardcode and Configuration
- Magic numbers without named constants
- Hardcoded URLs, ports, hostnames (should be configurable)
- Hardcoded paths (especially absolute paths)
- Docker: hardcoded env values instead of variables, latest tags

### 6. Documentation
- New public functions/components missing JSDoc or Go doc comments
- Complex business logic without explanatory comments
- Missing TODO/FIXME for incomplete work (per AGENTS.md format)

## Output Format

## Code Review Results

### Summary
- Files reviewed: <count>
- Critical issues: <count>
- Important issues: <count>
- Suggestions: <count>

### Critical Issues (must fix before commit)
For each issue:
- **File**: <path>:<line>
- **Category**: Security | Backdoor | Bug
- **Description**: <what is wrong>
- **Fix**: <how to fix it>

### Important Issues (should fix)
For each issue:
- **File**: <path>:<line>
- **Category**: Quality | Hardcode | Docs
- **Description**: <what is wrong>
- **Fix**: <how to fix it>

### Suggestions (optional improvements)
- <file>:<line> - <suggestion>

### Positive Findings
- <what is done well>

### Verdict
APPROVE | REQUEST_CHANGES
```

### Phase 3: Report

Present the subagent results to the user. If there are critical issues:
1. Highlight them clearly
2. Suggest specific fixes
3. Do not recommend committing until resolved

If the review passes:
1. Confirm the code is clean
2. Remind the user to commit when ready

## Important Notes

- This review is **read-only** — it never modifies code.
- The parent agent must NOT read diff content — all heavy I/O happens inside the subagent.
- If `AGENTS.md` or `CODE_STYLE.md` are missing, the subagent skips those checks and notes it in the report.
- Every finding must reference a specific `file:line` — no vague warnings.
