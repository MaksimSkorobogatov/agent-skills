---
name: fix-linter-issues
description: Fix linter errors and code style issues identified by make lint
---

## What I do

I analyze linter output from `make lint`, categorize issues by package, and spawn subagents to fix issues in parallel. I ensure fixes are safe and do not break project functionality.

## When to use me

Use this skill when:
- Code quality checks fail after running `make lint`
- There are style violations, unused code, or other linting errors
- A CI/CD pipeline reports linter failures

## Workflow

### Phase 1: Initial Analysis

1. Run `make lint` to identify all issues
2. Parse the output to categorize issues by:
   - Package path
   - Issue type (error, warning)
   - Linter rule violated
3. Create a summary of issues found

### Phase 2: Parallel Fix via Subagents

For each package with issues:

1. **Spawn a subagent** using the `general` agent type
2. **Assign scope**: Provide the subagent with:
   - Package path
   - List of specific issues for that package
   - Relevant file paths
   - Linter rule descriptions

3. **Subagent task instructions**:

   ```
   Fix linter issues in package: <package-path>
   
   Issues to fix:
   <list-of-issues>
   
   Files affected:
   <list-of-files>
   
    Guidelines:
    1. Read the affected file(s) and understand the context
    2. For each linter issue, ask yourself: "Will this fix break the project?"
    3. If the fix might break functionality:
       - DO NOT apply the fix
       - Document the issue with explanation why it shouldn't be changed
       - Suggest an alternative approach if possible
    4. If the fix is safe:
       - Apply the fix following project conventions
       - Ensure code remains idiomatic Go
    5. After fixes, verify the code still compiles: `go build ./<package-path>`
   
   Return a summary of:
   - Issues fixed
   - Issues skipped with explanations
   - Any concerns or recommendations
   ```

4. **Run subagents in parallel** for all packages

### Phase 3: Verification

1. Run `make lint` again to check for remaining issues
2. Analyze results:
   - If all issues are resolved: Report success
   - If issues remain:
     - Categorize as "cannot be fixed automatically" or "skipped by design"
     - Provide detailed explanation for each remaining issue
     - Suggest manual fixes where appropriate

## Safety Checks

Before applying any fix, subagents must consider:

1. **Functionality preservation**: Will the change alter program behavior?
2. **API compatibility**: Is the code part of a public API that must remain stable?
3. **Test coverage**: Are there tests that verify the current behavior?
4. **Dependencies**: Will removing code break other packages that depend on it?
5. **Code semantics**: Does the "unused" code serve a purpose (e.g., interface implementation)?
6. **Context handling** — NEVER merge unrelated contexts, even if the linter suggests it. This is critical:
   - If a function receives a short-lived context (e.g., `OnStart(ctx context.Context)`) and must launch a background operation with a longer lifespan, these contexts MUST remain separate.
   - The linter may flag this as "unused variable" or suggest merging contexts, but merging would create a bug: cancelling the parent context would prematurely terminate the long-running operation.
   - **Rule**: When a short context controls function entry/lifecycle and a long context controls background work, keep them as distinct variables. Suppress the linter warning if needed (e.g., `//nolint:...` or rename the short context to `_` if unused beyond the check).

## Example: Issue Categories to Handle

- **Unused imports**: Safe to remove unless used by code generation
- **Unused variables**: Usually safe, but check if side effects are intended.
- **Dead code**: Verify not needed for tests or future use
- **Style violations**: Formatting issues, naming conventions
- **Error handling**: Missing error checks, ignored return values
- **Complexity issues**: Cyclomatic complexity, function length

## Output Format

Final report should include:

```
## Linter Fix Summary

### Total Issues Found: <number>
### Issues Fixed: <number>
### Issues Requiring Manual Review: <number>

### Fixed Issues by Package:
- <package>: <count> issues fixed

### Skipped Issues (with explanations):
- <package>/<file>:<line> - <issue> - <reason>

### Remaining Issues (if any):
- <package>/<file>:<line> - <issue> - <recommendation>

### Verification:
- [ ] `make lint` passes
- [ ] `make build` succeeds
- [ ] `make test` passes
```

## Important Notes

- Never apply fixes that change program semantics without explicit approval
- When in doubt, document the concern rather than apply a risky fix
- Respect code that may be used by reflection, code generation, or external tools
- Prefer conservative fixes that maintain existing behavior
