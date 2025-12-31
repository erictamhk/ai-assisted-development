# Comprehensive Error Handling Guide

This document provides a comprehensive guide to error handling patterns in AI-assisted development, covering exception handling, result patterns, domain errors, and recovery strategies.

---

## Part 1: Error Handling Philosophy

### 1.1 Errors vs Exceptions

| Concept | Purpose | Examples |
|---------|---------|----------|
| **Errors** | Unexpected failures | Network outages, out of memory |
| **Exceptions** | Violations of contract | Invalid input, null reference |
| **Domain Errors** | Business rule violations | Insufficient funds, order cancelled |
| **Results** | Expected failure scenarios | Not found, validation failure |

### 1.2 When to Use Each Approach

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ERROR HANDLING DECISION TREE                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Is this an expected failure case?                                      │
│         │                                                               │
│    ┌────┴────┐                                                         │
│    │         │                                                         │
│   YES       NO                                                         │
│    │         │                                                         │
│    ▼         ▼                                                         │
│ Use Result   Is this a contract violation?                              │
│ Pattern      │                                                         │
│              │                                                         │
│    ┌─────────┴─────────┐                                               │
│    │                   │                                               │
│  Can be            Use Exception                                        │
│  represented       (throw)                                              │
│  as data?                                                               │
│    │                                                                   │
│    ├────────┐                                                          │
│    │        │                                                          │
│   YES      NO                                                          │
│    │        │                                                          │
│    ▼        ▼                                                          │
│ Return    Throw specific                                                │
│ Result    exception                                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.3 Error Handling Principles

**Principle 1: Fail Fast**
- Validate inputs immediately
- Throw exceptions on invalid state
- Don't continue with corrupted data

**Principle 2: Meaningful Errors**
- Error messages should explain what happened
- Include context (IDs, values)
- Use domain language, not technical terms

**Principle 3: Layer Consistency**
- Domain layer throws domain errors
- Application layer converts to output
- Presentation layer handles user feedback

---

## Part 2: Exception Handling Patterns

### 2.1 Exception Hierarchy

```
Error
├── SystemError
│   ├── NetworkError
│   ├── DatabaseError
│   └── OutOfMemoryError
├── ValidationError
│   ├── RequiredFieldError
│   ├── InvalidFormatError
│   └── ValueOutOfRangeError
├── DomainError
│   ├── EntityNotFoundError
│   ├── DuplicateEntityError
│   ├── InsufficientFundsError
│   └── InvalidStateTransitionError
└── ResultError (for Result pattern failures)
```

### 2.2 Base Exception Classes

```typescript
// System errors - unexpected failures
abstract class SystemError extends Error {
  abstract readonly code: string;
  readonly timestamp: Date = new Date();
  readonly recoverable: boolean;

  constructor(
    message: string,
    options?: { recoverable?: boolean; cause?: Error }
  ) {
    super(message, { cause: options?.cause });
    this.name = this.constructor.name;
    this.recoverable = options?.recoverable ?? false;
  }
}

// Domain errors - business rule violations
abstract class DomainError extends Error {
  abstract readonly code: string;
  readonly timestamp: Date = new Date();
  readonly domain: string;

  constructor(
    message: string,
    options?: { domain?: string; context?: Record<string, unknown> }
  ) {
    super(message);
    this.name = this.constructor.name;
    this.domain = options?.domain ?? 'general';
    Error.captureStackTrace(this, this.constructor);
  }
}

// Validation errors - input validation failures
abstract class ValidationError extends DomainError {
  readonly field: string;
  readonly value: unknown;

  constructor(
    field: string,
    value: unknown,
    message: string
  ) {
    super(message, { domain: 'validation' });
    this.field = field;
    this.value = value;
  }
}
```

### 2.3 Exception Handling Best Practices

```typescript
// ❌ BAD - Generic exception handling
try {
  await processOrder(order);
} catch (error) {
  console.log('Error');  // Silent failure!
}

// ❌ BAD - Swallowing exceptions
try {
  await processOrder(order);
} catch (error) {
  // Do nothing
}

// ✅ GOOD - Specific exception handling
try {
  await processOrder(order);
} catch (error) {
  if (error instanceof ValidationError) {
    return res.status(400).json({ error: error.message, field: error.field });
  }
  if (error instanceof DomainError) {
    return res.status(422).json({ error: error.message, code: error.code });
  }
  if (error instanceof SystemError) {
    logger.error('System error processing order', { error, orderId: order.id });
    return res.status(500).json({ error: 'Internal server error' });
  }
  throw error;  // Unknown error, rethrow
}

// ✅ GOOD - Using Result for expected failures
async function processOrder(order: Order): Promise<Result<OrderConfirmation, OrderError>> {
  if (!order.isValid()) {
    return { ok: false, error: new InvalidOrderError() };
  }
  
  try {
    const confirmation = await repository.save(order);
    return { ok: true, value: confirmation };
  } catch (error) {
    if (error instanceof DatabaseError) {
      return { ok: false, error: new OrderSaveError(error.message) };
    }
    throw error;
  }
}
```

---

## Part 3: Error Recovery Strategies

### 3.1 Retry Pattern

```typescript
async function withRetry<T>(
  operation: () => Promise<T>,
  options: {
    maxRetries?: number;
    baseDelay?: number;
    maxDelay?: number;
    backoff?: 'linear' | 'exponential';
    retryableErrors?: Error[];
  } = {}
): Promise<T> {
  const {
    maxRetries = 3,
    baseDelay = 1000,
    maxDelay = 30000,
    backoff = 'exponential',
    retryableErrors = [NetworkError, DatabaseError]
  } = options;

  let lastError: Error | undefined;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error as Error;
      
      // Check if error is retryable
      const isRetryable = retryableErrors.some(
        cls => lastError instanceof cls
      );
      if (!isRetryable || attempt === maxRetries) {
        throw lastError;
      }
      
      // Calculate delay
      const delay = Math.min(
        backoff === 'exponential'
          ? baseDelay * Math.pow(2, attempt)
          : baseDelay * (attempt + 1),
        maxDelay
      );
      
      // Add jitter
      const jitter = Math.random() * 0.3 * delay;
      await sleep(delay + jitter);
    }
  }
  
  throw lastError;
}
```

### 3.2 Circuit Breaker Pattern

```typescript
class CircuitBreaker {
  private state: 'closed' | 'open' | 'half-open' = 'closed';
  private failureCount: number = 0;
  private lastFailure: Date = new Date();
  private nextAttempt: Date = new Date();

  constructor(
    private readonly threshold: number = 5,
    private readonly timeout: number = 60000
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (new Date() >= this.nextAttempt) {
        this.state = 'half-open';
      } else {
        throw new CircuitOpenError(`Circuit is open. Retry after ${this.nextAttempt}`);
      }
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failureCount = 0;
    this.state = 'closed';
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailure = new Date();
    
    if (this.failureCount >= this.threshold) {
      this.state = 'open';
      this.nextAttempt = new Date(Date.now() + this.timeout);
    }
  }
}
```

### 3.3 Fallback Pattern

```typescript
async function getUserProfile(userId: string): Promise<UserProfile> {
  // Try primary source
  try {
    return await userService.getProfile(userId);
  } catch (error) {
    if (error instanceof ServiceUnavailableError) {
      // Fallback to cache
      const cached = await cache.get(`user:${userId}`);
      if (cached) {
        return cached;
      }
    }
    throw error;
  }
}

// Using fallback values
async function getExchangeRate(
  currency: string
): Promise<Result<number, ExchangeRateError>> {
  // Try primary API
  const primaryResult = await tryPrimaryAPI(currency);
  if (primaryResult.ok) return primaryResult;

  // Try secondary API
  const secondaryResult = await trySecondaryAPI(currency);
  if (secondaryResult.ok) return secondaryResult;

  // Use cached value as last resort
  const cached = await getCachedRate(currency);
  if (cached) return { ok: true, value: cached };

  // Return error
  return { ok: false, error: new ExchangeRateUnavailableError(currency) };
}
```

---

## Part 4: Error Monitoring and Observability

### 4.1 Error Logging

```typescript
interface ErrorContext {
  userId?: string;
  requestId?: string;
  operation?: string;
  metadata?: Record<string, unknown>;
}

class ErrorLogger {
  private readonly logger: Logger;

  log(error: Error, context: ErrorContext = {}): void {
    const enrichedContext = {
      ...context,
      errorType: error.constructor.name,
      errorMessage: error.message,
      stackTrace: error.stack,
      timestamp: new Date().toISOString(),
    };

    if (error instanceof SystemError) {
      this.logger.error('System error occurred', enrichedContext);
    } else if (error instanceof DomainError) {
      this.logger.warn('Domain error occurred', enrichedContext);
    } else {
      this.logger.error('Unexpected error occurred', enrichedContext);
    }
  }

  // Structured logging for errors
  logStructured(error: Error, context: ErrorContext = {}): void {
    console.log(JSON.stringify({
      level: error instanceof SystemError ? 'ERROR' : 'WARN',
      type: error.constructor.name,
      code: (error as any).code,
      message: error.message,
      context,
      timestamp: new Date().toISOString(),
      service: 'user-service',
      version: '1.0.0',
    }));
  }
}
```

### 4.2 Error Metrics

```typescript
class ErrorMetrics {
  private readonly counter: Counter;
  private readonly histogram: Histogram;

  recordError(error: Error, operation: string): void {
    this.counter.increment({
      errorType: error.constructor.name,
      operation,
      code: (error as any).code,
    });

    this.histogram.record(Date.now() - (error as any).timestamp, {
      errorType: error.constructor.name,
      operation,
    });
  }

  getErrorRate(operation: string): number {
    const errors = this.counter.get({ operation });
    const total = this.counter.get({ operation, status: 'success' });
    return errors / (errors + total);
  }

  getTopErrors(limit: number = 10): ErrorMetric[] {
    return this.counter.getAll()
      .filter(c => c.labels.errorType !== undefined)
      .sort((a, b) => b.value - a.value)
      .slice(0, limit)
      .map(c => ({
        type: c.labels.errorType,
        count: c.value,
        operation: c.labels.operation,
      }));
  }
}
```

### 4.3 Health Checks

```typescript
class HealthCheck {
  private readonly checks: HealthCheck[] = [];

  addCheck(name: string, check: () => Promise<HealthStatus>): void {
    this.checks.push({ name, check });
  }

  async performChecks(): Promise<HealthReport> {
    const results = await Promise.all(
      this.checks.map(async ({ name, check }) => {
        try {
          const status = await check();
          return { name, ...status, healthy: true };
        } catch (error) {
          return {
            name,
            healthy: false,
            error: (error as Error).message,
          };
        }
      })
    );

    return {
      status: results.every(r => r.healthy) ? 'healthy' : 'unhealthy',
      timestamp: new Date().toISOString(),
      checks: results,
    };
  }
}

// Usage
const healthCheck = new HealthCheck();
healthCheck.addCheck('database', async () => {
  await database.ping();
  return { status: 'up' };
});
healthCheck.addCheck('cache', async () => {
  await cache.ping();
  return { status: 'up' };
});
```

---

## Part 5: API Error Responses

### 5.1 Standard Error Response Format

```typescript
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
    timestamp: string;
    requestId?: string;
  };
}

// Examples
{
  "success": false,
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Validation failed for request",
    "details": {
      "fields": [
        { "field": "email", "message": "Invalid email format" },
        { "field": "age", "message": "Must be at least 18" }
      ]
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req-123-abc"
  }
}
```

### 5.2 Error Response by HTTP Status

| HTTP Status | Error Code | Pattern |
|-------------|------------|---------|
| 400 | VALIDATION_FAILED | Input validation failed |
| 401 | UNAUTHORIZED | Authentication required |
| 403 | FORBIDDEN | Permission denied |
| 404 | NOT_FOUND | Resource not found |
| 409 | CONFLICT | Conflict (duplicate, etc.) |
| 422 | BUSINESS_RULE_VIOLATION | Business rule failed |
| 429 | RATE_LIMITED | Too many requests |
| 500 | INTERNAL_ERROR | Server error |
| 503 | SERVICE_UNAVAILABLE | Service temporarily unavailable |

---

## Part 6: Anti-Patterns to Avoid

### 6.1 Exception Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Generic exceptions | Can't distinguish error types | Use specific exception classes |
| Silent catch blocks | Errors go unnoticed | Always log or handle |
| Catching Throwable | Catches everything | Catch specific exceptions |
| Exception for control flow | Misuse of exceptions | Use conditionals |
| Throwing strings | Not real exceptions | Throw Error instances |

### 6.2 Result Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Ignoring Result | Error cases ignored | Use exhaustive checking |
| Nested Result checks | Hard to read | Use flatMap/chain |
| Inconsistent Result types | Confusing API | Standardize Result type |
| Missing error context | Hard to debug | Include error details |

### 6.3 Recovery Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Infinite retry | Resource exhaustion | Set max retries |
| No backoff | Overloads system | Use exponential backoff |
| Ignoring errors | Silent failures | Log and monitor |
| Retry on domain errors | Wastes resources | Don't retry domain errors |

---

## Part 7: References

1. doc/design-by-contract.md - Design by contract patterns
2. doc/clean-code.md - Error handling in clean code
3. dev-knowledge/11-error-handling/design-by-contract.md - Contract patterns
4. dev-knowledge/11-error-handling/domain-errors.md - Domain error patterns
5. dev-knowledge/11-error-handling/result-pattern.md - Result pattern
6. ref/CONSTITUTION.md - Error handling conventions
7. ref/ai_agent_development_guidelines.md - AI agent error handling
8. Martin Fowler. "Error Handling Patterns"
9. Microsoft. "Error Handling Guidelines"
10. "Circuit Breaker Pattern" - Martin Fowler

---

## Appendix: Quick Reference

### Error Handling Decision Matrix

| Scenario | Approach | Example |
|----------|----------|---------|
| Expected failure | Result pattern | `findById` returns `Result<User, NotFoundError>` |
| Contract violation | Exception | `require(x > 0, 'x must be positive')` |
| Business rule | Domain error | `throw new InsufficientFundsError()` |
| System failure | System error + retry | `NetworkError` with retry |
| Validation | Validation error | `throw new InvalidEmailError('email')` |
| Not found | Result or exception | `return null` or `Result.none()` |

### Common Error Codes

| Code | Meaning | HTTP Status |
|------|---------|-------------|
| VALIDATION_FAILED | Input validation error | 400 |
| NOT_FOUND | Resource not found | 404 |
| DUPLICATE | Entity already exists | 409 |
| FORBIDDEN | Permission denied | 403 |
| RATE_LIMITED | Too many requests | 429 |
| INTERNAL_ERROR | Server error | 500 |
