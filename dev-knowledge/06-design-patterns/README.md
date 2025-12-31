# Design Patterns Knowledge Base

Design patterns are reusable solutions to commonly occurring software design problems. This section covers GoF patterns, architectural patterns, and domain-specific patterns used in AI-assisted development.

## Table of Contents

| Topic | Description | Related Files |
|-------|-------------|---------------|
| [GoF Catalog](gof-catalog.md) | Gang of Four design patterns (23 patterns) | Core reference |
| [Clean Architecture Patterns](clean-architecture-patterns.md) | Port/Adapter, Use Case, Repository patterns | Application architecture |
| [CQRS & Event Sourcing](cqrs-event-sourcing.md) | Command Query Responsibility Segregation | Read/write separation |
| [Event Sourcing](event-sourcing.md) | Event-based state management | Audit trail, temporal queries |
| [Pattern Language](pattern-language.md) | Christopher Alexander's pattern language | Pattern relationships |

## Quick Reference

### Pattern Categories

| Category | Focus | Key Patterns |
|----------|-------|--------------|
| Creational | Object creation | Factory, Builder, Singleton, Prototype |
| Structural | Class/object composition | Adapter, Bridge, Decorator, Facade, Proxy |
| Behavioral | Communication between objects | Command, Observer, Strategy, State |

### When to Use Patterns

| Problem | Recommended Patterns |
|---------|---------------------|
| Complex object construction | Builder, Factory Method |
| Interface incompatibility | Adapter |
| Multiple implementations | Strategy, Bridge |
| Dynamic behavior addition | Decorator, State |
| Decoupling components | Observer, Mediator, Command |
| Simplified subsystem access | Facade |

## Architectural Patterns

### Clean Architecture Layers

```
┌───────────────────────────────────────┐
│     Frameworks & Tools (Outer)        │
│  Web, DB, UI, External Services       │
└───────────────────┬───────────────────┘
                    │
┌───────────────────▼───────────────────┐
│      Interface Adapters (Middle)       │
│  Controllers, Gateways, Presenters     │
└───────────────────┬───────────────────┘
                    │
┌───────────────────▼───────────────────┐
│     Application Services (Use Cases)   │
│         Interactors, Services          │
└───────────────────┬───────────────────┘
                    │
┌───────────────────▼───────────────────┐
│          Domain Layer (Inner)          │
│   Entities, Value Objects, Events      │
└───────────────────────────────────────┘

Dependency Rule: Inner layers don't know about outer layers
```

### Key Patterns by Layer

| Layer | Patterns |
|-------|----------|
| Domain | Entity, Value Object, Aggregate, Domain Event |
| Application | Use Case, Command, Query, Service |
| Interface Adapters | Controller, Repository, Presenter, Gateway |
| Frameworks | Database, HTTP, Messaging |

## Domain-Driven Design Patterns

### Core Patterns

| Pattern | Purpose | Example |
|---------|---------|---------|
| Entity | Objects with identity and lifecycle | `Order`, `User` |
| Value Object | Objects defined by attributes, no identity | `Money`, `Address` |
| Aggregate | Consistency boundary with single root | `Order` + `OrderItems` |
| Domain Event | Capture state changes as events | `OrderCreated`, `PaymentReceived` |

### Integration Patterns

| Pattern | Purpose |
|---------|---------|
| Repository | Abstract persistence access |
| Factory | Create aggregates and entities |
| Service | Coordinate across aggregates |
| Specification | Encapsulate business rules |

## Data Transfer Patterns

From project constitution:

| Pattern | Purpose | Naming |
|---------|---------|--------|
| DTO | Transfer data between layers | `XxxDto` |
| PO | Database entity representation | `XxxPo` |
| Mapper | Convert between representations | `XxxMapper` |
| Input/Output | Use case communication | `XxxInput`, `XxxOutput` |

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Anemic Domain | Entities without behavior | Move logic to domain |
| God Interface | One port for everything | Separate interfaces |
| Dependency Inversion Violation | Domain depending on frameworks | Use dependency injection |
| Feature Scattering | Related code in multiple layers | Organize by feature |

## Pattern Selection Guide

```
┌─────────────────────────────────────────────────────────┐
│              PATTERN SELECTION DECISION TREE            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Is this about creating objects? ──► Yes ──► [CREATIONAL]
│       │                                           │
│       No                                          │
│       ▼                                           │
│  Is this about structure/composition? ──► Yes ──► [STRUCTURAL]
│       │                                           │
│       No                                          │
│       ▼                                           │
│  Is this about behavior/communication? ──► Yes ──► [BEHAVIORAL]
│       │                                           │
│       No                                          │
│       ▼                                           │
│  Consider architectural patterns                   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Cross-References

| Topic | Related Section |
|-------|-----------------|
| Clean Architecture | [03-architecture/clean-architecture.md](../03-architecture/clean-architecture.md) |
| DDD Tactical Patterns | [02-ddd/tactical-patterns.md](../02-ddd/tactical-patterns.md) |
| TDD with Patterns | [05-testing/tdd-workflow.md](../05-testing/tdd-workflow.md) |
| Coding Conventions | [04-coding-style/naming-conventions.md](../04-coding-style/naming-conventions.md) |

## References

- Gamma, Erich, et al. "Design Patterns: Elements of Reusable Object-Oriented Software." Addison-Wesley, 1994.
- Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
- Martin, Robert C. "Clean Architecture." Prentice Hall, 2017.
- Fowler, Martin. "Patterns of Enterprise Application Architecture." Addison-Wesley, 2002.
