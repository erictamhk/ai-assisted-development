# dev-knowledge AGENTS.md

**Universal AI Agent Knowledge Base for Software Development**

Version: 1.0  
Last Updated: 2026-01-01  
Framework: AI-Assisted Development with Domain-Driven Design

---

## Overview

This document defines the `dev-knowledge` folder as the **universal knowledge base** for all AI agents in your development framework. It contains 13 categories of curated software engineering practices, patterns, and conventions that guide AI behavior across all projects.

**Purpose:** Enable AI agents to build production-grade software by providing domain-specific rules, patterns, and constraints instead of relying on generic LLM knowledge.

---

## Knowledge Organization (13 Categories)

### 01-requirements: Capture What to Build
```
├── executable-specs.md         # Requirements → Automated tests (living docs)
├── problem-frames.md           # Michael Jackson's requirement analysis framework
├── requirements-and-specification.md  # Effective requirements documents
├── spec-by-example.md          # Concrete examples + Gherkin syntax
└── specification-driven-development.md # Specs drive implementation
```
**Used by:** Clarifier, Expert-Listener, Planner  
**Key principle:** Specifications are executable and testable, not prose documents.

---

### 02-ddd: Model Complex Domains
```
├── README.md                   # DDD overview + quick reference
├── strategic-patterns.md       # Bounded contexts, ubiquitous language
├── tactical-patterns.md        # Aggregates, value objects, entities, events
├── context-mapping.md          # Inter-context relationships
├── domain-event-storming.md    # Workshop technique for domain exploration
└── domain-storytelling.md      # Narrative understanding of domain flows
```
**Used by:** Architect, Expert-Listener, Coder, Reviewer  
**Key principle:** Domain model is the center of architecture; technology serves the domain.

---

### 03-architecture: Structure Systems
```
├── README.md                   # Architecture overview
├── clean-architecture.md       # Four-layer separation of concerns
├── layered-architecture.md     # Traditional layers (Presentation, App, Domain, Infra)
├── vertical-slice.md           # Feature-based organization
└── refactoring-journey.md      # 14-step monolith → clean architecture path
```
**Used by:** Architect, Planner, Coder  
**Key principle:** Architecture should be independent of frameworks; business logic isolated.

---

### 04-coding-style: Write Maintainable Code
```
├── clean-code.md               # SOLID principles + maintainable code patterns
├── typescript-conventions.md   # TS-specific: strict mode, types, async
├── naming-conventions.md       # Entity, VO, UseCase, Service, DTO naming
├── solid-principles.md         # SRP, OCP, LSP, ISP, DIP detailed guide
├── file-organization.md        # Feature-based directory structure
├── git-conventions.md          # Commit messages, branching, workflow
└── README.md                   # Coding style overview + quick reference
```
**Used by:** Coder, Reviewer, Planner  
**Key principle:** Code is read more than written; optimize for clarity and domain expression.

---

### 05-testing: Ensure Quality
```
├── README.md                   # Testing overview + quick reference
├── tdd-workflow.md             # Red-Green-Refactor methodology
├── bdd-gherkin.md              # Behavior-driven specs in Gherkin
├── test-pyramids.md            # Unit/Integration/E2E balance
└── testing-conventions.md      # AAA pattern, test doubles, coverage targets
```
**Used by:** Coder, Reviewer, Planner  
**Key principle:** Tests are specifications; they document behavior and enable refactoring.

---

### 06-design-patterns: Reusable Solutions
```
├── README.md                   # Pattern selection guide
├── gof-catalog.md              # 23 Gang of Four patterns
├── clean-architecture-patterns.md # Ports/Adapters, Use Case, Repository
├── cqrs-event-sourcing.md      # CQRS + Event Sourcing integration
├── event-sourcing.md           # Event-based state management
└── pattern-language.md         # Christopher Alexander's pattern relationships
```
**Used by:** Architect, Coder, Reviewer  
**Key principle:** Use patterns to solve recurring design problems; don't over-pattern.

---

### 07-review-checklists: Validate Quality
```
├── architecture-review.md      # Design pattern, layer, dependency checks
├── code-review.md              # Code quality, SOLID, naming, security
├── ddd-review.md               # Aggregate cohesion, event consistency
└── ai-code-review.md           # AI-specific review considerations
```
**Used by:** Reviewer  
**Key principle:** Reviews are conversations about design quality, not nitpicking style.

---

### 08-collaboration: Align Teams
```
├── impact-mapping.md           # Business goals → deliverables
├── example-mapping.md          # Rules → concrete examples (BDD discovery)
├── living-documentation.md     # Executable specs as docs
└── team-collaboration-guide.md # Communication patterns, workflow
```
**Used by:** Clarifier, Planner, Expert-Listener  
**Key principle:** Shared understanding emerges through collaborative exploration.

---

### 09-ai-development: Build with AI Agents
```
├── agents-md-standard.md       # AGENTS.md format for documenting constraints
├── ai-coding-patterns.md       # Pattern language for AI-assisted development
├── prompt-engineering.md       # Effective AI prompting techniques
├── ai-limitations.md           # Why agents skip rules and how to work around
├── context-engineering.md      # CAAP protocol for context management
└── on-demand-loading.md        # Load rules dynamically by task context
```
**Used by:** ALL AGENTS  
**Key principle:** AI agents have limitations; be explicit about constraints and rules.

---

### 10-legacy: Modernize Systems
```
├── README.md                   # Legacy modernization overview
├── monolith-analysis.md        # Identify domains in existing code
├── extraction-patterns.md      # Safe extraction of functionality
└── legacy-modernization-patterns.md # Refactoring strategies
```
**Used by:** Legacy-Analyzer, Refactorer, Architect  
**Key principle:** Understand existing system deeply before refactoring; strangler pattern.

---

### 11-error-handling: Build Robust Systems
```
├── README.md                   # Error handling overview
├── domain-errors.md            # Domain-specific errors, error events
├── result-pattern.md           # Result<T,E> type for error handling
├── design-by-contract.md       # Preconditions, postconditions, invariants
└── comprehensive-error-handling.md # Error recovery, propagation, logging
```
**Used by:** Coder, Reviewer, Architect  
**Key principle:** Errors are part of domain logic; handle them explicitly.

---

### 12-security: Build Secure Systems
```
├── README.md                   # Security patterns overview
├── domain-security.md          # Domain authorization patterns
├── auth-patterns.md            # OAuth, JWT, session management
└── comprehensive-security-guide.md # Vulnerabilities, mitigations, best practices
```
**Used by:** Architect, Reviewer, Coder  
**Key principle:** Security is a domain concern; integrate into business logic.

---

### 13-deployment: Release Reliably
```
├── README.md                   # Deployment overview + quick reference
├── deployment.md               # CI/CD, environment management
├── ci-cd-pipeline.md           # Pipeline patterns
└── staging-production.md       # Environment-specific patterns
```
**Used by:** Orchestrator, Reviewer  
**Key principle:** Deployment is rehearsed and automated; never manual.

---

## How AI Agents Use dev-knowledge

### Agent-by-Agent Reference

#### @researcher
```
Loads: 08-collaboration/impact-mapping.md
       01-requirements/problem-frames.md
Purpose: Research industry best practices for new domain
Output: Domain overview, key questions, best practices
```

#### @expert-listener
```
Loads: 02-ddd/domain-storytelling.md
       02-ddd/strategic-patterns.md
Purpose: Extract domain model from expert storytelling
Output: Domain model, invariants, events, boundaries
```

#### @clarifier
```
Loads: ALL of dev-knowledge (comprehensive clarity rules)
       08-collaboration/example-mapping.md
       01-requirements/spec-by-example.md
Purpose: Turn vague requirements into precise specs
Output: Acceptance criteria, edge cases, examples
```

#### @architect
```
Loads: 02-ddd/* (strategic + tactical)
       03-architecture/*
       06-design-patterns/clean-architecture-patterns.md
Purpose: Design system structure and boundaries
Output: Architecture diagram, bounded contexts, layers
```

#### @planner
```
Loads: 03-architecture/vertical-slice.md
       04-coding-style/file-organization.md
Purpose: Break architecture into implementable tasks
Output: Task list, file structure, acceptance criteria
```

#### @coder
```
Loads: 04-coding-style/*
       05-testing/*
       02-ddd/tactical-patterns.md
       06-design-patterns/*
       11-error-handling/*
Purpose: Implement production code
Output: Code, tests, domain events
```

#### @reviewer
```
Loads: 07-review-checklists/*
       02-ddd/tactical-patterns.md
       05-testing/testing-conventions.md
Purpose: Validate code quality and domain correctness
Output: Review with blockers, suggestions, approvals
```

#### @legacy-analyzer
```
Loads: 10-legacy/monolith-analysis.md
       02-ddd/strategic-patterns.md
Purpose: Understand existing monolithic system
Output: Domain map, coupling analysis, extraction targets
```

#### @refactorer
```
Loads: 10-legacy/extraction-patterns.md
       03-architecture/refactoring-journey.md
       06-design-patterns/clean-architecture-patterns.md
Purpose: Extract domains from monolith safely
Output: Migration plan, anti-corruption layers, go-live gates
```

#### @orchestrator
```
Loads: 09-ai-development/*
Purpose: Coordinate all agents, manage workflows, handle gates
Output: PROGRESS.md, DECISIONS.md, AI-WORKLOG.md
```

---

## Knowledge Loading Strategy (On-Demand)

Agents don't load all 13 categories. They load **only what they need** to avoid context overflow.

```
Agent        | Context Budget | Loads (~tokens) | Files
─────────────┼────────────────┼─────────────────┼────────
researcher   | 8K tokens      | 6K              | 3 files
clarifier    | 12K tokens     | 10K             | 8 files
architect    | 15K tokens     | 12K             | 12 files
coder        | 14K tokens     | 12K             | 10 files
reviewer     | 10K tokens     | 8K              | 5 files
```

**Rule:** Load files by folder, not individual pieces. Folders are cohesive units.

---

## Validation Rules for AI Agents

### All Agents Must
- [ ] Reference files with full paths: `[02-ddd/tactical-patterns.md]`
- [ ] Cite specific principles, not vague concepts
- [ ] Follow coding style from `04-coding-style/`
- [ ] Use examples from the documentation
- [ ] Call out anti-patterns from `04-coding-style/clean-code.md`

### Architect Must
- [ ] Design per `03-architecture/clean-architecture.md` (separation of concerns)
- [ ] Use DDD patterns from `02-ddd/strategic-patterns.md`
- [ ] Plan deployment per `13-deployment/deployment.md`

### Coder Must
- [ ] Write per `05-testing/tdd-workflow.md` (write test first)
- [ ] Follow naming from `04-coding-style/naming-conventions.md`
- [ ] Use DDD tactical patterns from `02-ddd/tactical-patterns.md`
- [ ] Handle errors per `11-error-handling/`

### Reviewer Must
- [ ] Use review checklists from `07-review-checklists/`
- [ ] Validate DDD invariants per `02-ddd/ddd-review.md`
- [ ] Check tests per `05-testing/testing-conventions.md`

---

## Important Constraints

### What dev-knowledge Does NOT Cover
- Project-specific domain knowledge (that's in `/projects/[project]/AGENTS.md`)
- External library APIs (loaded from documentation)
- Infrastructure setup (use `13-deployment/` as guide only)

### What dev-knowledge IS
- **Immutable principles** (don't change without versioning)
- **Reusable across all projects** (same for HRMS, banking, ecommerce)
- **Framework-agnostic** (applies to TypeScript, Python, Java, etc.)
- **Living documentation** (updated as industry standards evolve)

---

## Usage

AI agents should:

1. **Reference this knowledge base** for standard patterns
2. **Apply project-agnostic principles** to new code
3. **Suggest improvements** based on these guidelines
4. **Never introduce project-specific rules** in dev-knowledge

---

## Anti-Patterns to Avoid

- ❌ Company names or project names in examples
- ❌ Proprietary framework-specific code
- ❌ Language-specific idioms without alternatives
- ❌ Chinese/other language text
- ❌ Project-specific configurations

---

## File Structure

```
dev-knowledge/
├── index.md                    ← This document maps all files
├── README.md                   ← Folder overview (this file)
├── 01-requirements/            ← 5 files
├── 02-ddd/                     ← 6 files
├── 03-architecture/            ← 5 files
├── 04-coding-style/            ← 7 files
├── 05-testing/                 ← 5 files
├── 06-design-patterns/         ← 6 files
├── 07-review-checklists/       ← 5 files
├── 08-collaboration/           ← 4 files
├── 09-ai-development/          ← 8 files
├── 10-legacy/                  ← 4 files
├── 11-error-handling/          ← 5 files
├── 12-security/                ← 4 files
└── 13-deployment/              ← 4 files
```

**Total: 13 folders, 70 files, ~170K tokens of knowledge**

---

## Integration with AI Agent Framework

This `dev-knowledge` powers the `/ai-agents/` framework:

```
dev-knowledge/ → Informs → /ai-agents/roles/
                           ├── researcher.md
                           ├── clarifier.md
                           ├── architect.md
                           ├── planner.md
                           ├── coder.md
                           ├── reviewer.md
                           ├── legacy-analyzer.md
                           ├── refactorer.md
                           └── orchestrator.md
```

When agents are regenerated, they extract principles from dev-knowledge and embed them as rules.

---

## Maintenance & Updates

### Version Control
- `dev-knowledge/v1.0/` - Current stable
- Update with MINOR version for new files
- Update with MAJOR version for principle changes

### Regeneration Process
1. Add/update files in dev-knowledge
2. Run master prompt: `Generate AI Agent Framework from dev-knowledge`
3. New `/ai-agents/` automatically embeds updated knowledge
4. Commit both folders together

### Keeping Knowledge Current
- Review annually for industry changes
- Add new patterns as they emerge (async/await evolution, etc.)
- Maintain TypeScript examples (language updates)
- Update security section quarterly

---

## Quick Start for New Projects

```
1. Fork this dev-knowledge
2. Reference it in /ai-agents/README.md
3. Generate /ai-agents/ framework from master prompt
4. Create /projects/[project]/AGENTS.md (project-specific)
5. Start project: @orchestrator "Build [project]"
```

---

## Questions & Answers

**Q: Can I modify dev-knowledge for my company?**  
A: Yes! This is your foundation. Customize by:
- Adding company-specific patterns in each category
- Creating example files showing your code style
- Adding domain-specific error handling

**Q: What if an agent doesn't follow dev-knowledge?**  
A: Check the agent role definition in `/ai-agents/roles/`. It may have outdated rules. Regenerate from master prompt.

**Q: How big can dev-knowledge get?**  
A: Stay under 300K tokens (~60K words). Use folders to keep related files together. Link to external docs if needed.

**Q: Can multiple projects share dev-knowledge?**  
A: **YES!** That's the point. This is universal. Projects add specific knowledge in their own AGENTS.md.

---

## Success Metrics

dev-knowledge is working well if:
- ✅ AI agents cite specific files in their outputs
- ✅ Code follows conventions from `04-coding-style/`
- ✅ Domain models match patterns from `02-ddd/`
- ✅ Tests follow structure from `05-testing/`
- ✅ Reviews reference checklists from `07-review-checklists/`
- ✅ New projects reuse existing knowledge without modification

---

## File Manifest

| Category | Files | Topics | Used By |
|----------|-------|--------|---------|
| 01-requirements | 5 | Specs, examples, executable | Clarifier, Planner |
| 02-ddd | 6 | Domains, aggregates, events | Architect, Coder |
| 03-architecture | 5 | Clean arch, layers, structure | Architect, Planner |
| 04-coding-style | 7 | Style, SOLID, conventions | Coder, Reviewer |
| 05-testing | 5 | TDD, BDD, test strategy | Coder, Reviewer |
| 06-design-patterns | 6 | GoF, CQRS, event sourcing | Architect, Coder |
| 07-review-checklists | 5 | Quality gates, validation | Reviewer |
| 08-collaboration | 4 | Impact mapping, example mapping | Clarifier, Planner |
| 09-ai-development | 8 | AGENTS.md, prompt engineering | ALL AGENTS |
| 10-legacy | 4 | Monolith analysis, extraction | Legacy-Analyzer |
| 11-error-handling | 5 | Errors, contracts, results | Coder, Reviewer |
| 12-security | 4 | Auth, domain security | Architect, Reviewer |
| 13-deployment | 4 | CI/CD, staging, production | Orchestrator |

**Total: 70 files covering complete software development lifecycle**

---

**Next:** Read `/ai-agents/README.md` to learn how to use the AI agent framework powered by this knowledge base.
