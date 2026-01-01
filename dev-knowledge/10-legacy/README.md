# Legacy Systems

This folder contains comprehensive knowledge about working with legacy systems and modernization strategies for AI-assisted development.

## Contents

### Analysis & Understanding

- **[monolith-analysis.md](monolith-analysis.md)**: Techniques for analyzing and understanding monolithic systems to identify components, dependencies, and boundaries
- **[extraction-patterns.md](extraction-patterns.md)**: Patterns for safely extracting functionality from legacy systems without disrupting existing functionality

### Modernization

- **[legacy-modernization-patterns.md](legacy-modernization-patterns.md)**: Strategies and approaches for modernizing legacy applications including strangler fig pattern, database modernization, and incremental migration

## Quick Reference

### Legacy Analysis

| Aspect | Focus | Techniques |
|--------|-------|------------|
| **Code Structure** | Dependencies, modules | Dependency analysis, call graphs |
| **Data** | Schema, migrations | Database forensics, schema review |
| **Business Logic** | Rules, workflows | User journey mapping, documentation |
| **Integration** | External systems | API analysis, message flows |

### Modernization Strategies

| Strategy | Use Case | Risk |
|----------|----------|------|
| **Strangler Fig** | Gradual migration | Medium |
| **Database Encapsulation** | Data layer updates | Low |
| **Service Extraction** | Microservices | High |
| **API Facade** | External access | Low |

## Modernization Approaches

```
┌─────────────────────────────────────────────────────────────────┐
│                 LEGACY MODERNIZATION PATH                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│   │   ANALYZE    │───►│   STRANGLE   │───►│   REPLACE    │      │
│   │              │    │              │    │              │      │
│   │ • Map        │    │ • Add facade │    │ • Switch     │      │
│   │   dependencies│   │ • Route      │    │   traffic    │      │
│   │ • Identify   │    │   requests   │    │ • Remove old │      │
│   │   boundaries │    │ • Verify     │    │   code       │      │
│   └──────────────┘    │   behavior   │    └──────────────┘      │
│                        └──────────────┘                          │
│                               │                                   │
│                               ▼                                   │
│                        ┌──────────────┐                          │
│                        │   VERIFY     │                          │
│                        │              │                          │
│                        │ • Run tests  │                          │
│                        │ • Monitor    │                          │
│                        │ • Rollback   │                          │
│                        │   if needed  │                          │
│                        └──────────────┘                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Key Concepts

### The Strangler Fig Pattern

Named after the strangler fig tree that grows around host trees, this pattern involves:
1. Placing a facade in front of legacy system
2. Routing new requests to new implementation
3. Gradually migrating functionality
4. Removing old implementation over time

### Database Modernization

- Encapsulate database access
- Remove stored procedure logic
- Move to application layer
- Use ORM patterns

### Code Extraction

- Identify bounded contexts
- Extract with minimal changes
- Add automated tests
- Deploy independently

## Anti-Patterns to Avoid

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| **Big Bang Rewrite** | Complete rewrite at once | Incremental migration |
| **No Test Coverage** | Modifying without tests | Add tests first |
| **Feature Creep** | Adding features during migration | Stay focused |
| **Ignoring Data** | Focusing only on code | Modernize data layer |

## Related Topics

- **DDD**: [02-ddd/](../02-ddd/) - Bounded contexts for extraction
- **Architecture**: [03-architecture/](../03-architecture/) - Clean Architecture patterns
- **Testing**: [05-testing/](../05-testing/) - Testing legacy code
- **Error Handling**: [11-error-handling/](../11-error-handling/) - Handling legacy errors

## References

- Fowler, Martin. "Strangler Fig Application"
- "Working Effectively with Legacy Code" by Michael Feathers
- "Building Microservices" by Sam Newman
- "Database Modernization" patterns from Microsoft
