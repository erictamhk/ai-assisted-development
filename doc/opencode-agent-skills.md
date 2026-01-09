# OpenCode Agent Skills Implementation

**Version:** 1.0
**Last Updated:** 2026-01-09
**Source:** OpenCode Documentation (opencode.ai/docs)

---

## Overview

OpenCode is an open-source AI coding agent that supports Agent Skills through a native `skill` tool. Skills are reusable behavior definitions discovered from your repository or home directory.

---

## OpenCode Agent Architecture

### 1. Agent Types

OpenCode has two types of agents:

| Type | Description |
|------|-------------|
| **Primary Agents** | Main assistants you interact with directly (Build, Plan) |
| **Subagents** | Specialized assistants invoked by `@` mentions or automatically |

#### Built-in Agents

| Agent | Type | Purpose |
|-------|------|---------|
| **build** | Primary | Default, full access for development |
| **plan** | Primary | Read-only analysis and planning |
| **general** | Subagent | Complex searches, multi-step tasks |
| **explore** | Subagent | Fast codebase exploration |

---

## Agent Skills in OpenCode

### 1. Skill Structure

```
.opencode/
└── skill/
    └── <skill-name>/
        └── SKILL.md
```

### 2. File Locations

OpenCode searches these locations for skills:

| Location | Type | Path |
|----------|------|------|
| Project config | Local | `.opencode/skill/<name>/SKILL.md` |
| Global config | Global | `~/.config/opencode/skill/<name>/SKILL.md` |
| Claude-compatible | Local | `.claude/skills/<name>/SKILL.md` |
| Claude-compatible | Global | `~/.claude/skills/<name>/SKILL.md` |

### 3. Discovery Mechanism

- **Project-local**: Walks up from current directory to git worktree
- **Global**: Loads from `~/.config/opencode/skill/*/SKILL.md`
- **Claude compatibility**: Also loads from Claude-compatible paths

---

## SKILL.md Format

### 1. Required Frontmatter

```yaml
---
name: skill-name
description: Brief description (1-1024 chars)
license: MIT
compatibility: opencode
metadata:
  audience: maintainers
  workflow: github
---
```

### 2. Frontmatter Fields

| Field | Required | Rules |
|-------|----------|-------|
| `name` | Yes | 1-64 chars, lowercase alphanumeric, hyphens |
| `description` | Yes | 1-1024 chars |
| `license` | No | Any string |
| `compatibility` | No | e.g., "opencode" |
| `metadata` | No | String-to-string map |

### 3. Name Validation

```
^[a-z0-9]+(-[a-z0-9]+)*$
```

- Must match directory name
- No consecutive hyphens
- Cannot start/end with hyphen

### 4. Skill Content

After frontmatter, include:

```markdown
## What I do
- Description of capabilities

## When to use me
- Guidance on when to invoke

## Examples
- Usage examples
```

---

## Example: Git Release Skill

```yaml
---
name: git-release
description: Create consistent releases and changelogs
license: MIT
compatibility: opencode
metadata:
  audience: maintainers
  workflow: github
---

## What I do
- Draft release notes from merged PRs
- Propose a version bump
- Provide a copy-pasteable `gh release create` command

## When to use me
Use this when you are preparing a tagged release.
Ask clarifying questions if the target versioning scheme is unclear.
```

Location: `.opencode/skill/git-release/SKILL.md`

---

## Skill Tool Integration

### 1. Tool Description

OpenCode lists available skills in the `skill` tool description:

```xml
<available_skills>
  <skill>
    <name>git-release</name>
    <description>Create consistent releases and changelogs</description>
  </skill>
</available_skills>
```

### 2. Loading a Skill

Agents load a skill by calling:

```json
skill({ "name": "git-release" })
```

### 3. On-Demand Loading

Skills are loaded via the native `skill` tool - agents see available skills and can load full content when needed.

---

## Permission System

### 1. Global Permissions

Configure in `opencode.json`:

```json
{
  "permission": {
    "skill": {
      "pr-review": "allow",
      "internal-*": "deny",
      "experimental-*": "ask",
      "*": "allow"
    }
  }
}
```

| Permission | Behavior |
|------------|----------|
| `allow` | Skill loads immediately |
| `deny` | Skill hidden from agent, access rejected |
| `ask` | User prompted for approval |

### 2. Per-Agent Override

**For custom agents (in agent frontmatter):**

```yaml
---
permission:
  skill:
    "documents-*": "allow"
---
```

**For built-in agents (in opencode.json):**

```json
{
  "agent": {
    "plan": {
      "permission": {
        "skill": {
          "internal-*": "allow"
        }
      }
    }
  }
}
```

### 3. Disable Skills Entirely

**For custom agents:**

```yaml
---
tools:
  skill: false
---
```

**For built-in agents:**

```json
{
  "agent": {
    "plan": {
      "tools": {
        "skill": false
      }
    }
  }
}
```

When disabled, `<available_skills>` is omitted from tool description.

---

## AGENTS.md Integration

OpenCode also supports AGENTS.md for custom instructions:

### 1. File Locations

| Type | Location |
|------|----------|
| Project | `AGENTS.md` in project root |
| Global | `~/.config/opencode/AGENTS.md` |

### 2. Precedence

1. Local files (traversing up from current directory)
2. Global file (`~/.config/opencode/AGENTS.md`)

Both are combined if present.

### 3. Example

```markdown
# SST v3 Monorepo Project

This is an SST v3 monorepo with TypeScript.

## Project Structure
- `packages/` - Contains all workspace packages
- `infra/` - Infrastructure definitions

## Code Standards
- Use TypeScript with strict mode
- Shared code goes in `packages/core/`
```

### 4. External File References

OpenCode supports referencing external files:

**Using opencode.json (recommended):**

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": ["docs/guidelines.md", ".cursor/rules/*.md"]
}
```

**Manual in AGENTS.md:**

```markdown
## TypeScript Guidelines
For TypeScript best practices: @docs/typescript-guidelines.md
```

---

## Custom Agents with Skills

OpenCode allows creating custom agents that use skills:

### 1. JSON Configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "code-reviewer": {
      "description": "Reviews code for best practices",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-20250514",
      "prompt": "You are a code reviewer...",
      "tools": {
        "write": false,
        "edit": false
      },
      "permission": {
        "skill": {
          "pr-review": "allow"
        }
      }
    }
  }
}
```

### 2. Markdown Configuration

Place in `~/.config/opencode/agent/review.md`:

```yaml
---
description: Reviews code for quality
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.1
tools:
  write: false
  edit: false
---

You are in code review mode. Focus on:
- Code quality and best practices
- Potential bugs and edge cases
- Security considerations
```

### 3. Create Command

```bash
opencode agent create
```

Interactive command that:
1. Asks location (global or project)
2. Description of agent purpose
3. Generates system prompt
4. Selects available tools
5. Creates markdown file

---

## Troubleshooting

### Skill Not Showing Up

1. Verify `SKILL.md` is spelled in all caps
2. Check frontmatter includes `name` and `description`
3. Ensure skill names are unique across all locations
4. Check permissions (skills with `deny` are hidden)

### Permission Issues

- Patterns use wildcards: `internal-*` matches `internal-docs`
- Rules evaluated in order, last matching rule wins
- User can always invoke via `@` even if task permissions deny

---

## Comparison: OpenCode vs Claude Code Skills

| Feature | OpenCode | Claude Code |
|---------|----------|-------------|
| Skill Locations | `.opencode/skill/`, `~/.claude/skills/` | `.claude/skills/` |
| Frontmatter | YAML | YAML |
| Permission System | Pattern-based | Basic |
| Subagent Invocation | `@` mentions | Subagents |
| AGENTS.md Support | Yes | Via MCP |
| Open Source | Yes | Proprietary |

---

## Best Practices for OpenCode Skills

### 1. Naming
- Use descriptive, action-oriented names
- Follow pattern: `<action>-<target>`
- Examples: `git-release`, `pr-review`, `code-analyzer`

### 2. Description
- Keep under 1024 characters
- Be specific about when to use
- Mention key capabilities

### 3. Content Structure
- Use `## What I do` section
- Use `## When to use me` section
- Provide examples

### 4. Permissions
- Set appropriate permission levels
- Use patterns for organization
- Override per agent when needed

### 5. Testing
- Test skill discovery in clean environment
- Verify permissions work correctly
- Check cross-platform compatibility

---

## References

### Official Documentation
- [OpenCode Skills](https://opencode.ai/docs/skills/)
- [OpenCode Agents](https://opencode.ai/docs/agents/)
- [OpenCode Rules](https://opencode.ai/docs/rules/)
- [AGENTS.md Specification](https://agents.md/)

### GitHub Resources
- [OpenCode Repository](https://github.com/anomalyco/opencode)
- [Anthropics Skills](https://github.com/anthropics/skills)

### Related Standards
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Agent Skills (agentskills.io)](https://agentskills.io)

---

**End of Document**
