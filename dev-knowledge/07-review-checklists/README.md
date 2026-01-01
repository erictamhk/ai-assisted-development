# Code Review Checklists

This folder contains comprehensive checklists and guidelines for conducting effective code and architecture reviews in AI-assisted development environments.

## Contents

### Review Types

- **[ai-code-review.md](ai-code-review.md)**: AI-assisted code review guidelines, quality checkpoints, and AI-specific review considerations
- **[architecture-review.md](architecture-review.md)**: Checklist for reviewing architectural decisions, design patterns, and system structure
- **[code-review.md](code-review.md)**: Comprehensive code review checklist covering code quality, security, performance, and maintainability
- **[ddd-review.md](ddd-review.md)**: Domain-Driven Design review checklist for bounded contexts, aggregates, and domain logic

## Quick Reference

### Review Checklist Categories

| Category | Focus Areas | Priority |
|----------|-------------|----------|
| **Code Quality** | Naming, structure, complexity | High |
| **Architecture** | Layer separation, dependencies | High |
| **Security** | Auth, validation, sensitive data | Critical |
| **Performance** | Efficiency, scalability | Medium |
| **Test Coverage** | Unit tests, integration tests | High |
| **DDD Compliance** | Aggregate design, invariants | Medium |

### Review Process

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   PREPARE   │───►│   REVIEW    │───►│   APPROVE   │
│             │    │             │    │             │
│ • Understand│    │ • Check     │    │ • Verify    │
│   context   │    │   checklist │    │   fixes     │
│ • Review    │    │ • Provide   │    │ • Merge     │
│   spec      │    │   feedback  │    │   changes   │
└─────────────┘    └─────────────┘    └─────────────┘
```

## Review Checklists

### Code Review Essentials

- [ ] Code follows project naming conventions
- [ ] Functions are small and do one thing
- [ ] No duplicate code (DRY principle)
- [ ] Error handling is comprehensive
- [ ] No hardcoded values (use constants)
- [ ] Code is self-documenting
- [ ] Comments explain "why", not "what"

### Architecture Review

- [ ] Clean Architecture layers respected
- [ ] Dependencies point inward
- [ ] No circular dependencies
- [ ] Proper separation of concerns
- [ ] Domain logic in domain layer

### Security Review

- [ ] Input validation on all boundaries
- [ ] No sensitive data in logs
- [ ] Proper authentication/authorization
- [ ] SQL injection prevention
- [ ] XSS protection

### DDD Review

- [ ] Aggregate invariants enforced
- [ ] Value objects are immutable
- [ ] Domain events for state changes
- [ ] Ubiquitous language used

## Related Topics

- **Architecture**: [03-architecture/](../03-architecture/) - Architectural patterns and principles
- **DDD**: [02-ddd/](../02-ddd/) - Domain-Driven Design patterns
- **Clean Code**: [04-coding-style/clean-code.md](../04-coding-style/clean-code.md) - Code quality guidelines
- **Testing**: [05-testing/](../05-testing/) - Testing strategies

## References

- Martin, Robert C. "Clean Code: A Handbook of Agile Software Craftsmanship"
- Fowler, Martin. "Refactoring: Improving the Design of Existing Code"
- Effective Java by Joshua Bloch
