# Agent Knowledge Mapping

**Version:** 1.1
**Last Updated:** 2026-01-02
**Purpose:** Maps each agent role to relevant `dev-knowledge/` files for context loading

---

## RESEARCHER

**Purpose:** Gather and synthesize industry knowledge for AI-assisted development

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 09-ai-development | `09-ai-development/ai-coding-patterns.md` | Required |
| 09-ai-development | `09-ai-development/prompt-engineering.md` | Required |
| 09-ai-development | `09-ai-development/ai-agent-development-guidelines.md` | Required |
| 09-ai-development | `09-ai-development/ai-limitations.md` | Required |
| 09-ai-development | `09-ai-development/context-engineering.md` | Required |
| 09-ai-development | `09-ai-development/agents-md-standard.md` | Required |
| 01-requirements | `01-requirements/problem-frames.md` | Recommended |
| 01-requirements | `01-requirements/executable-specs.md` | Recommended |
| 01-requirements | `01-requirements/specification-driven-development.md` | Recommended |
| 07-review-checklists | `07-review-checklists/ai-code-review.md` | Recommended |

**Output Path:** RESEARCHER → Orchestrator → REVIEWER → Orchestrator → Human

---

## EXPERT-LISTENER

**Purpose:** Conduct domain expert interviews and capture tacit knowledge

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 08-collaboration | `08-collaboration/example-mapping.md` | Required |
| 08-collaboration | `08-collaboration/impact-mapping.md` | Required |
| 01-requirements | `01-requirements/spec-by-example.md` | Required |
| 02-ddd | `02-ddd/domain-storytelling.md` | Required |
| 02-ddd | `02-ddd/domain-event-storming.md` | Required |
| 08-collaboration | `08-collaboration/team-collaboration-guide.md` | Recommended |
| 08-collaboration | `08-collaboration/living-documentation.md` | Recommended |
| 02-ddd | `02-ddd/context-mapping.md` | Recommended |

**Output Path:** EXPERT-LISTENER → Orchestrator → REVIEWER → Orchestrator → Human

---

## CLARIFIER

**Purpose:** Transform vague requirements into precise specifications

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 01-requirements | `01-requirements/requirements-and-specification.md` | Required |
| 01-requirements | `01-requirements/spec-by-example.md` | Required |
| 08-collaboration | `08-collaboration/example-mapping.md` | Required |
| 02-ddd | `02-ddd/context-mapping.md` | Required |
| 08-collaboration | `08-collaboration/impact-mapping.md` | Required |
| 01-requirements | `01-requirements/problem-frames.md` | Required |
| 01-requirements | `01-requirements/executable-specs.md` | Recommended |
| 01-requirements | `01-requirements/specification-driven-development.md` | Recommended |
| 02-ddd | `02-ddd/domain-storytelling.md` | Recommended |
| 07-review-checklists | `07-review-checklists/ddd-review.md` | Recommended |

**Output Path:** CLARIFIER → Orchestrator → REVIEWER → Orchestrator → Human

---

## ARCHITECT

**Purpose:** Design system architecture following domain-driven and clean architecture principles

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 02-ddd | `02-ddd/strategic-patterns.md` | Required |
| 02-ddd | `02-ddd/tactical-patterns.md` | Required |
| 03-architecture | `03-architecture/clean-architecture.md` | Required |
| 06-design-patterns | `06-design-patterns/clean-architecture-patterns.md` | Required |
| 06-design-patterns | `06-design-patterns/cqrs-event-sourcing.md` | Required |
| 06-design-patterns | `06-design-patterns/pattern-language.md` | Required |
| 03-architecture | `03-architecture/vertical-slice.md` | Required |
| 03-architecture | `03-architecture/layered-architecture.md` | Recommended |
| 03-architecture | `03-architecture/refactoring-journey.md` | Recommended |
| 02-ddd | `02-ddd/context-mapping.md` | Recommended |
| 06-design-patterns | `06-design-patterns/gof-catalog.md` | Reference |
| 06-design-patterns | `06-design-patterns/event-sourcing.md` | Reference |
| 07-review-checklists | `07-review-checklists/architecture-review.md` | Required |

**Output Path:** ARCHITECT → Orchestrator → REVIEWER → Orchestrator → Human

---

## PLANNER

**Purpose:** Break down work into manageable tasks with clear dependencies

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 01-requirements | `01-requirements/problem-frames.md` | Required |
| 01-requirements | `01-requirements/specification-driven-development.md` | Required |
| 03-architecture | `03-architecture/vertical-slice.md` | Required |
| 08-collaboration | `08-collaboration/impact-mapping.md` | Required |
| 05-testing | `05-testing/tdd-workflow.md` | Required |
| 05-testing | `05-testing/test-pyramids.md` | Required |
| 05-testing | `05-testing/bdd-gherkin.md` | Recommended |
| 13-deployment | `13-deployment/ci-cd-pipeline.md` | Recommended |
| 13-deployment | `13-deployment/staging-production.md` | Recommended |
| 10-legacy | `10-legacy/monolith-analysis.md` | Reference |

**Output Path:** PLANNER → Orchestrator → REVIEWER → Orchestrator → Human

---

## CODER

**Purpose:** Implement features following clean code, SOLID, and testing principles

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 04-coding-style | `04-coding-style/clean-code.md` | Required |
| 04-coding-style | `04-coding-style/solid-principles.md` | Required |
| 04-coding-style | `04-coding-style/naming-conventions.md` | Required |
| 04-coding-style | `04-coding-style/typescript-conventions.md` | Required |
| 04-coding-style | `04-coding-style/file-organization.md` | Required |
| 05-testing | `05-testing/tdd-workflow.md` | Required |
| 05-testing | `05-testing/testing-conventions.md` | Required |
| 05-testing | `05-testing/bdd-gherkin.md` | Required |
| 11-error-handling | `11-error-handling/result-pattern.md` | Required |
| 11-error-handling | `11-error-handling/design-by-contract.md` | Required |
| 06-design-patterns | `06-design-patterns/gof-catalog.md` | Reference |
| 11-error-handling | `11-error-handling/domain-errors.md` | Recommended |
| 11-error-handling | `11-error-handling/comprehensive-error-handling.md` | Reference |
| 07-review-checklists | `07-review-checklists/code-review.md` | Required |

**Output Path:** CODER → Orchestrator → REVIEWER → Orchestrator → Human

---

## REVIEWER

**Purpose:** Validate code, architecture, and domain design quality

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 07-review-checklists | `07-review-checklists/code-review.md` | Required |
| 07-review-checklists | `07-review-checklists/architecture-review.md` | Required |
| 07-review-checklists | `07-review-checklists/ddd-review.md` | Required |
| 07-review-checklists | `07-review-checklists/ai-code-review.md` | Required |
| 04-coding-style | `04-coding-style/clean-code.md` | Reference |
| 04-coding-style | `04-coding-style/solid-principles.md` | Reference |
| 02-ddd | `02-ddd/strategic-patterns.md` | Reference |
| 03-architecture | `03-architecture/clean-architecture.md` | Reference |
| 05-testing | `05-testing/testing-conventions.md` | Reference |
| 11-error-handling | `11-error-handling/comprehensive-error-handling.md` | Reference |
| 12-security | `12-security/comprehensive-security-guide.md` | Reference |

**Output Path:** REVIEWER → Orchestrator → Human (final approval)

---

## LEGACY-ANALYZER

**Purpose:** Analyze legacy systems and plan modernization strategies

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 10-legacy | `10-legacy/monolith-analysis.md` | Required |
| 10-legacy | `10-legacy/extraction-patterns.md` | Required |
| 10-legacy | `10-legacy/legacy-modernization-patterns.md` | Required |
| 03-architecture | `03-architecture/refactoring-journey.md` | Required |
| 03-architecture | `03-architecture/clean-architecture.md` | Reference |
| 06-design-patterns | `06-design-patterns/clean-architecture-patterns.md` | Reference |
| 02-ddd | `02-ddd/tactical-patterns.md` | Reference |
| 12-security | `12-security/domain-security.md` | Reference |
| 13-deployment | `13-deployment/deployment.md` | Reference |

**Output Path:** LEGACY-ANALYZER → Orchestrator → REVIEWER → Orchestrator → Human

---

## REFACTORER

**Purpose:** Improve code quality while preserving functionality

**Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 04-coding-style | `04-coding-style/clean-code.md` | Required |
| 04-coding-style | `04-coding-style/solid-principles.md` | Required |
| 03-architecture | `03-architecture/refactoring-journey.md` | Required |
| 06-design-patterns | `06-design-patterns/gof-catalog.md` | Required |
| 06-design-patterns | `06-design-patterns/pattern-language.md` | Required |
| 06-design-patterns | `06-design-patterns/clean-architecture-patterns.md` | Required |
| 11-error-handling | `11-error-handling/result-pattern.md` | Required |
| 11-error-handling | `11-error-handling/design-by-contract.md` | Required |
| 11-error-handling | `11-error-handling/comprehensive-error-handling.md` | Reference |
| 02-ddd | `02-ddd/tactical-patterns.md` | Reference |
| 05-testing | `05-testing/tdd-workflow.md` | Recommended |
| 07-review-checklists | `07-review-checklists/code-review.md` | Required |

**Output Path:** REFACTORER → Orchestrator → REVIEWER → Orchestrator → Human

---

## Category Reference

| Category | Purpose | Agents |
|----------|---------|--------|
| 01-requirements | Specs, examples, problem frames | CLARIFIER, PLANNER, RESEARCHER |
| 02-ddd | Domain design, events, contexts | ARCHITECT, CLARIFIER, REFACTORER, REVIEWER |
| 03-architecture | Clean, layered, vertical slice | ARCHITECT, PLANNER, LEGACY-ANALYZER, REFACTORER |
| 04-coding-style | Clean code, naming, git | CODER, REFACTORER, REVIEWER |
| 05-testing | TDD, BDD, conventions | CODER, PLANNER, REFACTORER |
| 06-design-patterns | CQRS, event sourcing, patterns | ARCHITECT, REFACTORER |
| 07-review-checklists | Code, architecture, DDD review | REVIEWER |
| 08-collaboration | Example/impact mapping | CLARIFIER, EXPERT-LISTENER, PLANNER |
| 09-ai-development | AI agents, prompting | RESEARCHER |
| 10-legacy | Legacy extraction, modernization | LEGACY-ANALYZER |
| 11-error-handling | Result pattern, design by contract | CODER, REFACTORER |
| 12-security | Auth patterns, domain security | REVIEWER |
| 13-deployment | CI/CD, deployment | PLANNER |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.1 | 2026-01-02 | Added Output Path for each agent, quality gate pattern |
| 1.0 | 2026-01-02 | Initial knowledge mapping |

---

**Template Version:** 1.1
