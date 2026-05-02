---
name: create-skill
description: Create new OpenCode agent skills with proper structure and conventions
license: MIT
compatibility: opencode
metadata:
  audience: developers
  workflow: universal
---

## What I do

I guide the creation of new OpenCode agent skills. A skill is a reusable set of instructions defined in a `SKILL.md` file that agents can discover and load on demand.

## When to use me

Use this when:
- Creating a new skill for a specific task or domain
- Setting up skill infrastructure for a project or team
- Migrating existing agent behaviors into reusable skills

## Skill Creation Guide

### 1. Choose a Location

Skills are discovered in these locations (in order of precedence):
- Project: `.opencode/skills/<name>/SKILL.md`
- Global: `~/.config/opencode/skills/<name>/SKILL.md`
- Compatibility: `.claude/skills/`, `.agents/skills/` paths also supported

### 2. Create Directory Structure

```
.opencode/skills/<skill-name>/SKILL.md
```

The directory name must match the skill name exactly.

### 3. Write SKILL.md Header

Every skill MUST start with a YAML frontmatter block:

```yaml
---
name: <skill-name>
description: <1-1024 character description>
license: <optional, e.g., MIT>
compatibility: <optional, e.g., opencode>
metadata: <optional key-value pairs>
---
```

### 4. Name Validation Rules

The `name` field must:
- Be 1-64 characters long
- Use lowercase letters and digits with single hyphens
- NOT start or end with a hyphen
- NOT contain consecutive hyphens (`--`)
- Match the directory name containing `SKILL.md`

Regex: `^[a-z0-9]+(-[a-z0-9]+)*$`

### 5. Write Skill Content

After the header, include:
- Clear description of what the skill does
- When to use the skill
- Specific instructions or patterns to follow
- Examples when helpful

## Example: Creating a `code-review` Skill

1. Create directory:
```bash
mkdir -p .opencode/skills/code-review
```

2. Write `.opencode/skills/code-review/SKILL.md`:
```markdown
---
name: code-review
description: Perform thorough code reviews with focus on security, performance, and maintainability
license: MIT
compatibility: opencode
metadata:
  audience: developers
---

## What I do

I perform comprehensive code reviews analyzing:
- Security vulnerabilities and input validation
- Performance implications and optimizations
- Code maintainability and readability
- Error handling completeness
- Test coverage adequacy

## When to use me

Use this when asked to review pull requests, code changes, or to perform a security audit.

## Review Process

1. Understand the context and purpose of changes
2. Review code for the issues listed above
3. Provide specific, actionable feedback
4. Suggest improvements with examples when helpful
5. Approve or request changes with clear reasoning

## Output Format

Present findings in organized sections:
- **Critical**: Must fix before merge
- **Important**: Should address before merge
- **Suggestions**: Optional improvements
- **Positive**: What's done well
```

## Best Practices

- Keep descriptions specific enough for agents to make good choices
- Use consistent naming conventions (kebab-case)
- Place project-specific skills in `.opencode/skills/`
- Place team-wide skills in global config
- Document clear trigger conditions for when to use the skill
- Include concrete examples and patterns
