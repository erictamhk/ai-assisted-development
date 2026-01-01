# Error Handling

This folder contains comprehensive knowledge about error handling patterns and practices for building robust software systems with AI-assisted development.

## Contents

### Core Patterns

- **[comprehensive-error-handling.md](comprehensive-error-handling.md)**: Complete guide to error handling strategies, patterns, and best practices across the application
- **[design-by-contract.md](design-by-contract.md)**: Design by Contract methodology with preconditions, postconditions, and invariants for ensuring correctness
- **[domain-errors.md](domain-errors.md)**: Domain-specific error patterns and error handling in Domain-Driven Design contexts
- **[result-pattern.md](result-pattern.md)**: Result type pattern for handling failures without exceptions in a functional programming style

## Quick Reference

### Error Handling Strategies

| Strategy | Use Case | Example |
|----------|----------|---------|
| **Exceptions** | Unexpected failures | Database connection lost |
| **Result Types** | Expected failures | Validation errors |
| **Error Codes** | Performance critical | Low-level APIs |
| **Panic** | Unrecoverable state | Initialization failures |

### Error Categories

```
┌─────────────────────────────────────────────────────────────────┐
│                    ERROR CATEGORIES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────┐    ┌─────────────────┐                    │
│   │   DOMAIN ERRORS │    │  INFRASTRUCTURE │                    │
│   │                 │    │     ERRORS      │                    │
│   │  • Validation   │    │  • Database     │                    │
│   │  • Business     │    │  • Network      │                    │
│   │    rules        │    │  • External     │                    │
│   │  • Domain       │    │    services     │                    │
│   └─────────────────┘    └─────────────────┘                    │
│                                                                  │
│   ┌─────────────────┐    ┌─────────────────┐                    │
│   │    SYSTEM       │    │    USER         │                    │
│   │    ERRORS       │    │    ERRORS       │                    │
│   │                 │    │                 │                    │
│   │  • Out of       │    │  • Bad input    │                    │
│   │    memory       │    │  • Permissions  │                    │
│   │  • Stack        │    │  • Not found    │                    │
│   │    overflow     │    │                 │                    │
│   └─────────────────┘    └─────────────────┘                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Error Handling Principles

| Principle | Description |
|-----------|-------------|
| **Fail Fast** | Detect and report errors early |
| **Fail Safe** | Recover or gracefully degrade |
| **Be Specific** | Use specific error types |
| **Include Context** | Add relevant information |
| **Log Appropriately** | Log errors with context |
| **User Friendly** | Show helpful messages to users |

## Error Patterns

### Result Type Pattern

```typescript
type Result<T, E> =
  | { success: true; value: T }
  | { success: false; error: E };

function divide(a: number, b: number): Result<number, Error> {
  if (b === 0) {
    return { success: false, error: new Error('Division by zero') };
  }
  return { success: true, value: a / b };
}
```

### Domain Errors

```typescript
class DomainError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly context: Record<string, unknown>
  ) {
    super(message);
    this.name = 'DomainError';
  }
}
```

### Error Boundaries

```typescript
class ErrorBoundary {
  catch(error: Error): void {
    if (error instanceof ValidationError) {
      this.handleValidationError(error);
    } else if (error instanceof DomainError) {
      this.handleDomainError(error);
    } else {
      this.handleUnexpectedError(error);
    }
  }
}
```

## Anti-Patterns to Avoid

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| **Silent Failures** | Ignoring errors | Log and handle appropriately |
| **Generic Errors** | Using `Error` for everything | Use specific error types |
| **Swallowing Errors** | Empty catch blocks | Handle or rethrow |
| **Leaking Internals** | Exposing stack traces | Sanitize error messages |
| **Inconsistent Handling** | Different error approaches | Standardize error handling |

## Best Practices

### 1. Define Error Hierarchy

```typescript
abstract class BaseError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
}

class ValidationError extends BaseError {
  readonly code = 'VALIDATION_ERROR';
  readonly statusCode = 400;
}

class NotFoundError extends BaseError {
  readonly code = 'NOT_FOUND';
  readonly statusCode = 404;
}
```

### 2. Use Result Types for Expected Failures

For operations that can fail in expected ways:
- Input validation
- Business rule violations
- Resource not found

### 3. Use Exceptions for Unexpected Failures

For operations that should rarely fail:
- Database connection issues
- Memory exhaustion
- External service timeouts

### 4. Include Context in Errors

```typescript
throw new DomainError(
  'Order cannot be cancelled',
  'ORDER_CANNOT_CANCEL',
  { orderId: order.id, status: order.status }
);
```

### 5. Log Errors with Context

```typescript
logger.error('Failed to process order', {
  orderId: order.id,
  customerId: order.customerId,
  error: error.message,
  stack: error.stack
});
```

## Related Topics

- **DDD**: [02-ddd/](../02-ddd/) - Domain events and error handling
- **Architecture**: [03-architecture/](../03-architecture/) - Clean Architecture error handling
- **Testing**: [05-testing/](../05-testing/) - Testing error scenarios
- **Security**: [12-security/](../12-security/) - Security error handling

## References

- "Design by Contract" by Bertrand Meyer
- "Error Handling in Node.js" by Node.js documentation
- "Functional Error Handling" by Scott Wlaschin
- "Domain-Driven Design" by Eric Evans
