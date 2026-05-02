# My Agent Skills

A curated collection of reusable AI agent skills for [OpenCode](https://opencode.ai) and compatible agents (Claude, Codex, etc.).

Each skill is a self-contained set of instructions defined in a `SKILL.md` file that agents can discover and load on demand.

## Included Skills

| Skill | Description |
|-------|-------------|
| [`create-skill`](./create-skill/SKILL.md) | Guides the creation of new agent skills with proper structure, naming conventions, and YAML frontmatter |
| [`fix-linter-issues`](./fix-linter-issues/SKILL.md) | Analyzes `make lint` output, categorizes issues by package, and spawns subagents to fix them in parallel with safety checks |
| [`fix-todo-fixme`](./fix-todo-fixme/SKILL.md) | Scans the codebase for `TODO` and `FIXME` comments, plans fixes, executes them via parallel subagents, and removes the comments |
| [`git-worktrees`](./git-worktrees/SKILL.md) | Manages git worktrees through Makefile targets; includes a bundled `Makefile.worktrees` for creating isolated worktrees and merging them back |

## Installation

Skills are automatically discovered from these locations (in order of precedence):

1. **Project-local**: `.agents/skills/<name>/SKILL.md`
2. **Global**: `~/.agents/skills/<name>/SKILL.md`
3. **Compatibility paths**: eg. `.claude/skills/`

To install a skill from this repo, copy the desired skill directory into one of the locations above:

```bash
# Example: install git-worktrees globally
mkdir -p ~/.agents/skills
cp -r git-worktrees ~/.agents/skills/
```

## Skill Format

Every skill follows a standard structure:

```
<skill-name>/
  └── SKILL.md          # Required: skill definition with YAML frontmatter
  └── <auxiliary-files>  # Optional: Makefiles, templates, helpers
```

Each `SKILL.md` starts with YAML header:

```yaml
---
name: <skill-name>
description: <what the skill does>
license: <optional, e.g. MIT>
compatibility: <optional, e.g. opencode>
metadata:
  <optional key-value pairs>
---
```

## License

MIT © 2026 Maksim Skorobogatov

See [LICENSE](LICENSE) for details.
