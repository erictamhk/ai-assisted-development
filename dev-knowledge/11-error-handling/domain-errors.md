# Domain Errors

This document defines domain error patterns for AI-assisted development, focusing on specific, meaningful error types that communicate business rule violations clearly.

---

## 1. Error Philosophy

### 1.1 Domain Errors vs System Errors

| Type | Purpose | Examples |
|------|---------|----------|
| **Domain Errors** | Business rule violations | InvalidEmail, InsufficientFunds, OrderCancelled |
| **System Errors** | Technical failures | NetworkTimeout, DatabaseUnavailable, OutOfMemory |
| **Validation Errors** | Input validation failures | RequiredField, InvalidFormat, ValueOutOfRange |

### 1.2 When to Use Domain Errors

```typescript
// ✅ USE - Business rule violations
class OrderCannotBeCancelledError extends DomainError {
  constructor(orderId: OrderId, status: OrderStatus) {
    super(`Order ${orderId} cannot be cancelled in status: ${status}`);
  }
}

// ✅ USE - Domain invariants
class NegativeAmountError extends DomainError {
  constructor(field: string) {
    super(`${field} cannot be negative`);
  }
}

// ❌ DON'T - Technical errors
class DatabaseConnectionError extends DomainError {
  // Should be a system/infra error, not domain
}
```

---

## 2. Domain Error Structure

### 2.1 Base Error Class

```typescript
// Base class for all domain errors
abstract class DomainError extends Error {
  abstract readonly code: string;
  readonly timestamp: Date;

  constructor(
    message: string,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
    this.timestamp = new Date();
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON(): DomainErrorResponse {
    return {
      name: this.name,
      code: this.code,
      message: this.message,
      timestamp: this.timestamp.toISOString(),
      context: this.context,
    };
  }
}

// Response format for API
interface DomainErrorResponse {
  name: string;
  code: string;
  message: string;
  timestamp: string;
  context?: Record<string, unknown>;
}
```

### 2.2 Concrete Error Examples

```typescript
// Validation domain errors
class RequiredFieldError extends DomainError {
  readonly code = 'REQUIRED_FIELD';
  
  constructor(field: string) {
    super(`Field '${field}' is required`, { field });
  }
}

class InvalidFormatError extends DomainError {
  readonly code = 'INVALID_FORMAT';
  
  constructor(field: string, expected: string, actual: string) {
    super(
      `Field '${field}' has invalid format. Expected: ${expected}, got: ${actual}`,
      { field, expected, actual }
    );
  }
}

class ValueOutOfRangeError extends DomainError {
  readonly code = 'VALUE_OUT_OF_RANGE';
  
  constructor(
    field: string,
    min: number,
    max: number,
    actual: number
  ) {
    super(
      `Field '${field}' must be between ${min} and ${max}, got: ${actual}`,
      { field, min, max, actual }
    );
  }
}

// Business rule domain errors
class EntityNotFoundError extends DomainError {
  readonly code = 'ENTITY_NOT_FOUND';
  
  constructor(entityType: string, id: string) {
    super(`${entityType} with id '${id}' not found`, { entityType, id });
  }
}

class DuplicateEntityError extends DomainError {
  readonly code = 'DUPLICATE_ENTITY';
  
  constructor(entityType: string, field: string, value: string) {
    super(
      `${entityType} with ${field} '${value}' already exists`,
      { entityType, field, value }
    );
  }
}

class BusinessRuleViolationError extends DomainError {
  readonly code = 'BUSINESS_RULE_VIOLATION';
  
  constructor(rule: string, details?: string) {
    super(`Business rule violated: ${rule}${details ? `. ${details}` : ''}`, {
      rule,
      details,
    });
  }
}

// State transition errors
class InvalidStateTransitionError extends DomainError {
  readonly code = 'INVALID_STATE_TRANSITION';
  
  constructor(
    entity: string,
    currentState: string,
    attemptedAction: string
  ) {
    super(
      `Cannot ${attemptedAction} on ${entity} in state: ${currentState}`,
      { entity, currentState, attemptedAction }
    );
  }
}

// Concurrency errors
class ConcurrentModificationError extends DomainError {
  readonly code = 'CONCURRENT_MODIFICATION';
  
  constructor(entity: string, id: string, version: number) {
    super(
      `${entity} '${id}' was modified by another process. Current version: ${version}`,
      { entity, id, version }
    );
  }
}
```

---

## 3. Value Object Validation Errors

### 3.1 Email Validation

```typescript
class InvalidEmailError extends DomainError {
  readonly code = 'INVALID_EMAIL';
  
  constructor(email: string, reason?: string) {
    super(
      `Invalid email address: '${email}'${reason ? `. ${reason}` : ''}`,
      { email, reason }
    );
  }
}

// Usage in value object
class Email implements ValueObject {
  private readonly email: string;
  
  constructor(value: string) {
    if (!value || value.trim().length === 0) {
      throw new InvalidEmailError(value, 'Email cannot be empty');
    }
    
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) {
      throw new InvalidEmailError(value, 'Invalid email format');
    }
    
    this.email = value.toLowerCase().trim();
  }
  
  get value(): string {
    return this.email;
  }
}
```

### 3.2 Money Validation

```typescript
class InvalidAmountError extends DomainError {
  readonly code = 'INVALID_AMOUNT';
  
  constructor(amount: number, reason?: string) {
    super(
      `Invalid amount: ${amount}${reason ? `. ${reason}` : ''}`,
      { amount, reason }
    );
  }
}

class CurrencyMismatchError extends DomainError {
  readonly code = 'CURRENCY_MISMATCH';
  
  constructor(operation: string, expected: string, actual: string) {
    super(
      `Currency mismatch during ${operation}. Expected: ${expected}, Actual: ${actual}`,
      { operation, expected, actual }
    );
  }
}

class InsufficientFundsError extends DomainError {
  readonly code = 'INSUFFICIENT_FUNDS';
  
  constructor(
    accountId: string,
    available: Money,
    requested: Money
  ) {
    super(
      `Insufficient funds in account '${accountId}'. Available: ${available}, Requested: ${requested}`,
      { accountId, available: available.value, requested: requested.value }
    );
  }
}
```

### 3.3 Date/Time Validation

```typescript
class PastDateError extends DomainError {
  readonly code = 'PAST_DATE';
  
  constructor(field: string, date: Date) {
    super(
      `${field} cannot be in the past. Got: ${date.toISOString()}`,
      { field, date: date.toISOString() }
    );
  }
}

class InvalidDateRangeError extends DomainError {
  readonly code = 'INVALID_DATE_RANGE';
  
  constructor(startDate: Date, endDate: Date) {
    super(
      `Invalid date range. End date (${endDate.toISOString()}) must be after start date (${startDate.toISOString()})`,
      { startDate: startDate.toISOString(), endDate: endDate.toISOString() }
    );
  }
}
```

---

## 4. Aggregate-Specific Errors

### 4.1 Order Errors

```typescript
class OrderNotFoundError extends EntityNotFoundError {
  constructor(orderId: OrderId) {
    super('Order', orderId.value);
  }
}

class OrderCannotBeModifiedError extends InvalidStateTransitionError {
  constructor(orderId: OrderId, status: OrderStatus) {
    super('Order', status.toString(), 'modify');
  }
}

class OrderAlreadyCancelledError extends BusinessRuleViolationError {
  constructor(orderId: OrderId) {
    super('OrderCancellation', `Order ${orderId} is already cancelled`);
  }
}

class OrderMinimumNotMetError extends BusinessRuleViolationError {
  constructor(orderId: OrderId, minimum: Money, actual: Money) {
    super(
      'OrderMinimum',
      `Order ${orderId} minimum order amount not met. Minimum: ${minimum}, Actual: ${actual}`
    );
  }
}
```

### 4.2 User Errors

```typescript
class UserNotFoundError extends EntityNotFoundError {
  constructor(userId: UserId) {
    super('User', userId.value);
  }
}

class UserEmailAlreadyExistsError extends DuplicateEntityError {
  constructor(email: string) {
    super('User', 'email', email);
  }
}

class UserAccountLockedError extends DomainError {
  readonly code = 'ACCOUNT_LOCKED';
  
  constructor(userId: UserId, reason: string, lockedUntil?: Date) {
    super(
      `Account for user ${userId} is locked. Reason: ${reason}${lockedUntil ? `. Retry after: ${lockedUntil.toISOString()}` : ''}`,
      { userId: userId.value, reason, lockedUntil: lockedUntil?.toISOString() }
    );
  }
}

class InvalidCredentialsError extends DomainError {
  readonly code = 'INVALID_CREDENTIALS';
  
  constructor(attemptsRemaining?: number) {
    super(
      `Invalid credentials${attemptsRemaining !== undefined ? `. ${attemptsRemaining} attempts remaining` : ''}`,
      { attemptsRemaining }
    );
  }
}
```

### 4.3 Task/Project Errors

```typescript
class TaskNotFoundError extends EntityNotFoundError {
  constructor(taskId: TaskId) {
    super('Task', taskId.value);
  }
}

class TaskAlreadyCompletedError extends InvalidStateTransitionError {
  constructor(taskId: TaskId) {
    super('Task', 'COMPLETED', 'complete');
  }
}

class ProjectNotFoundError extends EntityNotFoundError {
  constructor(projectId: ProjectId) {
    super('Project', projectId.value);
  }
}

class CannotDeleteProjectWithTasksError extends BusinessRuleViolationError {
  constructor(projectId: ProjectId, taskCount: number) {
    super(
      'ProjectDeletion',
      `Cannot delete project ${projectId}. It has ${taskCount} active tasks`
    );
  }
}
```

---

## 5. Error Handling Patterns

### 5.1 Domain Error Handling in Use Cases

```typescript
interface Result<T, E extends DomainError = DomainError> {
  isSuccess: boolean;
  isFailure: boolean;
  getValue(): T;
  getError(): E;
}

class CreateOrderUseCase {
  constructor(
    private orderRepository: OrderRepository,
    private eventPublisher: EventPublisher
  ) {}

  async execute(input: CreateOrderInput): Promise<Result<Order, DomainError>> {
    try {
      // Validation
      if (!input.items || input.items.length === 0) {
        return Result.failure(
          new OrderMinimumNotMetError(OrderId.generate(), Money.zero(), Money.zero())
        );
      }

      // Business logic
      const order = Order.create(input);
      await this.orderRepository.save(order);
      
      // Events
      this.eventPublisher.publish(new OrderCreatedEvent(order.id));

      return Result.success(order);
    } catch (error) {
      if (error instanceof DomainError) {
        return Result.failure(error);
      }
      // Wrap unexpected errors
      return Result.failure(
        new UnexpectedError('An unexpected error occurred', { originalError: error })
      );
    }
  }
}
```

### 5.2 Error Translation at Boundaries

```typescript
// Controller level - translate domain errors to HTTP responses
class OrderController {
  constructor(private createOrderUseCase: CreateOrderUseCase) {}

  @Post('/orders')
  async createOrder(req: Request): Promise<Response> {
    const result = await this.createOrderUseCase.execute(req.body);

    if (result.isSuccess) {
      return Response.created(result.getValue());
    }

    const error = result.getError();
    const statusCode = this.mapErrorToStatusCode(error);

    return Response.error(statusCode, {
      code: error.code,
      message: error.message,
      details: error.context,
    });
  }

  private mapErrorToStatusCode(error: DomainError): number {
    switch (error.code) {
      case 'ENTITY_NOT_FOUND':
        return 404;
      case 'INVALID_FORMAT':
      case 'REQUIRED_FIELD':
      case 'VALUE_OUT_OF_RANGE':
        return 400;
      case 'DUPLICATE_ENTITY':
        return 409;
      case 'INVALID_STATE_TRANSITION':
      case 'BUSINESS_RULE_VIOLATION':
        return 422;
      default:
        return 500;
    }
  }
}
```

### 5.3 Error Recovery Patterns

```typescript
// Retry for transient errors
async function withRetry<T>(
  operation: () => Promise<T>,
  maxRetries: number = 3,
  delayMs: number = 1000
): Promise<T> {
  let lastError: Error | null = null;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error as Error;
      
      if (error instanceof TransientError) {
        const delay = delayMs * Math.pow(2, attempt - 1);
        await sleep(delay);
        continue;
      }
      
      throw error;
    }
  }

  throw lastError;
}

// Circuit breaker for repeated failures
class CircuitBreaker {
  private failures = 0;
  private lastFailure: Date | null = null;
  private state: 'closed' | 'open' | 'half-open' = 'closed';

  constructor(
    private readonly threshold: number = 5,
    private readonly timeoutMs: number = 30000
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailure!.getTime() > this.timeoutMs) {
        this.state = 'half-open';
      } else {
        throw new CircuitOpenError('Circuit breaker is open');
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
    this.failures = 0;
    this.state = 'closed';
  }

  private onFailure(): void {
    this.failures++;
    this.lastFailure = new Date();
    if (this.failures >= this.threshold) {
      this.state = 'open';
    }
  }
}
```

---

## 6. AI-Specific Guidelines

### 6.1 When Generating Domain Errors

1. **Create specific error types** - Don't use generic errors
2. **Include relevant context** - Pass relevant data to the error
3. **Use business language** - Error messages should be understandable by business stakeholders
4. **Follow naming conventions** - `{DomainConcept}{ErrorType}Error`
5. **Extend base class** - All domain errors should extend `DomainError`

### 6.2 Error Generation Examples

```typescript
// ✅ GOOD - Specific domain error with context
class InventoryInsufficientError extends DomainError {
  readonly code = 'INSUFFICIENT_INVENTORY';
  
  constructor(
    productId: ProductId,
    requested: number,
    available: number
  ) {
    super(
      `Insufficient inventory for product ${productId}. Requested: ${requested}, Available: ${available}`,
      { productId: productId.value, requested, available }
    );
  }
}

// ❌ BAD - Generic error
class InventoryError extends Error {
  // Too generic, no specific context
}

// ✅ GOOD - Proper error hierarchy
abstract class PaymentError extends DomainError {
  abstract readonly code: 'PAYMENT_FAILED' | 'PAYMENT_DECLINED' | 'INVALID_CARD';
}

class PaymentDeclinedError extends PaymentError {
  readonly code = 'PAYMENT_DECLINED';
  
  constructor(
    reason: string,
    declineCode?: string
  ) {
    super(`Payment declined: ${reason}`, { reason, declineCode });
  }
}
```

### 6.3 Common Domain Error Mistakes to Avoid

| Mistake | Correction |
|---------|------------|
| Using `Error` instead of `DomainError` | Create specific domain error types |
| Missing context in errors | Include relevant identifiers and values |
| Business logic errors with generic messages | Use business language, not technical terms |
| Not documenting error codes | Document all error codes and their meanings |
| Exposing internal errors to users | Translate internal errors to user-friendly messages |

---

## 7. Error Code Registry

| Code | Status | Description | HTTP Status |
|------|--------|-------------|-------------|
| `REQUIRED_FIELD` | 400 | Missing required field | 400 |
| `INVALID_FORMAT` | 400 | Field format is invalid | 400 |
| `VALUE_OUT_OF_RANGE` | 400 | Value outside allowed range | 400 |
| `ENTITY_NOT_FOUND` | 404 | Entity does not exist | 404 |
| `DUPLICATE_ENTITY` | 409 | Entity already exists | 409 |
| `INVALID_STATE_TRANSITION` | 422 | Invalid state change | 422 |
| `BUSINESS_RULE_VIOLATION` | 422 | Business rule not satisfied | 422 |
| `INSUFFICIENT_INVENTORY` | 422 | Not enough inventory | 422 |
| `INSUFFICIENT_FUNDS` | 422 | Not enough funds | 422 |
| `CONCURRENT_MODIFICATION` | 409 | Optimistic lock failure | 409 |
| `ACCOUNT_LOCKED` | 403 | Account is locked | 403 |
| `INVALID_CREDENTIALS` | 401 | Wrong credentials | 401 |

---

## 8. References

1. doc/design-by-contract.md - Design by contract patterns
2. ref/CONSTITUTION.md - Error handling guidelines
3. ref/ai_agent_development_guidelines.md - Domain error patterns
4. Martin Fowler. "Domain Primitive"
5. "Build error-handling classes for DDD and Clean Architecture with Typescript" - GitConnected

---

## Appendix A: Error Code Registry

### A.1 Client Error Codes (4xx)

| Code | HTTP Status | Description | Example |
|------|-------------|-------------|---------|
| `REQUIRED_FIELD` | 400 | Missing required field | `{"field": "email"}` |
| `INVALID_FORMAT` | 400 | Field format is invalid | `{"field": "email", "expected": "email"}` |
| `VALUE_OUT_OF_RANGE` | 400 | Value outside allowed range | `{"field": "age", "min": 0, "max": 150}` |
| `INVALID_DATE_FORMAT` | 400 | Date format is invalid | `{"field": "date", "expected": "YYYY-MM-DD"}` |
| `INVALID_UUID` | 400 | UUID format is invalid | `{"field": "id", "expected": "UUID v4"}` |
| `ENTITY_NOT_FOUND` | 404 | Entity does not exist | `{"entity": "User", "id": "uuid"}` |
| `DUPLICATE_ENTITY` | 409 | Entity already exists | `{"entity": "User", "field": "email"}` |
| `INVALID_STATE_TRANSITION` | 422 | Invalid state change | `{"entity": "Order", "current": "PAID", "requested": "DRAFT"}` |
| `BUSINESS_RULE_VIOLATION` | 422 | Business rule not satisfied | `{"rule": "max_items_per_order", "value": 100}` |
| `INSUFFICIENT_INVENTORY` | 422 | Not enough inventory | `{"product": "id", "requested": 10, "available": 5}` |
| `INSUFFICIENT_FUNDS` | 422 | Not enough funds | `{"account": "id", "requested": 100, "available": 50}` |
| `CONCURRENT_MODIFICATION` | 409 | Optimistic lock failure | `{"entity": "Order", "version": 5}` |
| `ACCOUNT_LOCKED` | 403 | Account is locked | `{"reason": "too_many_failed_attempts"}` |
| `INVALID_CREDENTIALS` | 401 | Wrong credentials | `{"field": "password"}` |
| `EXPIRED_TOKEN` | 401 | Token has expired | `{"tokenType": "access"}` |
| `INVALID_TOKEN` | 401 | Token is invalid | `{"reason": "malformed"}` |
| `PERMISSION_DENIED` | 403 | User lacks permission | `{"required": "admin", "current": "user"}` |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests | `{"retryAfter": 60}` |

### A.2 Server Error Codes (5xx)

| Code | HTTP Status | Description | Example |
|------|-------------|-------------|---------|
| `INTERNAL_ERROR` | 500 | Unexpected server error | `{"phase": "database_query"}` |
| `DATABASE_ERROR` | 500 | Database operation failed | `{"operation": "SELECT", "table": "users"}` |
| `EXTERNAL_SERVICE_ERROR` | 502 | External service failed | `{"service": "payment_gateway", "status": 503}` |
| `TIMEOUT` | 504 | Operation timed out | `{"operation": "long_query", "timeout": 30}` |
| `SERVICE_UNAVAILABLE` | 503 | Service is unavailable | `{"reason": "maintenance"}` |

---

## Appendix B: Error Handling Checklist

### B.1 Domain Error Checklist

- [ ] Domain errors extend base `DomainError` class
- [ ] Each error has unique error code
- [ ] Error messages are user-friendly
- [ ] Context is included for debugging
- [ ] Error codes are documented
- [ ] Error hierarchy is logical
- [ ] No generic `Error` types used in domain
- [ ] System errors are separated from domain errors

### B.2 Error Code Checklist

- [ ] Error codes are unique
- [ ] Error codes follow naming convention (SCREAMING_SNAKE_CASE)
- [ ] HTTP status mapping is correct
- [ ] Error codes are versioned if API changes
- [ ] Client and server errors are distinguished

### B.3 Error Handling in Layers

| Layer | Responsibility |
|-------|----------------|
| **Domain** | Throw domain-specific exceptions |
| **Application** | Catch domain errors, convert to output |
| **Infrastructure** | Catch system errors, translate to domain errors |
| **Presentation** | Convert errors to user-friendly responses |

---

## Appendix C: Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Generic exceptions | Cannot distinguish error types | Use specific error classes |
| Missing error context | Hard to debug | Include relevant identifiers |
| Technical errors in domain | Violates clean architecture | Use domain-specific errors |
| Exposing internals | Security risk | Translate to user-friendly messages |
| Silent failures | Bugs go unnoticed | Always log or return errors |
| Inconsistent error formats | API confusion | Standardize error response |
