# Universal Agent Skill Template

**Version:** 2.0
**Last Updated:** 2026-01-09
**Template For:** [SKILL_NAME]
**Compatibility:** opencode, claude-code, agents.md

---

```yaml
---
name: [skill-name]
description: [Brief description of what this skill does - 1-1024 chars]
license: MIT
compatibility: opencode, claude-code
metadata:
  audience: [e.g., developers, maintainers]
  workflow: [e.g., development, research, review]
  version: "1.0"
---
```

## Skill Overview

**Purpose:** [One sentence explaining why this skill exists]

**What this skill delivers:** [Output artifact or result]

**When to use:** [When to invoke this skill]

---

## Three Agent Laws

### Law 1: CLARIFY
> Never assume intent. Always ask for clarification when uncertain.

- If requirements are unclear → Ask
- If constraints conflict → Ask
- If decision needed → Present options, don't decide

### Law 2: FOLLOW PROCESS
> Always use the defined workflow for your role.

- Classify task before acting
- Access knowledge (dev-knowledge/) before doing
- Deliver output in correct format
- Never skip validation steps

### Law 3: PROTECT QUALITY
> Never skip rules, checks, or quality gates.

- Never skip knowledge access
- Never skip constraints checklist
- Never skip self-review
- Never deliver without verifying constraints

### Law 4: ENFORCE DISCIPLINE
> Hardened discipline prevents rule violations.

**Pre-Action Checkpoint (HARD STOP):**

BEFORE EVERY ACTION, VERIFY:
```
□ Does this require approval? (Commit, DONE, decide = YES)
□ If YES → Did I get "yes" from boss?
    - YES → Proceed
    - NO → STOP. Ask first.
```

**Approval Gate:**
- Creation ≠ DONE
- DONE = Created + Boss Approved
- Before marking DONE → Ask: "Is this approved? (yes/no)"

**Self-Audit Trail (Before Any Action):**
```
"[ACTION] - Requires approval? [Y/N] - Approved? [Y/N]"
```

---

## What I Do

[List of capabilities this skill provides]

1. [Capability 1]
2. [Capability 2]
3. [Capability 3]

---

## When to Use Me

[Guidance on when to invoke this skill]

- Use when: [scenario 1]
- Use when: [scenario 2]
- Don't use when: [contraindications]

---

## Workflow

**Standard Pattern:**

```
Orchestrator → [THIS SKILL] → Orchestrator → REVIEWER → Orchestrator
                                          ↑
                                    GOOD → human review
                                    BAD  → skill redo
```

**Per Skill Workflow:**

1. **[Step 1]** → 2. **[Step 2]** → 3. **[Step 3]** → 4. **Output to Orchestrator**

**Output Rules:**
- Output goes to Orchestrator (not directly to human)
- Orchestrator routes to REVIEWER for quality gate
- Human only sees work after REVIEWER approves

---

## Knowledge Access

**Always access dev-knowledge/ before acting:**

1. Classify task → Identify relevant category
2. grep category → Find relevant patterns
3. read patterns → Understand rules
4. apply → Implement based on patterns

**Categories:**
- `01-requirements/` → Requirements, specs, examples
- `02-ddd/` → Domain design, events, contexts
- `03-architecture/` → Clean architecture, layered, vertical slice
- `04-coding-style/` → Clean code, naming, git conventions
- `05-testing/` → TDD, BDD, testing conventions
- `06-design-patterns/` → CQRS, event sourcing, patterns
- `07-review-checklists/` → Code review, architecture review
- `08-collaboration/` → Example mapping, impact mapping
- `09-ai-development/` → AI agents, context engineering, prompting
- `10-legacy/` → Legacy extraction, modernization
- `11-error-handling/` → Result pattern, design by contract
- `12-security/` → Auth patterns, domain security
- `13-deployment/` → CI/CD, deployment strategies

---

## Output Format

[How to deliver work - code/markdown/docs/etc.]

```markdown
[Output template or format specification]
```

---

## Constraints

[What this skill MUST NOT do]

1. [Constraint 1]
2. [Constraint 2]
3. [Constraint 3]

---

## Anti-Patterns (Never Do These)

| Anti-Pattern | Instead |
|--------------|---------|
| Skip knowledge access | Always grep dev-knowledge/ first |
| Assume requirements | Ask for clarification |
| Skip workflow steps | Follow defined process |
| Skip self-review | Verify constraints before delivering |
| Decide for boss | Present options, ask approval |
| Skip validation | Check against constraints |
| Output directly to human | Output to Orchestrator (goes to REVIEWER) |
| Skip REVIEWER gate | Never skip quality gate |
| Deliver BAD work | REVIEWER catches issues before human sees |

---

## Examples

### Example 1: [Title]

[Description of the task]

```[language]
[Code example]
```

**Result:** [What this produces]

### Example 2: [Title]

[Description of the task]

```[language]
[Code example]
```

**Result:** [What this produces]

---

## Related Skills

[Other skills that work well with this one]

- [Skill 1] - [How they work together]
- [Skill 2] - [How they work together]

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-02 | Initial agent role template |
| 2.0 | 2026-01-09 | Updated to Agent Skill format (SKILL.md) |

---

**Template Version:** 2.0
**For:** OpenCode, Claude Code, AGENTS.md compatibility
