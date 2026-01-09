# Agent Skills Best Practices

**Version:** 1.0
**Last Updated:** 2026-01-09
**Sources:** Anthropic Engineering, Claude Docs, OpenCode, AGENTS.md specification

---

## Overview

Agent Skills are organized folders of instructions, scripts, and resources that AI agents can discover and load dynamically to perform specialized tasks. Introduced by Anthropic in October 2025, Skills have become an open standard ([agentskills.io](https://agentskills.io)) adopted across Claude Code, Claude.ai, OpenAI Codex, Google Jules, OpenCode, and 20+ AI coding agents.

This document consolidates best practices for designing, implementing, and maintaining Agent Skills for our AI-assisted development framework.

---

## 1. Skill Anatomy

### 1.1 Core Structure

```
skill-name/
├── SKILL.md          # Required: Main documentation
├── reference.md      # Optional: Additional reference material
├── examples.txt      # Optional: Usage examples
├── script.py         # Optional: Executable code
└── config.json       # Optional: Configuration
```

### 1.2 SKILL.md Required Format

Every SKILL.md must start with YAML frontmatter:

```markdown
---
name: skill-name
description: Brief description of what this skill does
---

# Skill Name

## Purpose
Detailed explanation...

## Usage
How to use this skill...

## Examples
...
```

### 1.3 Progressive Disclosure

Skills implement a 3-level progressive disclosure pattern:

| Level | Content | When Loaded |
|-------|---------|-------------|
| **Level 1** | name + description (100 tokens) | Always in system prompt |
| **Level 2** | Full SKILL.md | When skill is triggered |
| **Level 3+** | Additional files | As needed by context |

This design keeps context efficient while providing rich functionality when required.

---

## 2. Best Practices

### 2.1 Naming and Description

**Best Practice:** Clear, actionable naming with specific descriptions.

```markdown
---
name: code-reviewer
description: Analyzes code for bugs, security issues, and maintainability concerns
---
```

**Anti-pattern:**
```markdown
---
name: reviewer
description: Helps with code
---
```

### 2.2 Progressive Disclosure Structure

**Best Practice:** Keep SKILL.md focused, use additional files for depth.

```markdown
# PDF Skill

## Overview
Extract and manipulate PDF documents.

## Quick Reference
- Use `extract_text.py` for text extraction
- Use `fill_form.py` for form filling

## Form Filling
See [forms.md](./forms.md) for detailed form-filling instructions.
```

### 2.3 Code Inclusion

Skills can include executable code for:
- Deterministic operations (math, parsing)
- Local processing (sensitive data stays local)
- Tool integration (existing scripts)

**Example:**
```markdown
## Usage

Run the extraction script:
```bash
python extract.py <input-file>
```

The script returns structured results without loading data into context.
```

### 2.4 Single Responsibility

**Best Practice:** One skill, one purpose.

```
✅ good-code-analyzer
   └── Detects code smells and suggests improvements

❌ code-analyzer-and-refactorer-and-tester
   └── Too broad, split into multiple skills
```

---

## 3. Skill Development Workflow

### 3.1 Start with Evaluation

1. Run agents on representative tasks
2. Identify capability gaps
3. Build skills incrementally to address shortcomings

### 3.2 Structure for Scale

When SKILL.md becomes unwieldy:
- Split into separate files
- Reference them from main SKILL.md
- Keep mutually exclusive contexts separate

### 3.3 Think from Claude's Perspective

- Monitor skill usage in real scenarios
- Iterate based on observations
- Pay attention to `name` and `description` triggers
- Watch for unexpected trajectories or overreliance

### 3.4 Iterate with Claude

1. Work on a task with Claude
2. Ask Claude to capture successful approaches
3. Ask for self-reflection when things go wrong
4. Codify patterns into reusable skills

---

## 4. Security Considerations

### 4.1 Trust Requirements

- Install skills only from trusted sources
- Audit skills from less-trusted sources before use
- Review all bundled files, dependencies, and scripts

### 4.2 Network Access

Pay attention to instructions that:
- Connect to external network sources
- Process untrusted data
- Execute third-party code

### 4.3 Code Execution

Skills with code can:
- Run locally on your machine
- Access file systems and environment
- Execute shell commands

**Only install skills you trust to run code on your system.**

---

## 5. Cross-Platform Compatibility

### 5.1 Supported Platforms

Agent Skills work across:

| Platform | Support |
|----------|---------|
| Claude Code | Full |
| Claude.ai (web) | Full |
| Claude mobile apps | Full |
| Claude API | Full |
| OpenCode | Via AGENTS.md |
| OpenAI Codex | Via AGENTS.md |
| Google Jules | Via AGENTS.md |
| Zed | Via AGENTS.md |
| VS Code | Via AGENTS.md |
| Cursor | Via AGENTS.md |
| Windsurf | Via AGENTS.md |
| Devin | Via AGENTS.md |

### 5.2 AGENTS.md Integration

For maximum portability, skills can also be expressed as AGENTS.md files:

```markdown
# AGENTS.md

## Setup
- Install deps: `pnpm install`

## Code Style
- TypeScript strict mode
- Single quotes, no semicolons

## Testing
- Run tests: `pnpm test`
```

AGENTS.md is supported by 60k+ open-source projects and stewarded by the Agentic AI Foundation (Linux Foundation).

---

## 6. Skill Categories for AI-Assisted Development

Based on our framework requirements, skills should cover:

### 6.1 Research & Discovery
- **researcher**: Industry research, documentation search
- **expert-listener**: Domain expert interview patterns
- **clarifier**: Requirements gathering, question formulation

### 6.2 Design & Planning
- **architect**: System design, patterns, trade-offs
- **planner**: Task breakdown, roadmap creation

### 6.3 Implementation
- **coder**: Code generation, refactoring
- **legacy-analyzer**: Code understanding, documentation
- **refactorer**: Code improvement, modernization

### 6.4 Quality Assurance
- **reviewer**: Code review, quality gates

### 6.5 Orchestration
- **orchestrator**: Workflow management, agent dispatch

---

## 7. Anti-Patterns to Avoid

### 7.1 Overly Broad Skills
❌ "helps with coding"
✅ "generates TypeScript React components with hooks"

### 7.2 Missing Progressive Disclosure
❌ SKILL.md with 2000+ tokens of context
✅ Focused SKILL.md + referenced additional files

### 7.3 Vague Descriptions
❌ "analyzes things"
✅ "detects security vulnerabilities in JavaScript code"

### 7.4 Ignoring Claude's Perspective
- Not monitoring skill usage
- Not iterating based on observations
- Overloading context without need

### 7.5 Skipping Security Review
- Installing from untrusted sources
- Not auditing bundled code
- Ignoring network access patterns

---

## 8. References

### Official Documentation
- [Agent Skills Overview](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Agent Skills Announcement](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

### Open Standards
- [AGENTS.md Specification](https://agents.md/)
- [Agent Skills (agentskills.io)](https://agentskills.io)

### Community Resources
- [Claude Skills Cookbook](https://github.com/anthropics/claude-cookbooks/tree/main/skills)
- [Anthropics Skills Repository](https://github.com/anthropics/skills)

### Related Tools
- [OpenCode](https://opencode.ai)
- [Model Context Protocol](https://modelcontextprotocol.io)

---

## 9. Quick Reference

### Skill Checklist

- [ ] YAML frontmatter with name and description
- [ ] Clear, focused purpose
- [ ] Progressive disclosure structure
- [ ] Usage examples included
- [ ] Code (if needed) is secure and audited
- [ ] Tested with representative tasks
- [ ] Name and description trigger correctly

### Development Tips

1. **Start small**: Begin with minimal SKILL.md, expand as needed
2. **Test iteratively**: Run skills on real tasks, observe behavior
3. **Iterate with Claude**: Ask Claude to help improve skills
4. **Monitor triggers**: Ensure name/description work as expected
5. **Document patterns**: Capture successful approaches

---

**End of Document**
