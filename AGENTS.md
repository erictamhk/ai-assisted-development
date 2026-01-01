# AI-Assisted Development Repository AGENTS.md

**Version:** 1.0  
**Last Updated:** 2026-01-01  
**Framework:** AI-Assisted Development with Domain-Driven Design

---

## Overview

This repository contains knowledge bases, templates, and session logs for building and improving AI-assisted development practices. It serves as both a working repository and a meta-framework for AI agent collaboration.

**Purpose:** This document defines constraints for AI agents working on this repository. It references `dev-knowledge/` for universal patterns and adds repository-specific rules for managing documentation, templates, and session workflows.

---

## Project Context

### Domain Model

This is a meta-project focused on **knowledge management for AI-assisted software development**. It doesn't build a traditional application but rather a framework for organizing, documenting, and improving AI agent workflows.

### Key Entities

| Entity | Description | Key Attributes |
|--------|-------------|----------------|
| Knowledge Base | `dev-knowledge/` - 13 categories of engineering practices | 70 files, ~170K tokens |
| Templates | `template/` - Project templates and prompts | AGENTS-template.md, validators |
| Session Logs | `session-log/` - AI collaboration sessions | Decisions, progress, learnings |
| References | `ref/` - Original research and guidelines | Constitutional docs, engineering guides |

### Bounded Contexts

| Context | Purpose | Boundaries |
|---------|---------|------------|
| Knowledge Engineering | Maintain `dev-knowledge/` | Universal patterns, never project-specific |
| Template Development | Create `template/` for projects | Reusable, project-agnostic |
| Session Documentation | Record in `session-log/` | Chronological, decision-focused |

---

## Technology Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| Documentation | Markdown | Git-managed, universal format |
| AI Agents | Multi-agent framework | See `dev-knowledge/09-ai-development/` |
| Version Control | Git | See `04-coding-style/git-conventions.md` |
| Validation | AI prompts | `template/agents/agent-md-validator.md` |

---

## Architecture

### Repository Structure

```
ai-assisted-development/
├── dev-knowledge/           # Universal knowledge base (13 categories)
├── template/                # Project templates
│   └── agents/             # AGENTS.md templates and validators
├── doc/                     # Original documentation (legacy)
├── ref/                     # Reference guidelines
├── session-log/             # AI collaboration sessions
└── AGENTS.md               # This file (root constraints)
```

### Dependency Rule

**Documentation dependencies always point to `dev-knowledge/` for patterns.**

- Project-specific rules → `AGENTS.md` in project root
- Universal patterns → `dev-knowledge/[category]/`
- Templates → `template/agents/`

---

## Coding Conventions

### Naming Standards

| Pattern | Convention | Example |
|---------|------------|---------|
| AGENTS.md | PascalCase, descriptive | `dev-knowledge/AGENTS.md` |
| Documentation | Kebab-case, descriptive | `clean-architecture.md` |
| Session Logs | Descriptive, date-based | `session-dev-knowledge-001.md` |
| Templates | Descriptive, -template.md suffix | `AGENTS-template.md` |

### File Organization

```
[category]-[topic].md        # Documentation files
session-[context]-[NNN].md   # Session logs
[purpose]-template.md        # Templates
```

### Documentation Standards

- ✅ Use English only (no Chinese/other languages)
- ✅ Reference `dev-knowledge/` for patterns
- ✅ Use generic examples (no company names)
- ✅ Follow `dev-knowledge/04-coding-style/` conventions
- ❌ Avoid proprietary framework-specific code
- ❌ No magic strings/numbers in examples

---

## Testing Requirements

### Documentation Review

| Check | Tool | Command |
|-------|------|---------|
| AGENTS.md validation | AI Prompt | Load `template/agents/agent-md-validator.md` |
| Link validation | Manual | Verify all references exist |
| Format consistency | Manual | Follow existing file patterns |

### Review Process

1. Load validation prompt: `template/agents/agent-md-validator.md`
2. Run through checklist
3. Fix any issues found
4. Commit changes

---

## Error Handling

### Error Types

| Error Type | Pattern | Resolution |
|------------|---------|------------|
| Missing reference | Link points to non-existent file | Create file or fix link |
| Format inconsistency | Deviations from conventions | Align with `dev-knowledge/` |
| Validation failure | AGENTS.md fails checklist | Fix missing sections |

---

## Git Workflow

### Branch Naming

| Type | Pattern | Example |
|------|---------|---------|
| Documentation | `docs/[topic]` | `docs/add-validation-prompt` |
| Template | `template/[name]` | `template/add-agents-validator` |
| Knowledge | `knowledge/[category]` | `knowledge/update-ddd-patterns` |

### Commit Message Format

Follow `dev-knowledge/04-coding-style/git-conventions.md`:

```
[type]([scope]): [subject]

[body - optional]

[footer - optional]
```

**Types:** feat, fix, docs, style, refactor, chore

**Example:**
```
docs: add AGENTS.md validation prompt to template folder

Add agent-md-validator.md with comprehensive checklist for
validating AGENTS.md files against required sections and
anti-patterns.

Refs: #template-001
```

---

## AI Agent Behavior

### Agent Roles

| Agent | Loads | Responsibility |
|-------|-------|----------------|
| @temp | All relevant docs | General assistance, commands |
| @reviewer | `template/agents/agent-md-validator.md` | Validate AGENTS.md files |

### Agent Constraints

AI agents must:

- ✅ Reference `dev-knowledge/` for universal patterns
- ✅ Follow this AGENTS.md for repository rules
- ✅ Use validation prompt when reviewing AGENTS.md files
- ✅ Document decisions in session logs
- ✅ Reference files with full paths: `[dev-knowledge/04-coding-style/git-conventions.md]`
- ✅ Use `template/agents/agent-md-validator.md` for validation

AI agents must NOT:

- ❌ **Never auto git commit** - Always wait for user approval
- ❌ Skip validation when creating/modifying AGENTS.md
- ❌ Introduce project-specific rules in `dev-knowledge/`
- ❌ Use company names in examples
- ❌ Write documentation in non-English languages

### Commands to Remember

**Git Commands:**
```bash
# Check status
git status

# Stage changes
git add [file/folder]

# Commit with conventional message
git commit -m "[type]([scope]): [description]"

# View recent commits
git log --oneline -5

# Check diff
git diff --stat
```

**Validation Commands:**
```bash
# Validate AGENTS.md (via AI agent)
Load: template/agents/agent-md-validator.md
Task: Validate AGENTS.md
```

---

## Code Review Checklist

### Before Submitting Changes

- [ ] AGENTS.md validated with `template/agents/agent-md-validator.md`
- [ ] Commit message follows conventional format
- [ ] No project-specific names in examples
- [ ] All references point to existing files
- [ ] File naming follows conventions
- [ ] Self-review completed

### Reviewer Checks

- [ ] Documentation follows `dev-knowledge/` patterns
- [ ] No proprietary or company-specific content
- [ ] English only in documentation
- [ ] Templates are project-agnostic
- [ ] Session logs document decisions

---

## Definition of Done (DoD)

A change is complete when:

- [ ] Validation passes (`template/agents/agent-md-validator.md`)
- [ ] Commit message follows conventional format
- [ ] No anti-patterns found (company names, non-English text)
- [ ] All references verified
- [ ] Changes committed (with user approval)

---

## Validation Commands

```bash
# Validate AGENTS.md file
Load: template/agents/agent-md-validator.md
Task: Validate the AGENTS.md file

# Check git status
git status

# View commit history
git log --oneline -10
```

---

## References

- **dev-knowledge:** `/dev-knowledge/` - Universal patterns for AI agents
- **Template:** `template/agents/AGENTS-template.md` - Create new AGENTS.md files
- **Validation:** `template/agents/agent-md-validator.md` - Validate AGENTS.md files
- **Git Conventions:** `dev-knowledge/04-coding-style/git-conventions.md` - Commit message standards

---

**Questions?** Check session logs in `session-log/` for context or create a new session.
