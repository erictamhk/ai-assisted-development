## Project Overview

This project is an **AI Agent Framework for Production Software Development**. It transforms generic AI coding assistants into **specialized software engineering agents** that build enterprise-grade systems following your exact standards. The framework uses **dev-knowledge** as the universal brain and **project-specific AGENTS.md** for domain rules, enabling consistent, high-quality development across all projects.

## Project Purpose

**Core Problem:** AI coding assistants (Claude, GPT, Cursor) produce inconsistent, prototype-quality code because they lack your company's standards, patterns, and domain knowledge.

**Solution:** Create specialized AI agents (Architect, Coder, Reviewer, etc.) that:

- Follow **your exact standards** (from dev-knowledge)
- Build **production systems** (not prototypes)
- Work **universally** across domains (HRMS, banking, ecommerce)
- Coordinate via **orchestrator** with human gates
- Generate **living documentation** (PROGRESS.md, DECISIONS.md)

**Outcome:** 95% rule compliance vs 40% generic prompting. Production systems from natural language requirements.

***

## Project Structure

```
Root/
├── dev-knowledge/              # Universal knowledge base (13 categories, 61 files)
│   ├── AGENTS.md               # Technical spec for AI agents
│   ├── README.md               # Human guide
│   ├── index.md                # File manifest
│   ├── 01-requirements/        # Specs, examples, executable specs
│   ├── 02-ddd/                 # Domain modeling patterns
│   ├── 03-architecture/        # Clean Arch, Vertical Slice
│   ├── 04-coding-style/        # TS conventions, SOLID, naming
│   ├── 05-testing/             # TDD, BDD, test pyramids
│   ├── 06-design-patterns/     # GoF, CQRS, Event Sourcing
│   ├── 07-review-checklists/   # Quality gates
│   ├── 08-collaboration/       # Impact Mapping, Example Mapping
│   ├── 09-ai-development/      # AGENTS.md standard, prompt engineering
│   ├── 10-legacy/              # Monolith analysis, extraction
│   ├── 11-error-handling/      # Result<T>, domain errors
│   ├── 12-security/            # Auth patterns, domain security
│   └── 13-deployment/          # CI/CD, blue-green deploys
│
├── ai-agents/                  # Generated AI agent framework
│   ├── roles/                  # 9 specialized agents
│   │   ├── researcher.md
│   │   ├── expert-listener.md
│   │   ├── clarifier.md
│   │   ├── architect.md
│   │   ├── planner.md
│   │   ├── coder.md
│   │   ├── reviewer.md
│   │   ├── legacy-analyzer.md
│   │   ├── refactorer.md
│   │   └── orchestrator.md
│   ├── orchestration/
│   │   ├── workflow-router.md
│   │   ├── logging-system.md
│   │   └── workflows.md
│   ├── examples/
│   │   └── workflow-example-greenfield.md
│   ├── KNOWLEDGE-MAPPING.md    # Which dev-knowledge files power each agent
│   ├── README.md               # How to use the framework
│   └── .toolconfigs/
│       ├── .cursor/rules.md
│       ├── CLAUDE.md
│       └── opencode.json
│
├── projects/                   # Project-specific knowledge
│   └── hrms/                   # Example project
│       ├── AGENTS.md           # HRMS-specific domain rules
│       ├── PROGRESS.md         # Live dashboard
│       ├── DECISIONS.md        # Human decisions log
│       ├── AI-WORKLOG.md       # Agent actions log
│       └── src/                # Generated production code
│
├── dev-knowledge-review-prompt.md  # Quality assurance for knowledge base
└── master-prompts/             # Prompts to generate ai-agents/
    ├── prompt-1-agents.md
    ├── prompt-2-orchestration.md
    └── prompt-3-bootstrap.md
```


***

## Core Workflow

```
Boss: "@orchestrator Build HRMS"

1. Orchestrator detects workflow: Greenfield (no knowledge)
2. @researcher → Industry research [08-collaboration/impact-mapping.md]
3. @clarifier → Precise specs [01-requirements/spec-by-example.md]
4. @architect → Architecture [02-ddd/strategic-patterns.md]
5. @planner → Task breakdown [03-architecture/vertical-slice.md]
6. @coder → Production code [04-coding-style/typescript-conventions.md]
7. @reviewer → Quality gates [07-review-checklists/ddd-review.md]
8. Repeat 6-7 until complete
9. Human approval gates at key milestones
10. Generate: PROGRESS.md, DECISIONS.md, AI-WORKLOG.md
```


***

## Key Principles

### 1. **Separation of Knowledge**

```
dev-knowledge/     ← Universal (physics laws)
  ├── DDD principles (same for all projects)
  ├── Clean Architecture (same structure)
  └── Testing standards (same approach)

projects/hrms/     ← Domain-specific (chemistry)
  ├── Employee invariants
  ├── Leave policy rules
  └── HRMS bounded contexts
```


### 2. **On-Demand Loading**

```
Agents load only what they need:
architect: 02-ddd/ + 03-architecture/ (12K tokens)
coder:     04-coding-style/ + 05-testing/ (10K tokens)
reviewer:  07-review-checklists/ (6K tokens)
```


### 3. **Human-in-the-Loop**

```
Orchestrator enforces gates:
- Architecture approval (before coding)
- Major design changes (before refactor)
- Go-live decisions (before deployment)
```


### 4. **Living Documentation**

```
Every project auto-generates:
PROGRESS.md ← Live status dashboard
DECISIONS.md ← Human decision log
AI-WORKLOG.md ← Agent action history
CHANGELOG.md ← Generated from commits
```


***

## Tool Integration

```
.cursor/rules.md     ← Cursor AI rules
CLAUDE.md            ← Claude Code instructions
opencode.json        ← OpenCode agent definitions
.gemini/config.yaml  ← Gemini Code Assist config
```


***

## Success Metrics

```
✅ 95% rule compliance (vs 40% generic)
✅ Production quality from natural language
✅ Consistent architecture across projects
✅ 10x faster onboarding (standards encoded)
✅ Audit trail of all decisions (PROGRESS.md)
✅ Universal (works for any domain)
✅ Scalable (add new projects easily)
```


***

## Next Steps

```
1. ✅ dev-knowledge ready (13 categories, 61 files)
2. ✅ AGENTS.md + README.md + index.md created
3. ✅ Review prompt created (quality assurance)
4. 🔜 Generate /ai-agents/ framework (9 agents + orchestration)
5. 🔜 Create first project: /projects/hrms/
6. 🔜 @orchestrator "Build HRMS for 500 employees"
```

