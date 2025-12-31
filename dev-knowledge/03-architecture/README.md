# Software Architecture

This folder contains comprehensive knowledge about software architecture patterns, including Clean Architecture, Layered Architecture (Ports and Adapters), Vertical Slice Architecture, and refactoring guidance.

## Contents

### Core Architecture Patterns

- **[clean-architecture.md](clean-architecture.md)**: Core Clean Architecture principles by Robert C. Martin, including the dependency rule, layer structure, and implementation examples

- **[layered-architecture.md](layered-architecture.md)**: Detailed guide to Ports and Adapters architecture, covering the three core layers (Domain, Application, Adapter), ports and adapters patterns, dependency injection, and CQRS integration

### Feature-Based Architecture

- **[vertical-slice.md](vertical-slice.md)**: Feature-based code organization pattern that combines modular architecture with feature-focused development, including folder structures, dependency rules, and implementation examples

### Practical Guidance

- **[refactoring-journey.md](refactoring-journey.md)**: A proven 14-step refactoring journey from monolithic application to Clean Architecture with DDD, including practical patterns and lessons learned

## Quick Reference

### Architecture Patterns Comparison

| Aspect | Clean/Layered | Vertical Slice |
|--------|--------------|----------------|
| **Organization** | By technical layer | By business feature |
| **Dependencies** | Point inward | Within feature + shared |
| **Best For** | Small-medium apps | Large, complex apps |
| **Team Scaling** | Limited | Excellent |
| **Feature Isolation** | Low | High |

### Layer Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    FRAMEWORKS & DRIVERS                         │
│              (Web, DB, UI, External Services)                   │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                      ADAPTER LAYER                               │
│    (Controllers, Presenters, Repositories, Gateways)            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                    APPLICATION LAYER                             │
│         (Use Cases, Services, Input/Output Ports)               │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                       DOMAIN LAYER                                │
│        (Entities, Value Objects, Aggregates, Events)            │
└─────────────────────────────────────────────────────────────────┘

Dependencies point INWARD toward the domain layer
```

### Key Principles

| Principle | Description |
|-----------|-------------|
| **Dependency Rule** | Dependencies always point inward |
| **Single Responsibility** | Each layer has one clear purpose |
| **Interface Segregation** | Small, focused interfaces |
| **Dependency Inversion** | Depend on abstractions, not implementations |
| **Aggregate Boundary** | Aggregate root controls access |

### Naming Conventions

| Component | Convention | Example |
|-----------|------------|---------|
| Entity | Singular noun | `Project`, `Task` |
| Value Object | Identity + `record` | `ProjectName`, `TaskId` |
| Use Case | Verb + noun + `UseCase` | `CreateProjectUseCase` |
| Service | Noun + `Service` | `ProjectService` |
| Controller | Verb + `Controller` | `CreateProjectController` |
| DTO | Noun + `Dto` | `ProjectDto` |
| PO | Noun + `Po` | `ProjectPo` |
| Repository Interface | Entity + `Repository` | `ProjectRepository` |

## Folder Structure Examples

### Layer-Based Structure

```
src/
├── domain/                    # Domain Layer
│   ├── entities/
│   ├── value-objects/
│   ├── aggregates/
│   └── events/
├── application/               # Application Layer
│   ├── use-cases/
│   ├── ports/
│   └── services/
├── adapters/                  # Adapter Layer
│   ├── inbound/
│   │   └── controllers/
│   ├── outbound/
│   │   ├── repositories/
│   │   └── services/
│   └── presenters/
└── frameworks/                # Frameworks & Drivers
    ├── web/
    └── database/
```

### Vertical Slice Structure

```
src/
├── feature-a/                 # Feature Slice
│   ├── domain/
│   ├── usecase/
│   └── adapter/
├── feature-b/
│   ├── domain/
│   ├── usecase/
│   └── adapter/
└── shared/                    # Cross-cutting concerns
    ├── adapter/
    └── io/
```

## Related Topics

- **DDD**: [02-ddd/](../02-ddd/) - Domain-Driven Design patterns that complement architecture
- **Design Patterns**: [06-design-patterns/](../06-design-patterns/) - CQRS, event sourcing, and pattern language
- **Testing**: [05-testing/](../05-testing/) - Testing strategies for layered architecture

## References

- Martin, Robert C. "Clean Architecture: A Craftsman's Guide to Software Structure and Design"
- Cockburn, Alistair. "Hexagonal Architecture"
- Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software"
- Vernon, Vaughn. "Implementing Domain-Driven Design"
- Bogard, Jimmy. "Vertical Slice Architecture"
