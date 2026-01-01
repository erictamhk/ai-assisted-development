# AI-Assisted Development Repository AGENTS.md

**Version:** 2.0
**Last Updated:** 2026-01-01
**Framework:** AI-Assisted Development with Domain-Driven Design

---

## Overview

This repository contains knowledge bases, templates, and session logs for building and improving AI-assisted development practices. It serves as both a working repository and a meta-framework for AI agent collaboration.

**Purpose:** This document defines constraints for AI agents working on this repository. It references `dev-knowledge/` for universal patterns and adds repository-specific rules.

---

## Core Rules 🔴

**Critical rules that must never be violated.**

### Git Operations

| Symbol | Rule | Description |
|--------|------|-------------|
| 🔴 NEVER | Auto-commit | Always ask for user approval before committing |
| ⚠️ MUST | Commit format | Follow conventional commits (see git-conventions.md) |

### Git Push

| Symbol | Rule | Description |
|--------|------|-------------|
| 💡 TIP | Recommend push | I can suggest git push after commit |
| 🔴 NEVER | Execute push | I will never run git push for you |

### Documentation Rules

| Symbol | Rule | Description |
|--------|------|-------------|
| 🔴 NEVER | AGENTS.md validation | Always validate when creating/modifying AGENTS.md files |
| 🔴 NEVER | Non-English docs | Use English only in all documentation |
| 🔴 NEVER | Company names | Use generic examples, no company-specific content |
| ⚠️ MUST | Pattern references | Reference `dev-knowledge/` for universal patterns |

### General Behavior

| Symbol | Rule | Description |
|--------|------|-------------|
| 🔴 NEVER | Project-specific rules | Never add project-specific rules to `dev-knowledge/` |
| ⚠️ MUST | Session logging | Document decisions in session logs |
| ✅ SHOULD | Full paths | Reference files with full paths: `dev-knowledge/04-coding-style/git-conventions.md` |

### Rule Priority System

| Symbol | Meaning | Violation Consequence |
|--------|---------|----------------------|
| 🔴 NEVER | Absolute prohibition | Workflow halts immediately |
| ⚠️ MUST | Required action | Results considered invalid |
| ✅ SHOULD | Strong recommendation | Suboptimal but not blocking |

---

## Commands ⚡

**Predefined workflows. Always follow these patterns exactly.**

### Git Commit Workflow

Git commit is a command YOU use. I only help you execute it manually.

1. Run `git status` and `git diff`
2. Summarize changes for you
3. Ask: "Ready to commit? (yes/no)"
4. Wait for your explicit "yes"
5. Run: `git commit -m "[type]([scope]): [subject]"` (I type it, you use it)
6. Verify with `git status`

**I will NEVER commit without your approval.**

### AGENTS.md Creation/Modification Workflow

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
| Version Control | Git | See `dev-knowledge/04-coding-style/git-conventions.md` |
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

- ✅ Use English only
- ✅ Reference `dev-knowledge/` for patterns
- ✅ Use generic examples (no company names)
- ✅ Follow `dev-knowledge/04-coding-style/` conventions
- ❌ Avoid proprietary framework-specific code
- ❌ No magic strings/numbers in examples

---

## Git Workflow

**Summary only. Full rules in `dev-knowledge/04-coding-style/git-conventions.md`.**

### Commit Message Format

```
[type]([scope]): [subject]
```

**Valid types:** feat, fix, docs, style, refactor, chore

### Key Rules

- 🔴 NEVER commit without user approval
- ⚠️ MUST use conventional commit format
- ✅ SHOULD include ticket/reference number in footer

### Common Commands

```bash
# Check status
git status

# View recent commits
git log --oneline -5

# Check diff
git diff --stat
```

---

## AI Agent Behavior

### Agent Roles

| Agent | Loads | Responsibility |
|-------|-------|----------------|
| @temp | All relevant docs | General assistance, commands |
| @reviewer | `template/agents/agent-md-validator.md` | Validate AGENTS.md files |

### Standard Workflow

1. Read relevant docs (AGENTS.md, dev-knowledge/)
2. Identify applicable rules from Core Rules section
3. Plan approach using defined Commands workflows
4. Execute with explicit user confirmations
5. Verify against rules before proceeding

### Agent Constraints

AI agents must:

- ✅ Reference `dev-knowledge/` for universal patterns
- ✅ Follow Core Rules with priority symbols
- ✅ Use Commands section for defined workflows
- ✅ Document decisions in session logs
- ✅ Reference files with full paths

AI agents must NOT:

- ❌ Auto-commit without approval
- ❌ Skip AGENTS.md validation
- ❌ Introduce project-specific rules in `dev-knowledge/`
- ❌ Use company names in examples
- ❌ Write documentation in non-English languages

---

## Code Review Checklist

### Before Submitting Changes

- [ ] Git operations followed Commands workflow
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

- [ ] Git workflow completed with user approvals
- [ ] Commit message follows conventional format
- [ ] No anti-patterns found (company names, non-English text)
- [ ] All references verified
- [ ] Changes committed (with user approval)

---

## References

- **dev-knowledge:** `/dev-knowledge/` - Universal patterns for AI agents
- **Template:** `template/agents/AGENTS-template.md` - Create new AGENTS.md files
- **Validation:** `template/agents/agent-md-validator.md` - Validate AGENTS.md files
- **Git Conventions:** `dev-knowledge/04-coding-style/git-conventions.md` - Full git rules
- **Coding Style:** `dev-knowledge/04-coding-style/` - All coding conventions

---

**Questions?** Check session logs in `session-log/` for context or create a new session.
