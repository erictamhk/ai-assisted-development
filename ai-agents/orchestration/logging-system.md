# Logging System

The logging system provides complete audit trail for AI-assisted development. It consists of three log files managed by the orchestrator.

## Files

| File | Purpose | Managed By |
|------|---------|------------|
| `PROGRESS.md` | Live status dashboard | Orchestrator |
| `DECISIONS.md` | Human decisions log | Orchestrator |
| `AI-WORKLOG.md` | Agent actions log | Each agent |

---

## 1. PROGRESS.md

**Purpose:** Real-time status dashboard showing project progress, metrics, and next actions.

**Updated By:** Orchestrator

**Update Triggers:**
- Hourly interval
- After each domain completion
- After boss checkpoint

**Format:**

```markdown
# [Project Name] PROGRESS | [Timestamp]

## STATUS: [X]% COMPLETE

### Domains
| Domain | Status | Tests |
|--------|--------|-------|
| [Name] | [✅ DONE/🔄 IN PROGRESS/⏳ PENDING] | [XX%] |

### Metrics
- Code: [XXX] LOC | Tests: [XXX] LOC
- Coverage: [XX]% | Linting: [X] errors
- Token usage: [XX]K avg

### Blockers
- [None or description]

### Next Action
- [Description]
```

---

## 2. DECISIONS.md

**Purpose:** Audit trail of all human decisions with timestamp, context, and impact.

**Updated By:** Orchestrator (when boss makes a decision)

**Update Triggers:**
- Boss answers clarification questions
- Boss approves/rejects architecture
- Boss changes scope
- Boss requests review
- Boss approves deployment

**Format:**

```markdown
# DECISIONS

## [DECISION #XXX] | [Timestamp] | [Type]

**Context:** [Brief description]

**Decision:**
- [Boss's input or directive]

**Impact:**
- [What changes as a result]

**Status:** [APPROVED/PENDING/IMPLEMENTED]

---
```

**Decision Types:**
- `Boss Directive` - Initial requirements
- `Architecture Approval` - Design sign-off
- `Scope Change` - Mid-project changes
- `Review Request` - Boss wants to review
- `Deployment Approval` - Go-live decision

---

## 3. AI-WORKLOG.md

**Purpose:** Complete trace of every AI agent action with inputs, outputs, and metrics.

**Updated By:** Each agent (via orchestrator)

**Update Triggers:**
- Every agent action completion
- Every tool execution
- Every file creation/modification

**Format:**

```markdown
# AI WORKLOG

## [Timestamp] | [AGENT NAME] | [Action Description]

**Input:** [What the agent received]
**Output:** [What the agent produced]
**Metrics:**
- Token usage: [XX]K
- Duration: [Xm Xs]
- Files: [list]

---

```

---

## Update Flow

```
Agent completes action
        ↓
Orchestrator logs to AI-WORKLOG.md
        ↓
If significant change → Update DECISIONS.md
        ↓
If hourly/domain milestone → Update PROGRESS.md
        ↓
Notify boss if required
```

---

## Boss Intervention Points

Logged in DECISIONS.md with options:
- `CONTINUE` - Proceed automatically
- `REVIEW NOW` - Live demo session
- `CHANGE SCOPE` - Modify requirements
- `APPROVE & DEPLOY` - Go to next environment
- `STOP` - Halt workflow

---

## File Locations

| Project Type | Location |
|--------------|----------|
| Any Project | `projects/[name]/PROGRESS.md` |
| Any Project | `projects/[name]/DECISIONS.md` |
| Any Project | `projects/[name]/AI-WORKLOG.md` |
