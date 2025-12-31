# Testing Knowledge Base

Testing is a critical discipline in software development that ensures code correctness, enables refactoring, and serves as living documentation. This section covers comprehensive testing strategies from unit testing to behavior-driven development, with specific guidance for AI-assisted development.

## Table of Contents

| Topic | Description | Related Files |
|-------|-------------|---------------|
| [Test Pyramids](test-pyramids.md) | Testing strategy overview with unit/integration/E2E layers | Core foundation for all testing |
| [TDD Workflow](tdd-workflow.md) | Red-Green-Refactor methodology with advanced patterns | Complete TDD implementation guide |
| [BDD with Gherkin](bdd-gherkin.md) | Behavior-driven development using Gherkin syntax | Living documentation approach |

## Quick Reference

### Testing Types Comparison

| Type | Speed | Scope | Count | Purpose |
|------|-------|-------|-------|---------|
| Unit | Milliseconds | Single class/method | Many (60-80%) | Verify isolated logic |
| Integration | Seconds | Multiple components | Some (15-30%) | Verify component interaction |
| End-to-End | Seconds-Minutes | Full system | Few (5-10%) | Verify user scenarios |
| Manual | Minutes-Hours | Full system | Minimal | Exploratory testing |

### Testing Conventions

From project constitution:

| Convention | Rule | Example |
|------------|------|---------|
| Package Structure | Test packages mirror main | `domain.entity` → `domain.entity.test` |
| Naming | `XxxTest` for unit tests | `CalculatorTest` |
| Method Names | Describe behavior with underscores | `add_two_positive_numbers_correctly()` |
| Organization | One assertion per test (guideline) | Each test verifies one behavior |
| Setup | Use in-memory repositories | `InMemoryRepository` for fast tests |

### F.I.R.S.T. Principles

```
Fast:       Tests should run in milliseconds
Independent: Tests should not depend on each other
Repeatable:  Tests produce same result every time
Self-Validating: Clear pass/fail output
Timely:     Written before production code
```

### TDD Cycle

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│     RED      │ →  │    GREEN     │ →  │   REFACTOR   │
│ Write failing│    │ Write minimal│    │ Improve code │
│ test first   │    │ code to pass │    │ design       │
└──────────────┘    └──────────────┘    └──────────────┘
```

## Testing Pyramid

```
                    ┌─────────────┐
                   /   Manual     \
                  /    Testing     \
                 └─────────────────┘
                /    Integration     \
               /       Tests          \
              └───────────────────────┐
             /                         \
            /      End-to-End Tests     \
           /       (Few - Slow)          \
          └─────────────────────────────┐
         /                               \
        /      Integration Tests          \
       /       (Some - Medium)             \
      └─────────────────────────────────┐
     /                                   \
    /       Unit Tests (Many - Fast)      \
   /       (Foundation of TDD)             \
  └───────────────────────────────────────┘
```

### Layer Details

| Layer | Focus | Dependencies | Examples |
|-------|-------|--------------|----------|
| Unit | Business logic | None (mocked) | Pure functions, value objects |
| Integration | Component interaction | Real implementations | Repository, use cases |
| E2E | User workflows | Full stack | User scenarios, critical paths |

## Testing in AI-Assisted Development

### AI Testing Guidelines

When working with AI assistants for testing:

1. **Provide Clear Specifications**: Define expected behavior before generating tests
2. **Generate Comprehensive Cases**: Use AI to identify edge cases
3. **Validate AI-Generated Tests**: Review all AI-generated tests
4. **Use AI for Property-Based Testing**: AI can identify invariants
5. **Combine with BDD**: Translate requirements to Gherkin scenarios

### Common AI Testing Mistakes to Avoid

| Mistake | Solution |
|---------|----------|
| Testing implementation details | Test behavior, not internal state |
| Over-mocked tests | Test interactions where possible |
| Brittle tests | Focus on observable behavior |
| Slow unit tests | Mock I/O, databases, networks |

## Key Concepts

### Test-Driven Development (TDD)

TDD emphasizes writing tests before implementation. The Red-Green-Refactor cycle ensures:
- Every line of code has a test
- Design emerges from requirements
- Fearless refactoring is possible

### Behavior-Driven Development (BDD)

BDD extends TDD with natural language syntax (Gherkin):

```gherkin
Feature: Account Balance Management

  Scenario: Depositing money into an account
    Given an account with balance of 100
    When a deposit of 50 is made
    Then the new balance should be 150
```

### Property-Based Testing

Instead of specific examples, test properties that hold for all inputs:

```typescript
fc.assert(
  fc.property(fc.string(), (input) => {
    const reversed = reverse(input);
    const doubleReversed = reverse(reversed);
    expect(doubleReversed).toBe(input);
  })
);
```

## Testing Standards

### Test Organization

```
src/
├── main/
│   └── java/.../
└── test/
    └── java/...
        └── [mirror of main package structure]
```

### Test Naming Conventions

| Pattern | Example |
|---------|---------|
| Unit test | `CalculatorTest` |
| Integration test | `CreateProductUseCaseIntegrationTest` |
| Behavior test | `CreateProduct.feature` (Gherkin) |
| Method name | `should_add_two_numbers_correctly()` |

### Test Structure (Arrange-Act-Assert)

```java
@Test
void should_add_two_positive_numbers() {
    // Arrange
    Calculator calculator = new Calculator();
    
    // Act
    int result = calculator.add(2, 3);
    
    // Assert
    assertEquals(5, result);
}
```

## Advanced Topics

### Transformation Priority Premise

Kent Beck's ordering of transformations:

```
( () -> () )                           // Empty test
↓
( {assertion} -> {return constant} )  // Return a constant
↓
{return constant} -> {return input}   // Return the input
↓
{return input} -> {return input + 1}  // Apply transformation
...
```

### London School vs Classic TDD

| Approach | Focus | Mock Usage |
|----------|-------|------------|
| London School (Mockist) | Object interactions | Heavy mocking |
| Classic TDD (Detroit) | Observable behavior | Minimal mocking |

### Mutation Testing

Validates test quality by introducing code mutations:

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
</plugin>
```

Mutations that survive indicate weak tests.

## Cross-References

| Topic | Related Section |
|-------|-----------------|
| Clean Architecture Testing | [03-architecture/clean-architecture.md](../03-architecture/clean-architecture.md) |
| DDD Testing Patterns | [02-ddd/tactical-patterns.md](../02-ddd/tactical-patterns.md) |
| Clean Code in Tests | [04-coding-style/clean-code.md](../04-coding-style/clean-code.md) |
| Test Naming Conventions | [04-coding-style/naming-conventions.md](../04-coding-style/naming-conventions.md) |

## References

- Beck, Kent. "Test-Driven Development: By Example." Addison-Wesley, 2003.
- Freeman, Steve, and Nat Pryce. "Growing Object-Oriented Software, Guided by Tests." Addison-Wesley, 2009.
- Meszaros, Gerard. "xUnit Test Patterns: Refactoring Test Code." Addison-Wesley, 2007.
