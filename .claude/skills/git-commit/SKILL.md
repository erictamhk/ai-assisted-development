---
name: git-commit
description: Ensure proper git commit discipline with pre-approval checkpoints, self-audit trails, and conventional commit validation
license: MIT
compatibility: claude-code, opencode, agents.md
metadata:
  audience: developers, AI agents
  workflow: version control, commit discipline
  version: "1.0"
---

# Git Commit Skill

**Purpose:** Enforce strict git commit discipline with pre-approval checkpoints, self-audit trails, and conventional commit message validation

**What this skill delivers:** Properly staged commits with verified approval, conventional commit messages, and audit trail documentation

**When to use:** Before every git commit action to ensure compliance with commit discipline rules

---

## Three Agent Laws

### Law 1: CLARIFY
> Never assume intent. Always ask for clarification when uncertain.

- If commit scope is unclear → Ask
- If commit message format is unclear → Ask
- If multiple files have unclear changes → Ask

### Law 2: FOLLOW PROCESS
> Always use the defined workflow for your role.

- Run pre-action checkpoint before every commit
- Verify approval status explicitly
- Validate commit message format
- Log self-audit trail
- Never skip validation steps

### Law 3: PROTECT QUALITY
> Never skip rules, checks, or quality gates.

- Never skip pre-approval verification
- Never skip commit message validation
- Never skip self-audit logging
- Never commit without explicit approval

### Law 4: ENFORCE DISCIPLINE
> Hardened discipline prevents rule violations.

**Pre-Action Checkpoint (HARD STOP):**

BEFORE EVERY COMMIT, VERIFY:
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
"[COMMIT] - Requires approval? [Y/N] - Approved? [Y/N]"
```

---

## What I Do

1. **Pre-Action Checkpoint** - Verify approval requirement before any commit action
2. **Commit Message Validation** - Ensure conventional commit format (type(scope): subject)
3. **Self-Audit Trail** - Log approval status before committing
4. **Rule Enforcement** - Never commit without explicit approval
5. **Staging Verification** - Confirm only intended files are staged
6. **Git Status Check** - Verify repository state before commit

---

## When to Use Me

**Use when:**
- Before every git commit action
- When staging changes for commit
- When writing commit messages
- When verifying commit discipline compliance

**Don't use when:**
- Read-only git operations (git status, git log, git diff)
- Non-commit operations (git push, git pull, git checkout)

---

## Commit Workflow

**Standard Pattern:**

```
Human → AGENT → Pre-Action Checkpoint → AGENT → Ask Approval → Human
                                              ↓
                                        YES → Commit
                                              ↓
                                        NO → STOP
                                              ↓
                                        Audit Trail → Complete
```

**Per Commit Workflow:**

1. **Check** → Verify what requires approval
2. **Verify** → Check if approval received
3. **Validate** → Ensure commit message format
4. **Log** → Create self-audit trail
5. **Execute** → Run git add and git commit
6. **Verify** → Confirm commit success

---

## Commit Message Format

**Required Format:**

```
[type]([scope]): [subject]

[optional body explaining WHY]
```

**Valid Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation changes
- `style` - Formatting, missing semicolons, etc.
- `refactor` - Code restructuring
- `test` - Adding or modifying tests
- `chore` - Maintenance tasks

**Common Scopes:**
- `agent`, `logging`, `docs`, `orchestrator`, `template`

**Example:**
```
feat(agent): add researcher skill with progressive disclosure

Add researcher skill in Claude-compatible open standard format
with YAML frontmatter, Three Agent Laws, and research workflow.

Researcher skill includes:
- Web search and code search capabilities
- Pattern documentation with source citation
- Quality gate integration with reviewer pattern
```

---

## Self-Audit Trail Template

```markdown
**Self-Audit Trail:**

[COMMIT] - Requires approval? [Y] - Approved? [Y/N]

**Files Changed:**
- [File 1]
- [File 2]

**Commit Message:**
[type]([scope]): [subject]

**Verification:**
- [ ] Pre-action checkpoint completed
- [ ] Approval received
- [ ] Commit message validated
- [ ] Files verified
```

---

## Knowledge Access

**Always access dev-knowledge/ before acting:**

1. Classify task → Identify commit type
2. grep category → Find relevant patterns
3. read patterns → Understand commit conventions
4. apply → Apply validation rules

**Required Knowledge Files:**
- `04-coding-style/git-conventions.md` - Commit message format and conventions
- `09-ai-development/ai-agent-development-guidelines.md` - Agent discipline rules

**Reference Knowledge Files:**
- `04-coding-style/clean-code.md` - Code organization patterns
- `07-review-checklists/code-review.md` - Review workflow patterns

---

## Constraints

1. Never commit without explicit "yes" approval
2. Never skip the pre-action checkpoint
3. Never use non-conventional commit message format
4. Never commit without logging self-audit trail
5. Never commit without verifying staged files
6. Never commit without checking git status
7. Never commit without validating commit message

---

## Anti-Patterns (Never Do These)

| Anti-Pattern | Instead |
|--------------|---------|
| Skip pre-action checkpoint | Always run Pre-Action Checkpoint |
| Commit without approval | Ask "Ready to commit? (yes/no)" |
| Skip self-audit trail | Log "[COMMIT] - Requires approval? [Y] - Approved? [Y/N]" |
| Use non-conventional format | Follow type(scope): subject format |
| Skip commit message | Always provide meaningful commit message |
| Skip file verification | Check git status before committing |
| Commit directly to main | Follow branch and PR workflow |
| Skip quality gate | Ensure work is REVIEWED before commit |

---

## Examples

### Example 1: Standard Commit with Approval

**Task:** Commit feature changes

**Process:**
1. Run Pre-Action Checkpoint
2. Present files and commit message
3. Ask: "Ready to commit? (yes/no)"
4. Wait for "yes"
5. Log self-audit trail
6. Execute commit

**Result:**
```bash
git add src/agent/researcher.ts
git commit -m "feat(researcher): add research workflow with quality gate

Add research workflow following Three Agent Laws pattern
with classify, search, analyze, document, output steps."
```

**Self-Audit Trail:**
```
[COMMIT] - Requires approval? [Y] - Approved? [Y]

Files Changed:
- src/agent/researcher.ts
- tests/researcher.test.ts

Commit Message:
feat(researcher): add research workflow with quality gate
```

### Example 2: Approval Denied

**Task:** Commit changes without clear message

**Process:**
1. Run Pre-Action Checkpoint
2. Present commit message for review
3. Ask: "Ready to commit? (yes/no)"
4. User says: "No, make message clearer"
5. Improve commit message
6. Re-submit for approval

**Result:** Commit not executed until approval received with proper message.

### Example 3: Validation Failure

**Task:** Commit with invalid format

**Process:**
1. Run Pre-Action Checkpoint
2. Validate commit message
3. Format: "updated file" (invalid)
4. Error: "Invalid commit message format"
5. Auto-correct to: "chore: update file"
6. Re-validate
7. Present for approval

**Result:** Only valid conventional commits proceed to approval.

---

## Related Skills

- **researcher** - Uses git-commit for documenting research findings
- **reviewer** - Quality gate before commits
- **planner** - Task tracking for commit workflows

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-09 | Initial git-commit skill |

---

**Skill Version:** 1.0
**Compatibility:** claude-code, opencode, agents.md
