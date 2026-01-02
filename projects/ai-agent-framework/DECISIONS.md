# DECISIONS

All human decisions are logged here with timestamp, context, and impact.

---

## [DECISION #001] | 2026-01-01 21:00:00 | Boss Directive

**Context:** Project kickoff - defining the goal of building AI Agent Framework

**Decision:**
- Focus on logging system first (PROGRESS.md, DECISIONS.md, AI-WORKLOG.md)
- No example project (HRMS) for now
- Focus on OpenCode integration first

**Impact:**
- Logging system created as foundation
- Project structure established

**Status:** APPROVED → Started implementation

---

## [DECISION #002] | 2026-01-01 22:00:00 | Boss Directive

**Context:** Adding 3 new commands to AGENTS.md and creating project tracking

**Decision:**
- Add Log Decision Command to AGENTS.md
- Add Log AI Worklog Command to AGENTS.md
- Add Update Progress Command to AGENTS.md
- Create ai-agent-framework project for tracking

**Impact:**
- Clear logging workflows defined
- Project progress can be tracked

**Status:** APPROVED → Implementation in progress

---

## [DECISION #003] | 2026-01-01 22:30:00 | Boss Directive

**Context:** AI auto-committed without approval, violating workflow rules

**Decision:**
- Add critical rule "Never assume intent, always clarify" to AGENTS.md
- Add Strict Executor persona to all AGENTS.md files
- Update root AGENTS.md and project template

**Impact:**
- All agents must ask before acting
- No more auto-commits
- Clear workflow for approvals

**Status:** APPROVED → Implemented

---

## [DECISION #005] | 2026-01-02 01:30:00 | Architecture Approval

**Context:** Defining the two-mode agent workflow and quality gate pattern

**Decision:**
- Simple role agent workflow: Input query → role agent → output work
- Orchestrator only dispatches agents (no quality judgment)
- Quality gate pattern: Human → Orchestrator → AGENT → Orchestrator → REVIEWER → Orchestrator → Human
- REVIEWER catches issues, agent redoes if BAD
- Human only sees work after REVIEWER says GOOD
- REVIEWER uses different POV to catch hallucinations/gaps
- Workflow phases:
  - Phase 1: Build agents, human direct review first
  - Phase 2: Integrate orchestrator dispatcher
  - Phase 3: Orchestrator grows, human approves final only

**Impact:**
- Clear separation: Orchestrator = dispatcher, REVIEWER = quality gate, Human = final approval
- All agents output to Orchestrator (goes to REVIEWER)
- REVIEWER reports GOOD/BAD to Orchestrator
- Orchestrator delivers feedback to agent or human based on review result
- Human never sees BAD work

**Status:** APPROVED → Update all templates and mappings
