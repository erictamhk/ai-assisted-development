# AGENTS.md Standard

This document covers the AGENTS.md standard - a cross-platform format for providing persistent project context and instructions to AI coding agents.

## Core Concept

AGENTS.md is a simple Markdown file placed at the root of a repository (or in subdirectories for monorepos) that AI coding agents automatically discover and incorporate into their context. Think of it as a "README for agents" - a dedicated file that gives AI assistants the context, conventions, and guidelines they need to work effectively on your codebase without requiring repetitive explanations in every conversation.

The fundamental insight behind AGENTS.md is that large language models are stateless functions. They have no inherent knowledge of your project structure, coding standards, or team conventions. AGENTS.md solves this by providing a single, predictable location where agents can automatically load project-specific instructions at the start of every session.

---

## AGENTS.md Format Specification

### Hierarchical Discovery

AGENTS.md files follow a hierarchical discovery pattern:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 AGENTS.md DISCOVERY HIERARCHY                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  /AGENTS.md                          ← Root-level (applies to all)     │
│  │                                                                       │
│  /packages/                                                                │
│    ├── backend/                                                           │
│    │   ├── AGENTS.md           ← Backend-specific rules                 │
│    │   └── src/                                                         │
│    │                                                                       │
│    └── frontend/                                                          │
│        ├── AGENTS.md           ← Frontend-specific rules                │
│        └── src/                                                         │
│                                                                         │
│  Discovery order:                                                         │
│  1. Start from current working directory                                 │
│  2. Walk up to repository root                                           │
│  3. Concatenate all AGENTS.md files found                               │
│  4. Closer files take precedence (override earlier)                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Core Sections

While AGENTS.md has no required fields, effective files typically include:

| Section | Purpose | Example Content |
|---------|---------|-----------------|
| **Project Overview** | What the project does | "E-commerce platform built with Node.js" |
| **Tech Stack** | Key technologies | "TypeScript, React, PostgreSQL, Redis" |
| **Architecture** | High-level patterns | "Clean Architecture with DDD" |
| **Development Setup** | Getting started | Build commands, environment setup |
| **Code Standards** | Style and patterns | Naming conventions, file organization |
| **Testing Requirements** | Quality standards | Coverage targets, test types |
| **Security Guidelines** | Constraints | Auth patterns, secret handling |

### Example AGENTS.md

```markdown
# E-Commerce Platform

A modern e-commerce platform built with TypeScript, React, and Node.js.

## Overview

- **Purpose**: Multi-tenant e-commerce solution for B2B and B2C
- **Tech Stack**: TypeScript, React, Node.js, PostgreSQL, Redis
- **Architecture**: Clean Architecture with DDD patterns

## Development Setup

### Prerequisites
- Node.js 20+
- pnpm 8+
- Docker (for local services)

### Getting Started
```bash
pnpm install
pnpm dev
```

### Running Tests
```bash
pnpm test          # Run all tests
pnpm test:unit     # Unit tests only
pnpm test:integration  # Integration tests
```

## Code Standards

### Language Conventions
- TypeScript with strict mode
- Use functional patterns where possible
- Prefer immutability

### Code Organization
```
src/
  domain/     # Business logic (DDD entities, value objects)
  application/ # Use cases, services
  infrastructure/ # External integrations, repositories
  interfaces/  # Public APIs, DTOs
```

### Testing Requirements
- Unit tests for all business logic
- 80%+ coverage on new code
- Integration tests for API endpoints

## Security Guidelines

### Sensitive Data
- Never commit .env files
- Use environment variables for secrets
- Rotate API keys immediately if compromised

### Code Review Requirements
- Security review for auth changes
- Performance review for data operations
- All changes require at least one reviewer
```

---

## CLI AI Agent Tools Supporting AGENTS.md

### OpenAI Codex

Codex CLI reads AGENTS.md files with sophisticated discovery logic:

```json
// ~/.codex/settings.json
{
  "project_doc_filenames": ["AGENTS.md", "AGENTS.override.md"],
  "project_doc_fallback_filenames": ["TEAM_GUIDE.md", ".agents.md"],
  "context_depth": 5
}
```

**Discovery precedence:**
1. Global scope: `~/.codex/AGENTS.override.md` or `AGENTS.md`
2. Project scope: Walk from root to current directory
3. Concatenate all found files, later files override earlier

### OpenCode

OpenCode reads AGENTS.md from both project and global locations:

```json
// ~/.config/opencode/opencode.json
{
  "instructions": [
    "AGENTS.md",
    "packages/*/AGENTS.md"
  ],
  "load_strategy": "on-demand"
}
```

- Project-specific rules apply when working in a directory
- Global rules in `~/.config/opencode/AGENTS.md` apply across all sessions
- On-demand loading prevents context overflow

### Gemini CLI

```json
// ~/.gemini/settings.json
{
  "contextFileName": "AGENTS.md",
  "model": "gemini-2.5-pro",
  "maxOutputTokens": 8192
}
```

### Aider

```yaml
# .aider.conf.yml
read: AGENTS.md
auto-remerge: true
```

---

## Best Practices

### Keep It Concise Yet Complete

AGENTS.md should contain everything an agent needs, but avoid unnecessary verbosity. Agents have limited context windows, so prioritize the most important information:

- Focus on conventions that, if violated, would require significant rework
- Include the "why" behind conventions when it helps understanding
- Be specific rather than using vague terms like "clean code"

### Update It Regularly

Treat AGENTS.md as living documentation:
- Add new conventions when established
- Update when existing patterns change
- Include "Last Updated" or version information
- Link to changelog for traceability

### Use Nested Files for Large Projects

For monorepos, create AGENTS.md files in individual packages:

```
/AGENTS.md              # Project-wide conventions
/packages/
  backend/
    AGENTS.md           # Backend-specific rules
  frontend/
    AGENTS.md           # Frontend-specific rules
```

### Document Commands Explicitly

Include exact commands rather than expecting agents to discover them:
```markdown
## Build Commands
- `pnpm build` - Build all packages
- `pnpm build:api` - Build only API package
```

### Include Security and Operational Guidelines

Document constraints so agents work within them:
```markdown
## Operational Constraints
- Production deployments require approval from Tech Lead
- Database migrations must be reviewed before execution
- External API calls must have timeout and retry logic
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| **Stale Documentation** | Outdated guidance is worse than none | Regular review schedule |
| **Vague Instructions** | No actionable guidance | Specific patterns and examples |
| **Human-Only Content** | Duplication, wasted context | Link to README, don't copy |
| **Ignoring Size Limits** | Context overflow, important info lost | Split into focused files |

---

## On-Demand Rule Loading

For large projects, use on-demand loading to prevent context overflow:

```json
// opencode.json
{
  "instructions": [
    "AGENTS.md"
  ],
  "onDemandRules": [
    {
      "pattern": "packages/api/**/*",
      "rules": ["packages/api/AGENTS.md", "docs/api-rules.md"]
    },
    {
      "pattern": "packages/web/**/*", 
      "rules": ["packages/web/AGENTS.md", "docs/frontend-rules.md"]
    }
  ],
  "load_strategy": "on-demand",
  "default_rules": ["AGENTS.md"]
}
```

This approach keeps base context small while allowing agents to load relevant rules when working on specific areas.

---

## AGENTS.md Ecosystem

### Tool Support

| Category | Tools |
|----------|-------|
| **CLI Agents** | Codex, Gemini CLI, OpenCode, Aider |
| **IDE Extensions** | VS Code, Zed, Cursor |
| **Commercial** | Devin, Copilot Coding Agent |

### Interoperability

AGENTS.md works with similar formats:
- **CLAUDE.md**: Similar functionality, largely interoperable
- Some tools support both formats simultaneously
- Consider maintaining both for maximum compatibility

---

## References

1. AGENTS.md Official Website: https://agents.md
2. doc/prompt-engineering-ai-agents.md - Prompt Engineering guide
3. doc/agents-md-cli-ai-agent-tools.md - CLI tools documentation
4. doc/on-demand-rule-loading.md - On-demand loading patterns
