# [PROJECT NAME] AGENTS.md

**Version:** 1.0  
**Last Updated:** YYYY-MM-DD  
**Framework:** AI-Assisted Development with Domain-Driven Design

---

## Overview

[Project description - 2-3 sentences explaining what this project does]

**Purpose:** This document defines project-specific constraints for AI agents working on [Project Name]. It complements the universal `dev-knowledge/` base by adding project-specific rules, conventions, and context.

---

## Project Context

### Domain Model

[Brief description of the domain - e.g., "E-commerce order management system", "Healthcare appointment scheduling", etc.]

### Key Entities

| Entity | Description | Key Attributes |
|--------|-------------|----------------|
| [Entity1] | [Description] | [Key fields] |
| [Entity2] | [Description] | [Key fields] |
| [Entity3] | [Description] | [Key fields] |

### Bounded Contexts

| Context | Purpose | Boundaries |
|---------|---------|------------|
| [Context1] | [Description] | [What it owns] |
| [Context2] | [Description] | [What it owns] |

---

## Technology Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| Language | [TypeScript/Python/Java] | Version: [X.Y] |
| Framework | [NestJS/Django/Spring] | Version: [X.Y] |
| Database | [PostgreSQL/MongoDB] | Version: [X.Y] |
| ORM | [TypeORM/Prisma/JPA] | - |
| Testing | [Jest/JUnit/Pytest] | - |
| CI/CD | [GitHub Actions/Jenkins] | - |

---

## Architecture

### Layer Structure

```
┌─────────────────────────────────────────────────┐
│                 Presentation Layer               │
│            (Controllers, Resolvers)              │
└─────────────────────┬───────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────┐
│                Application Layer                │
│              (Use Cases, Services)               │
└─────────────────────┬───────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────┐
│                  Domain Layer                   │
│            (Entities, Value Objects)             │
└─────────────────────┬───────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────┐
│              Infrastructure Layer               │
│           (Repositories, Adapters)              │
└─────────────────────────────────────────────────┘
```

### Dependency Rule

**Source code dependencies always point INWARD toward the domain.**

---

## Coding Conventions

### Naming Standards

| Pattern | Convention | Example |
|---------|------------|---------|
| Entity | PascalCase, singular | `class Order` |
| Value Object | PascalCase, descriptive | `class OrderId` |
| Use Case | PascalCase, Verb+Noun | `class CreateOrderUseCase` |
| Repository Interface | PascalCase, Entity+Repository | `interface OrderRepository` |
| DTO | PascalCase, Purpose+Dto | `class CreateOrderDto` |
| Test | Entity+Test | `class OrderTest` |

### File Organization

```
src/
├── [module-name]/           # Feature slice
│   ├── domain/             # Entities, Value Objects, Events
│   ├── application/        # Use Cases, Services
│   ├── infrastructure/     # Repositories, Adapters
│   └── interface/          # Controllers, Resolvers
└── shared/                 # Cross-cutting concerns
```

### TypeScript Standards

- ✅ Use strict mode (`"strict": true` in tsconfig.json)
- ✅ Value Objects for type safety
- ✅ Result types for error handling (`Result<T, E>`)
- ✅ Domain events for state changes
- ❌ Avoid `any` type
- ❌ Avoid magic numbers/strings (use constants)
- ❌ No business logic in controllers

---

## Testing Requirements

### Test Pyramid

```
         ┌─────────────┐
        │   E2E Tests  │   ← 10% (Critical paths)
       ┌┴─────────────┴┐
      │ Integration Tests │  ← 30% (API, Database)
     ┌┴─────────────────┴┐
    │    Unit Tests       │  ← 60% (Domain logic)
   └─────────────────────┘
```

### Test Conventions

| Test Type | Location | Naming |
|-----------|----------|--------|
| Unit | `[module]/domain/**/*.test.ts` | `[Entity].[method].test.ts` |
| Integration | `[module]/**/*.integration.test.ts` | `[Feature].integration.test.ts` |
| E2E | `e2e/**/*.e2e.test.ts` | `[Feature].e2e.test.ts` |

### TDD Workflow

1. **Red** - Write failing test
2. **Green** - Write minimal code to pass
3. **Refactor** - Improve code while keeping tests passing

---

## Error Handling

### Error Types

| Error Type | Pattern | Example |
|------------|---------|---------|
| Domain Error | Custom error class | `class OrderError extends Error` |
| Validation | Result type | `Result<T, ValidationError>` |
| Infrastructure | Try-catch with logging | - |

### Error Response Format

```typescript
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
  };
}
```

---

## Git Workflow

### Branch Naming

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/[ticket-id]-[description]` | `feature/123-add-order-validation` |
| Bugfix | `bugfix/[ticket-id]-[description]` | `bugfix/456-fix-payment-race` |
| Hotfix | `hotfix/[ticket-id]-[description]` | `hotfix/789-critical-security` |

### Commit Messages

```
[type]: [subject]

[body - optional]

[footer - optional]

Types:
- feat: New feature
- fix: Bug fix
- refactor: Code change (no feature/fix)
- test: Adding tests
- docs: Documentation changes
- chore: Maintenance tasks
```

Example:
```
feat(order): Add order cancellation with refund

Implement order cancellation flow with automatic refund processing
for cancelled orders within 24 hours.

Closes #123
```

---

## AI Agent Behavior

### Agent Roles

| Agent | Loads | Responsibility |
|-------|-------|----------------|
| @coder | 04-coding-style, 05-testing, 02-ddd/tactical | Write code, tests |
| @reviewer | 07-review-checklists | Review PRs |
| @architect | 02-ddd, 03-architecture | Design decisions |

### Agent Constraints

AI agents must:

- ✅ Reference `dev-knowledge/` for patterns
- ✅ Follow this AGENTS.md for project rules
- ✅ Write tests first (TDD)
- ✅ Use Result types for error handling
- ✅ Generate domain events for state changes
- ✅ Document decisions in DECISIONS.md

AI agents must NOT:

- ❌ Skip tests
- ❌ Use `any` type
- ❌ Put business logic in controllers
- ❌ Commit directly to main
- ❌ Introduce breaking changes without discussion

---

## Code Review Checklist

### Before Submitting PR

- [ ] All tests pass (`npm test`)
- [ ] Linter passes (`npm run lint`)
- [ ] TypeScript compilation succeeds (`npm run build`)
- [ ] No `any` types introduced
- [ ] Domain logic has unit tests (80%+ coverage)
- [ ] Integration tests for critical paths
- [ ] Self-review completed

### Reviewer Checks

- [ ] Architecture follows layer separation
- [ ] DDD patterns correctly applied
- [ ] Error handling is explicit (not try-catch silence)
- [ ] No magic numbers/strings
- [ ] Code is self-documenting (clear naming)
- [ ] Tests follow AAA pattern (Arrange, Act, Assert)

---

## Definition of Done (DoD)

A feature is complete when:

- [ ] Requirements satisfied (acceptance criteria met)
- [ ] Code reviewed and approved
- [ ] Tests written and passing (80%+ coverage)
- [ ] Documentation updated
- [ ] No linting/type errors
- [ ] Deployed to staging environment
- [ ] QA sign-off (if applicable)

---

## Validation Commands

```bash
# Run all tests
npm test

# Run linter
npm run lint

# Type check
npm run typecheck

# Build project
npm run build

# Run e2e tests
npm run test:e2e
```
---

## References

- **dev-knowledge:** `/path/to/dev-knowledge/` - Universal patterns
- **API Documentation:** [Link to Swagger/OpenAPI]
- **Database Schema:** [Link to ER diagram]

---

**Questions?** Create an issue or ask in [Slack/Discord channel]
