# Template Folder

This folder contains templates for AI-assisted development.

---

## Contents

| Path | Description |
|------|-------------|
| `agents/AGENTS-template.md` | Template for creating project-specific AGENTS.md files |
| `agents/agent-md-validator.md` | Validation prompt for AI agents to validate AGENTS.md files |

---

## Quick Start

### Create AGENTS.md from Template

```bash
# Copy template to your project
cp template/agents/AGENTS-template.md ./AGENTS.md

# Edit with your project specifics
nano AGENTS.md
```

---

## AI Agent Validation

AI agents should load `template/agents/agent-md-validator.md` when validating AGENTS.md files:

```
Load: template/agents/agent-md-validator.md
Task: Validate the project's AGENTS.md file against the checklist
```

---

## Template Sections

| Section | Required | Description |
|---------|----------|-------------|
| Overview | Yes | Project description and purpose |
| Project Context | Yes | Domain model, entities, bounded contexts |
| Technology Stack | Yes | Languages, frameworks, databases |
| Architecture | Yes | Layer structure, dependency rules |
| Coding Conventions | Yes | Naming, file organization, standards |
| Testing Requirements | Yes | TDD, coverage, test types |
| Error Handling | Yes | Error types, response format |
| Git Workflow | Yes | Branch naming, commit messages |
| AI Agent Behavior | Yes | Constraints for AI agents |
| Code Review Checklist | Yes | PR review requirements |
| Definition of Done | Yes | When is work complete? |
| Validation Commands | Yes | How to validate code |

---

## Best Practices

### 1. Keep it Updated

- Update AGENTS.md when technology stack changes
- Review quarterly for accuracy
- Version control changes

### 2. Link to dev-knowledge

Always reference `dev-knowledge/` for universal patterns:

```markdown
For coding standards, see [dev-knowledge/04-coding-style/](../dev-knowledge/04-coding-style/)
```

### 3. Be Specific

Avoid vague statements:

```markdown
# ❌ Bad
We follow best practices.

# ✅ Good
We use TypeScript strict mode (see dev-knowledge/04-coding-style/typescript-conventions.md)
```

### 4. Include Validation Commands

Make it easy to validate:

```markdown
## Validation Commands

```bash
npm test && npm run lint && npm run typecheck
```
```

---

## Files

```
template/
├── agents/
│   ├── AGENTS-template.md     # Template for AGENTS.md
│   └── agent-md-validator.md  # Validation prompt for AI agents
└── README.md                  # This file
```
