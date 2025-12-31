# Dev Knowledge Index

This folder contains curated knowledge for AI-assisted development, organized by topic. Each section provides comprehensive documentation with practical examples, anti-patterns, and best practices.

## 01-requirements

Requirements engineering and specification practices for capturing and validating software needs.

| File | Description |
|------|-------------|
| [executable-specs.md](01-requirements/executable-specs.md) | Transform requirements into automated tests that serve as living documentation |
| [problem-frames.md](01-requirements/problem-frames.md) | Michael Jackson's framework for analyzing software requirements through problem decomposition |
| [requirements-and-specification.md](01-requirements/requirements-and-specification.md) | Guide to creating effective requirements documents and specifications |
| [spec-by-example.md](01-requirements/spec-by-example.md) | Requirements discovery through concrete examples with Gherkin syntax |
| [specification-driven-development.md](01-requirements/specification-driven-development.md) | Development approach where specifications drive implementation and testing |

## 02-ddd

Domain-Driven Design knowledge covering strategic patterns for modeling complex domains and tactical patterns for implementation.

| File | Description |
|------|-------------|
| [README.md](02-ddd/README.md) | Overview of DDD concepts including strategic and tactical patterns with quick reference tables |
| [context-mapping.md](02-ddd/context-mapping.md) | Patterns for defining relationships between bounded contexts (Shared Kernel, Anticorruption Layer, etc.) |
| [domain-event-storming.md](02-ddd/domain-event-storming.md) | Workshop technique for collaborative exploration of domain events and business processes |
| [domain-storytelling.md](02-ddd/domain-storytelling.md) | Narrative approach to understanding domain event flows through storytelling |
| [strategic-patterns.md](02-ddd/strategic-patterns.md) | Strategic design patterns including Bounded Context, Ubiquitous Language, and Context Maps |
| [tactical-patterns.md](02-ddd/tactical-patterns.md) | Tactical patterns including Entities, Value Objects, Aggregates, Repositories, Factories, and Domain Events |

## 03-architecture

Architectural patterns and principles for structuring software systems.

| File | Description |
|------|-------------|
| [README.md](03-architecture/README.md) | Overview of architectural approaches including Clean Architecture, Layered Architecture, and Vertical Slice |
| [clean-architecture.md](03-architecture/clean-architecture.md) | Robert C. Martin's Clean Architecture with four-layer separation and dependency rules |
| [layered-architecture.md](03-architecture/layered-architecture.md) | Traditional layered architecture patterns (Presentation, Application, Domain, Infrastructure) |
| [refactoring-journey.md](03-architecture/refactoring-journey.md) | 14-step proven journey from monolith to Clean Architecture |
| [vertical-slice.md](03-architecture/vertical-slice.md) | Feature-based project organization that groups all related code together |

## 04-coding-style

Coding standards, conventions, and principles for writing maintainable code.

| File | Description |
|------|-------------|
| [clean-code.md](04-coding-style/clean-code.md) | Writing maintainable, expressive code with SOLID principles, error handling, and AI coding anti-patterns |
| [file-organization.md](04-coding-style/file-organization.md) | Feature-based file and directory organization with DDD layer patterns |
| [git-conventions.md](04-coding-style/git-conventions.md) | Git workflow, commit message conventions, and branching strategies |
| [naming-conventions.md](04-coding-style/naming-conventions.md) | Detailed naming conventions for entities, value objects, use cases, services, DTOs, and more |
| [solid-principles.md](04-coding-style/solid-principles.md) | Comprehensive SOLID principles guide (SRP, OCP, LSP, ISP, DIP) with TypeScript examples |
| [typescript-conventions.md](04-coding-style/typescript-conventions.md) | TypeScript-specific conventions including strict mode, type annotations, and async patterns |

## 05-testing

Testing methodologies, strategies, and conventions for ensuring code quality.

| File | Description |
|------|-------------|
| [bdd-gherkin.md](05-testing/bdd-gherkin.md) | Behavior-Driven Development using Gherkin syntax for readable specifications |
| [tdd-workflow.md](05-testing/tdd-workflow.md) | Test-Driven Development Red-Green-Refactor methodology with advanced patterns |
| [test-pyramids.md](05-testing/test-pyramids.md) | Testing strategy overview balancing unit, integration, and E2E tests |
| [testing-conventions.md](05-testing/testing-conventions.md) | Comprehensive testing conventions including AAA pattern, test doubles, coverage targets, and TDD/BDD practices |

## 06-design-patterns

Design patterns catalog and architectural patterns for reusable software solutions.

| File | Description |
|------|-------------|
| [README.md](06-design-patterns/README.md) | Overview of design patterns including GoF catalog, Clean Architecture patterns, and pattern selection guide |
| [clean-architecture-patterns.md](06-design-patterns/clean-architecture-patterns.md) | Port/Adapter, Use Case, Repository, and other Clean Architecture patterns |
| [cqrs-event-sourcing.md](06-design-patterns/cqrs-event-sourcing.md) | Command Query Responsibility Segregation with Event Sourcing integration |
| [event-sourcing.md](06-design-patterns/event-sourcing.md) | Event-based state management pattern for audit trails and temporal queries |
| [gof-catalog.md](06-design-patterns/gof-catalog.md) | Gang of Four design patterns (23 patterns) with TypeScript examples |
| [pattern-language.md](06-design-patterns/pattern-language.md) | Christopher Alexander's pattern language concept for documenting pattern relationships |

## 07-review-checklists

Checklists and guidelines for conducting effective code and architecture reviews.

| File | Description |
|------|-------------|
| [ai-code-review.md](07-review-checklists/ai-code-review.md) | AI-assisted code review guidelines and quality checkpoints |
| [architecture-review.md](07-review-checklists/architecture-review.md) | Checklist for reviewing architectural decisions and design patterns |
| [code-review.md](07-review-checklists/code-review.md) | Comprehensive code review checklist covering code quality, security, and performance |
| [ddd-review.md](07-review-checklists/ddd-review.md) | Domain-Driven Design review checklist for bounded contexts, aggregates, and domain logic |

## 08-collaboration

Team collaboration practices and techniques for effective software development.

| File | Description |
|------|-------------|
| [example-mapping.md](08-collaboration/example-mapping.md) | BDD discovery technique for mapping rules to examples through conversation |
| [impact-mapping.md](08-collaboration/impact-mapping.md) | Gojko Adzic's strategic planning technique linking business goals to deliverables |
| [living-documentation.md](08-collaboration/living-documentation.md) | Approaches to keeping documentation current through executable specifications and automation |
| [team-collaboration-guide.md](08-collaboration/team-collaboration-guide.md) | Guide to effective team collaboration including communication patterns and workflow |

## 09-ai-development

Guidelines, patterns, and best practices for AI-assisted software development.

| File | Description |
|------|-------------|
| [agents-md-standard.md](09-ai-development/agents-md-standard.md) | AGENTS.md standard for documenting AI agent constraints and workflows |
| [ai-agent-development-guidelines.md](09-ai-development/ai-agent-development-guidelines.md) | Comprehensive guidelines for developing with AI coding assistants |
| [ai-coding-patterns.md](09-ai-development/ai-coding-patterns.md) | Pattern language for AI-assisted development with anti-patterns and best practices |
| [ai-limitations.md](09-ai-development/ai-limitations.md) | Understanding AI agent limitations and how to work around them |
| [context-engineering.md](09-ai-development/context-engineering.md) | Managing AI context effectively with the Context-Aware Action Protocol (CAAP) |
| [on-demand-loading.md](09-ai-development/on-demand-loading.md) | Techniques for loading rules dynamically based on task context |
| [prompt-engineering.md](09-ai-development/prompt-engineering.md) | Effective prompt techniques for AI coding assistants |

## 10-legacy

Patterns and approaches for working with legacy systems and modernization.

| File | Description |
|------|-------------|
| [extraction-patterns.md](10-legacy/extraction-patterns.md) | Patterns for extracting functionality from legacy systems |
| [legacy-modernization-patterns.md](10-legacy/legacy-modernization-patterns.md) | Strategies for modernizing legacy applications |
| [monolith-analysis.md](10-legacy/monolith-analysis.md) | Techniques for analyzing and understanding monolithic systems |

## 11-error-handling

Error handling patterns and practices for robust software systems.

| File | Description |
|------|-------------|
| [comprehensive-error-handling.md](11-error-handling/comprehensive-error-handling.md) | Complete guide to error handling strategies and patterns |
| [design-by-contract.md](11-error-handling/design-by-contract.md) | Design by Contract methodology with preconditions, postconditions, and invariants |
| [domain-errors.md](11-error-handling/domain-errors.md) | Domain-specific error patterns and error handling in DDD |
| [result-pattern.md](11-error-handling/result-pattern.md) | Result type pattern for handling failures without exceptions |

## 12-security

Security patterns and best practices for building secure applications.

| File | Description |
|------|-------------|
| [auth-patterns.md](12-security/auth-patterns.md) | Authentication and authorization patterns including OAuth, JWT, and session management |
| [comprehensive-security-guide.md](12-security/comprehensive-security-guide.md) | Complete security guide covering common vulnerabilities and mitigations |
| [domain-security.md](12-security/domain-security.md) | Domain-driven security patterns and domain-specific security concerns |

## 13-deployment

Deployment practices, CI/CD pipelines, and environment management for reliable software delivery.

| File | Description |
|------|-------------|
| [deployment.md](13-deployment/deployment.md) | Comprehensive deployment guide including CI/CD pipelines, environment management, deployment strategies (Blue-Green, Canary, Rolling), and release procedures |

---

## Quick Reference by Topic

### Architecture & Design
- Clean Architecture: [03-architecture/clean-architecture.md](03-architecture/clean-architecture.md)
- Design Patterns: [06-design-patterns/gof-catalog.md](06-design-patterns/gof-catalog.md)
- DDD Patterns: [02-ddd/tactical-patterns.md](02-ddd/tactical-patterns.md)

### Code Quality
- Clean Code: [04-coding-style/clean-code.md](04-coding-style/clean-code.md)
- SOLID Principles: [04-coding-style/solid-principles.md](04-coding-style/solid-principles.md)
- Testing: [05-testing/testing-conventions.md](05-testing/testing-conventions.md)

### AI Development
- Context Engineering: [09-ai-development/context-engineering.md](09-ai-development/context-engineering.md)
- AI Patterns: [09-ai-development/ai-coding-patterns.md](09-ai-development/ai-coding-patterns.md)
- AGENTS.md: [09-ai-development/agents-md-standard.md](09-ai-development/agents-md-standard.md)

### Collaboration
- BDD & Examples: [05-testing/bdd-gherkin.md](05-testing/bdd-gherkin.md)
- Example Mapping: [08-collaboration/example-mapping.md](08-collaboration/example-mapping.md)
- Impact Mapping: [08-collaboration/impact-mapping.md](08-collaboration/impact-mapping.md)

### DevOps & Deployment
- CI/CD Pipelines: [13-deployment/deployment.md](13-deployment/deployment.md)
- Environment Management: [13-deployment/deployment.md](13-deployment/deployment.md)
- Deployment Strategies: [13-deployment/deployment.md](13-deployment/deployment.md)

---

## Using This Documentation

### For AI Coding Assistants
1. Start with [09-ai-development/agents-md-standard.md](09-ai-development/agents-md-standard.md) to understand project constraints
2. Reference pattern documents for implementation guidance
3. Use [04-coding-style/solid-principles.md](04-coding-style/solid-principles.md) for design decisions
4. Follow [05-testing/testing-conventions.md](05-testing/testing-conventions.md) for test quality

### For Team Members
1. [01-requirements/](01-requirements/) - Understanding what to build
2. [02-ddd/](02-ddd/) - Modeling the domain
3. [03-architecture/](03-architecture/) - Structuring the system
4. [04-coding-style/](04-coding-style/) - Writing quality code
5. [05-testing/](05-testing/) - Ensuring quality

### For Architects
1. [03-architecture/](03-architecture/) - Architectural patterns
2. [06-design-patterns/](06-design-patterns/) - Pattern selection
3. [07-review-checklists/](07-review-checklists/) - Review guidance
4. [13-deployment/](13-deployment/) - Deployment strategies
