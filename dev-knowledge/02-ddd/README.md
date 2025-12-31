# Domain-Driven Design (DDD)

This folder contains comprehensive knowledge about Domain-Driven Design, covering both strategic and tactical patterns for building complex domain models.

## Contents

### Strategic Design
- **[strategic-patterns.md](strategic-patterns.md)**: Strategic design patterns including bounded contexts, ubiquitous language, and domain landscape organization

### Tactical Design
- **[tactical-patterns.md](tactical-patterns.md)**: Tactical patterns including entities, value objects, aggregates, repositories, factories, domain services, and domain events

### Context Mapping
- **[context-mapping.md](context-mapping.md)**: Patterns for defining relationships between bounded contexts including shared kernel, customer-supplier, conformist, anticorruption layer, and open host service

### Event Discovery
- **[domain-event-storming.md](domain-event-storming.md)**: Workshop technique for collaborative domain exploration through domain events
- **[domain-storytelling.md](domain-storytelling.md)**: Narrative approach to understanding domain event flows and business processes

## Quick Reference

### Strategic Patterns
| Pattern | Purpose |
|---------|---------|
| Bounded Context | Define explicit boundaries for domain models |
| Ubiquitous Language | Shared vocabulary between domain experts and developers |
| Context Map | Document relationships between contexts |
| Subdomain Classification | Identify core, supporting, and generic domains |

### Tactical Patterns
| Pattern | Purpose |
|---------|---------|
| Entity | Objects with identity and lifecycle |
| Value Object | Immutable objects without identity |
| Aggregate Root | Entry point to consistency boundary |
| Domain Event | Capture significant state changes |
| Repository | Abstract persistence access |
| Factory | Encapsulate complex creation |
| Domain Service | Operations spanning multiple objects |

### Context Mapping Patterns
| Pattern | Use Case |
|---------|----------|
| Shared Kernel | Common code between contexts |
| Customer-Supplier | Upstream/downstream service relationship |
| Conformist | Accept upstream model as-is |
| Anticorruption Layer | Translate between incompatible models |
| Open Host Service | Published API for integration |

## Related Topics

- **Architecture**: [03-architecture](../03-architecture/) - Clean architecture and layered patterns
- **Design Patterns**: [06-design-patterns](../06-design-patterns/) - CQRS, event sourcing, and pattern language
- **Testing**: [05-testing](../05-testing/) - BDD and TDD workflows
- **Collaboration**: [08-collaboration](../08-collaboration/) - Example mapping and living documentation

## References

- Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software."
- Vernon, Vaughn. "Implementing Domain-Driven Design."
- Vernon, Vaughn. "Domain-Driven Design Distilled."
- Brandolini, Alberto. "Introducing EventStorming"
