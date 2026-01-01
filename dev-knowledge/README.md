# README.md: dev-knowledge Folder

**Your Universal Knowledge Base for AI-Assisted Software Development**

Welcome to `dev-knowledge` - the foundation of your AI agent framework. This folder contains 61 files organized into 13 categories, covering the complete software development lifecycle with production-grade patterns and best practices.

---

## 🎯 What Is dev-knowledge?

`dev-knowledge` is the **"encyclopedia" that powers your AI agents**. Instead of relying on generic LLM knowledge, your agents reference specific files in this folder to:

- ✅ **Build production systems** (not prototypes)
- ✅ **Follow consistent patterns** across all projects
- ✅ **Make better design decisions** based on proven practices
- ✅ **Communicate clearly** using shared vocabulary

```
Without dev-knowledge:
  AI: "I'll use any[] types because that's common"
  
With dev-knowledge:
  AI: "Per /dev-knowledge/04-coding-style/typescript-conventions.md,
       I'll use strict types with proper Value Objects"
```

---

## 📁 Folder Structure (13 Categories)

### 1️⃣ **01-requirements** - Capture What to Build
Turning vague ideas into executable specifications.

| File | Purpose |
|------|---------|
| `executable-specs.md` | Transform requirements into automated tests |
| `problem-frames.md` | Michael Jackson's requirement analysis |
| `spec-by-example.md` | Concrete examples with Gherkin syntax |
| `specification-driven-development.md` | Specs drive implementation |
| `requirements-and-specification.md` | Effective requirements documents |

**When to use:** Understanding what to build before design  
**Used by:** Clarifier, Planner  
**Key insight:** Specifications should be testable; write tests from specs

---

### 2️⃣ **02-ddd** - Model Complex Domains
Domain-Driven Design for building software that reflects business reality.

| File | Purpose |
|------|---------|
| `README.md` | DDD overview + quick reference tables |
| `strategic-patterns.md` | Bounded contexts, ubiquitous language |
| `tactical-patterns.md` | Aggregates, value objects, entities, events |
| `context-mapping.md` | How bounded contexts relate to each other |
| `domain-event-storming.md` | Workshop to discover domain events |
| `domain-storytelling.md` | Narrative understanding of domain flows |

**When to use:** Modeling domain logic, designing boundaries  
**Used by:** Architect, Expert-Listener, Coder  
**Key insight:** Domain model is the center; technology serves the domain

---

### 3️⃣ **03-architecture** - Structure Systems
Architectural patterns for organizing code and dependencies.

| File | Purpose |
|------|---------|
| `README.md` | Architecture overview + comparison |
| `clean-architecture.md` | Four-layer separation of concerns |
| `layered-architecture.md` | Traditional layers (Presentation, App, Domain, Infra) |
| `vertical-slice.md` | Feature-based organization |
| `refactoring-journey.md` | 14-step monolith → clean architecture path |

**When to use:** Structuring new projects, planning refactors  
**Used by:** Architect, Planner  
**Key insight:** Dependency flows inward; domain is independent

---

### 4️⃣ **04-coding-style** - Write Maintainable Code
Standards and conventions for consistent, expressive code.

| File | Purpose |
|------|---------|
| `clean-code.md` | SOLID principles + maintainable patterns |
| `typescript-conventions.md` | Strict TS, type safety, async patterns |
| `naming-conventions.md` | Entity, VO, UseCase, Service, DTO naming |
| `solid-principles.md` | SRP, OCP, LSP, ISP, DIP detailed |
| `file-organization.md` | Feature-based directory structure |
| `git-conventions.md` | Commits, branches, workflow |

**When to use:** Every day - writing, reviewing code  
**Used by:** Coder, Reviewer  
**Key insight:** Code is read more than written; optimize for clarity

---

### 5️⃣ **05-testing** - Ensure Quality
Testing methodologies and strategies for reliable code.

| File | Purpose |
|------|---------|
| `tdd-workflow.md` | Red-Green-Refactor methodology |
| `bdd-gherkin.md` | Behavior-driven specs in Gherkin |
| `test-pyramids.md` | Unit/Integration/E2E balance strategy |
| `testing-conventions.md` | AAA pattern, test doubles, coverage |

**When to use:** Writing code (write test first per TDD)  
**Used by:** Coder, Reviewer  
**Key insight:** Tests are specifications; they document behavior

---

### 6️⃣ **06-design-patterns** - Reusable Solutions
Design patterns for solving common problems elegantly.

| File | Purpose |
|------|---------|
| `README.md` | Pattern selection guide |
| `gof-catalog.md` | 23 Gang of Four patterns |
| `clean-architecture-patterns.md` | Ports/Adapters, Use Case, Repository |
| `cqrs-event-sourcing.md` | CQRS with event sourcing |
| `event-sourcing.md` | Event-based state management |
| `pattern-language.md` | How patterns relate to each other |

**When to use:** Designing complex solutions  
**Used by:** Architect, Coder  
**Key insight:** Use patterns to solve problems; don't over-pattern

---

### 7️⃣ **07-review-checklists** - Validate Quality
Checklists for architecture and code reviews.

| File | Purpose |
|------|---------|
| `architecture-review.md` | Design, layers, dependencies, patterns |
| `code-review.md` | Quality, SOLID, naming, security |
| `ddd-review.md` | Aggregate cohesion, event consistency |
| `ai-code-review.md` | AI-specific review considerations |

**When to use:** Pull request reviews  
**Used by:** Reviewer  
**Key insight:** Reviews are conversations about design quality

---

### 8️⃣ **08-collaboration** - Align Teams
Techniques for shared understanding and team alignment.

| File | Purpose |
|------|---------|
| `impact-mapping.md` | Business goals → deliverables |
| `example-mapping.md` | Rules → concrete examples (BDD) |
| `living-documentation.md` | Executable specs as docs |
| `team-collaboration-guide.md` | Communication, workflow, alignment |

**When to use:** Planning, discovering requirements  
**Used by:** Clarifier, Planner, Expert-Listener  
**Key insight:** Shared understanding emerges through exploration

---

### 9️⃣ **09-ai-development** - Build with AI Agents
Guidelines and patterns for AI-assisted development.

| File | Purpose |
|------|---------|
| `agents-md-standard.md` | AGENTS.md format for agent constraints |
| `ai-coding-patterns.md` | Pattern language for AI assistance |
| `prompt-engineering.md` | Effective AI prompting techniques |
| `ai-limitations.md` | Why agents skip rules, workarounds |
| `context-engineering.md` | CAAP protocol for context management |
| `on-demand-loading.md` | Load rules dynamically by task |
| `ai-agent-development-guidelines.md` | Comprehensive AI dev guidelines |

**When to use:** Every interaction with AI agents  
**Used by:** ALL AGENTS  
**Key insight:** AI agents have limits; be explicit about rules

---

### 🔟 **10-legacy** - Modernize Systems
Patterns for understanding and refactoring legacy systems.

| File | Purpose |
|------|---------|
| `monolith-analysis.md` | Identify domains in existing code |
| `extraction-patterns.md` | Safe extraction of functionality |
| `legacy-modernization-patterns.md` | Refactoring strategies |

**When to use:** Refactoring monoliths or legacy code  
**Used by:** Legacy-Analyzer, Refactorer  
**Key insight:** Understand deeply before refactoring

---

### 1️⃣1️⃣ **11-error-handling** - Build Robust Systems
Error handling patterns for production reliability.

| File | Purpose |
|------|---------|
| `domain-errors.md` | Domain-specific errors, error events |
| `result-pattern.md` | Result<T,E> type for error handling |
| `design-by-contract.md` | Preconditions, postconditions, invariants |
| `comprehensive-error-handling.md` | Recovery, propagation, logging |

**When to use:** Designing error-prone operations  
**Used by:** Coder, Reviewer, Architect  
**Key insight:** Errors are domain logic; handle explicitly

---

### 1️⃣2️⃣ **12-security** - Build Secure Systems
Security patterns and practices for protected applications.

| File | Purpose |
|------|---------|
| `domain-security.md` | Domain authorization patterns |
| `auth-patterns.md` | OAuth, JWT, session management |
| `comprehensive-security-guide.md` | Vulnerabilities, mitigations |

**When to use:** Designing authentication/authorization  
**Used by:** Architect, Reviewer  
**Key insight:** Security is a domain concern

---

### 1️⃣3️⃣ **13-deployment** - Release Reliably
Deployment practices for reliable production delivery.

| File | Purpose |
|------|---------|
| `README.md` | Deployment overview + quick reference |
| `deployment.md` | CI/CD, blue-green, canary, rolling deploys |
| `ci-cd-pipeline.md` | Pipeline patterns and practices |
| `staging-production.md` | Environment-specific patterns |

**When to use:** Planning deployment strategy  
**Used by:** Orchestrator  
**Key insight:** Deployment is automated, never manual

---

## 🚀 Quick Start (5 Minutes)

### For New Projects
```bash
1. Copy dev-knowledge/ to your project
2. Read 01-requirements/ (understand what to build)
3. Read 02-ddd/ (model the domain)
4. Read 03-architecture/ (structure the system)
5. Reference other sections as needed
```

### For AI Agents
```
@researcher    Load: 08-collaboration/impact-mapping.md
@clarifier     Load: ALL (comprehensive clarity rules)
@architect     Load: 02-ddd/ + 03-architecture/
@coder         Load: 04-coding-style/ + 05-testing/
@reviewer      Load: 07-review-checklists/
```

### For Code Review
```
1. Architecture checklist: 07-review-checklists/architecture-review.md
2. Code checklist: 07-review-checklists/code-review.md
3. DDD checklist: 07-review-checklists/ddd-review.md
```

---

## 📖 How to Use dev-knowledge

### Scenario 1: Building New Project
```
1. Read: 01-requirements/ (spec from examples)
2. Read: 02-ddd/strategic-patterns.md (model domain)
3. Read: 03-architecture/clean-architecture.md (structure system)
4. Reference: 04-coding-style/ (while coding)
5. Reference: 05-testing/ (write tests first)
6. Reference: 07-review-checklists/ (review PRs)
```

### Scenario 2: Reviewing Code
```
1. Use: 07-review-checklists/code-review.md
2. Validate: Against 04-coding-style/
3. Check: DDD patterns per 02-ddd/tactical-patterns.md
4. Verify: Tests per 05-testing/testing-conventions.md
5. Approve: If all checklists pass
```

### Scenario 3: Refactoring Monolith
```
1. Analyze: Using 10-legacy/monolith-analysis.md
2. Plan: Using 10-legacy/extraction-patterns.md
3. Design: Using 03-architecture/refactoring-journey.md
4. Implement: Following 02-ddd/ patterns
5. Deploy: Using 13-deployment/deployment.md
```

### Scenario 4: Teaching Team Member
```
1. "How do I structure code?" → 04-coding-style/file-organization.md
2. "What's DDD?" → 02-ddd/README.md
3. "How do I test?" → 05-testing/tdd-workflow.md
4. "How do I review?" → 07-review-checklists/
```

---

## 🎓 Learning Path

**Complete Path (5 weeks):**

**Week 1: Fundamentals**
- 01-requirements/problem-frames.md
- 02-ddd/README.md
- 03-architecture/README.md

**Week 2: Design**
- 02-ddd/strategic-patterns.md
- 02-ddd/tactical-patterns.md
- 03-architecture/clean-architecture.md

**Week 3: Code Quality**
- 04-coding-style/clean-code.md
- 04-coding-style/solid-principles.md
- 05-testing/tdd-workflow.md

**Week 4: Patterns & Review**
- 06-design-patterns/README.md
- 07-review-checklists/code-review.md

**Week 5: Advanced**
- 08-collaboration/impact-mapping.md
- 09-ai-development/agents-md-standard.md
- 13-deployment/deployment.md

---

## 🔍 By Role

### Architect
**Priority files:**
- 02-ddd/strategic-patterns.md
- 03-architecture/clean-architecture.md
- 06-design-patterns/clean-architecture-patterns.md
- 07-review-checklists/architecture-review.md

### Developer/Coder
**Priority files:**
- 04-coding-style/typescript-conventions.md
- 05-testing/tdd-workflow.md
- 02-ddd/tactical-patterns.md
- 11-error-handling/domain-errors.md

### QA/Test Engineer
**Priority files:**
- 05-testing/test-pyramids.md
- 05-testing/bdd-gherkin.md
- 07-review-checklists/code-review.md

### DevOps/Platform
**Priority files:**
- 13-deployment/deployment.md
- 04-coding-style/git-conventions.md

### Product Manager/BA
**Priority files:**
- 01-requirements/spec-by-example.md
- 08-collaboration/impact-mapping.md
- 08-collaboration/example-mapping.md

---

## 🔗 Integration with AI Framework

dev-knowledge powers your AI agent framework:

```
dev-knowledge/
    ↓
    Informs ↓
    ↓
/ai-agents/roles/
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

When agents are generated, they extract principles from dev-knowledge and embed them as rules.

---

## 📊 Knowledge Statistics

```
Total Categories:    13
Total Files:         70
Total Knowledge:     ~170K tokens (~35K words)
Topics Covered:      Complete SDLC

Breakdown:
├── Requirements:     5 files
├── DDD:             6 files
├── Architecture:    5 files
├── Coding Style:    7 files
├── Testing:         5 files
├── Design Patterns: 6 files
├── Review:          5 files
├── Collaboration:   4 files
├── AI Development:  8 files
├── Legacy:          4 files
├── Error Handling:  5 files
├── Security:        4 files
└── Deployment:      4 files
```
Total Categories:    13
Total Files:         61
Total Knowledge:     ~150K tokens (~30K words)
Topics Covered:      Complete SDLC

Breakdown:
├── Requirements:     5 files
├── DDD:             6 files
├── Architecture:    5 files
├── Coding Style:    7 files
├── Testing:         5 files
├── Design Patterns: 6 files
├── Review:          5 files
├── Collaboration:   4 files
├── AI Development:  8 files
├── Legacy:          4 files
├── Error Handling:  5 files
├── Security:        4 files
└── Deployment:      4 files
```

---

## ❓ FAQ

**Q: Can I modify dev-knowledge for my company?**  
A: **YES!** This is your knowledge base. Customize by:
- Adding company standards to each section
- Creating example files showing your code style
- Adding domain-specific patterns

**Q: What if I disagree with a principle?**  
A: Document your rationale and create an alternative file. Both approaches can coexist.

**Q: How do I keep it current?**  
A: Review annually for industry changes. Update:
- TypeScript examples when language evolves
- Security section quarterly
- Architecture patterns as frameworks mature

**Q: Can multiple projects share dev-knowledge?**  
A: **YES! That's the entire point.** dev-knowledge is universal (like physics laws). Projects add specific knowledge in their own AGENTS.md (like chemistry).

**Q: What if an AI agent doesn't follow dev-knowledge?**  
A: Check the agent role definition in `/ai-agents/roles/`. It may have outdated rules. Regenerate agents from dev-knowledge using the master prompt.

**Q: How big can dev-knowledge get?**  
A: Stay under 300K tokens (~60K words). Use folders to keep related content together. Link to external docs if content exceeds limits.

---

## ✅ Success Metrics

dev-knowledge is working well if:

- ✅ AI agents cite specific files in their outputs
- ✅ Code follows conventions from `04-coding-style/`
- ✅ Domain models match patterns from `02-ddd/`
- ✅ Tests follow structure from `05-testing/`
- ✅ Reviews reference checklists from `07-review-checklists/`
- ✅ Team speaks same vocabulary
- ✅ New projects reuse existing knowledge without modification
- ✅ Quality is consistent across projects

---

## 🔐 Version Control

```
.gitignore          (exclude nothing - this is source of truth)
dev-knowledge/
├── v1.0/           (current stable)
├── CHANGELOG.md    (track updates)
└── .git-history    (preserve evolution)
```

**Versioning Strategy:**
- **MAJOR** (v2.0): Principle changes
- **MINOR** (v1.1): New files, new patterns
- **PATCH** (v1.0.1): Bug fixes, clarifications

---

## 🚢 Deploying dev-knowledge

### For New Team Members
```
1. Clone repo with dev-knowledge/
2. Read: /dev-knowledge/README.md (this file)
3. Browse: Category folders relevant to your role
4. Reference: Specific files while working
```

### For New Projects
```
1. Copy dev-knowledge/ to project root
2. Create: /projects/[project]/AGENTS.md (project-specific)
3. Reference: dev-knowledge in project AGENTS.md
4. Generate: /ai-agents/ framework from master prompt
```

### For Tool Integration
```
.cursor/rules.md            ← Reference dev-knowledge
.opencode/opencode.json     ← Load dev-knowledge docs
claude.md                   ← Instructions for Claude
```

---

## 🎯 Next Steps

1. **Explore:** Browse folders that match your role
2. **Reference:** Keep relevant files open while working
3. **Contribute:** Add company-specific knowledge to folders
4. **Share:** Link colleagues to relevant sections
5. **Generate:** Run master prompt to create `/ai-agents/` framework
6. **Iterate:** Update dev-knowledge, regenerate agents quarterly

---

## 📚 File Manifest

**See `index.md` for complete file listing with descriptions.**

---

## 💡 Philosophy

> "dev-knowledge is the DNA of your development practice. It encodes your standards, patterns, and beliefs. When you share it with AI agents, you're not just building software - you're building software that reflects your values and best practices."

---

## 🙋 Support

**Questions about specific files?** Browse the relevant category README.  
**Want to update a principle?** Create a pull request with rationale.  
**Need to add new knowledge?** Create a new file in the appropriate category.

---

**Last Updated:** 2026-01-01  
**Version:** 1.0  
**Maintained by:** Your Team  
**Used by:** All AI agents in `/ai-agents/` framework
