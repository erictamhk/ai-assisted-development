# Coding Style

This folder contains comprehensive knowledge about coding style, conventions, and best practices for software development, with a focus on AI-assisted development.

## Contents

### Core Principles

- **[clean-code.md](clean-code.md)**: Writing maintainable, expressive code with SOLID principles, error handling, and AI coding anti-patterns

### Conventions

- **[naming-conventions.md](naming-conventions.md)**: Detailed naming conventions for entities, value objects, use cases, services, controllers, DTOs, and more

### Language-Specific

- **[typescript-conventions.md](typescript-conventions.md)**: TypeScript-specific conventions including strict mode, type annotations, async patterns, and AI guidelines

### Organization

- **[file-organization.md](file-organization.md)**: Feature-based file and directory organization with DDD layer patterns

## Quick Reference

### Naming Conventions Summary

| Component | Convention | Example |
|-----------|------------|---------|
| Entity | Singular noun | `Project`, `Task` |
| Value Object | Noun/phrase + identity | `ProjectName`, `TaskId` |
| Use Case | Verb + noun + `UseCase` | `AddTaskUseCase` |
| Service | Noun + `Service` | `AddTaskService` |
| Controller | Verb + `XxxController` | `AddConsoleController` |
| DTO | Entity + `Dto` | `TaskDto` |
| PO | Entity + `Po` | `TaskPo` |
| Mapper | Entity + `Mapper` | `TaskMapper` |

### Code Quality Principles

| Principle | Description |
|-----------|-------------|
| **KISS** | Keep It Simple, Stupid - prefer straightforward solutions |
| **DRY** | Don't Repeat Yourself - extract common patterns |
| **SRP** | Single Responsibility - one reason to change |
| **Tell, Don't Ask** | Objects should tell others what to do, not be asked about state |

### SOLID Principles

| Principle | Purpose |
|-----------|---------|
| **S**ingle Responsibility | Each class has one reason to change |
| **O**pen/Closed | Open for extension, closed for modification |
| **L**iskov Substitution | Subtypes can replace base types |
| **I**nterface Segregation | Small, focused interfaces |
| **D**ependency Inversion | Depend on abstractions, not implementations |

### Clean Code Checklist

- [ ] Meaningful, descriptive names
- [ ] Small, focused functions
- [ ] Comments explain "why", not "what"
- [ ] Proper error handling with specific exceptions
- [ ] Immutable data where possible
- [ ] Constructor injection for dependencies
- [ ] TypeScript strict mode enabled
- [ ] No magic numbers or strings
- [ ] Consistent formatting
- [ ] Single responsibility per class

## Related Topics

- **Architecture**: [03-architecture/](../03-architecture/) - Clean Architecture and layered patterns
- **DDD**: [02-ddd/](../02-ddd/) - Domain-Driven Design patterns
- **Design Patterns**: [06-design-patterns/](../06-design-patterns/) - Design pattern implementations
- **Testing**: [05-testing/](../05-testing/) - Testing conventions

## References

- Martin, Robert C. "Clean Code: A Handbook of Agile Software Craftsmanship"
- Martin, Robert C. "Clean Architecture: A Craftsman's Guide to Software Structure and Design"
- Effective Java by Joshua Bloch
- TypeScript Deep Dive by Basarat Ali Syed
