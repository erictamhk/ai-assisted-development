# Result Pattern

This document defines the Result pattern for error handling in AI-assisted development, providing type-safe alternatives to exceptions for expected failure cases.

---

## 1. Why Result Pattern

### 1.1 Problems with Exceptions

| Issue | Exception Pattern | Result Pattern |
|-------|------------------|----------------|
| **Expected errors invisible** | Can't tell expected from unexpected | Explicit in return type |
| **Error suppression** | Empty catch blocks common | Errors must be handled |
| **Type inference** | Errors lost in catch | Full type safety |
| **Async error handling** | try/catch in promises | Composable operations |
| **Checked exceptions** | Verbose, often bypassed | Type-level error tracking |

```typescript
// ❌ BAD - Exceptions hide expected failures
async function getUser(id: string): Promise<User> {
  const user = await db.findById(id);
  if (!user) {
    throw new Error('User not found'); // Expected error!
  }
  return user;
}

// ✅ GOOD - Result makes failure explicit
async function getUser(id: string): Promise<Result<User, UserNotFoundError>> {
  const user = await db.findById(id);
  if (!user) {
    return failure(new UserNotFoundError(id));
  }
  return success(user);
}
```

### 1.2 When to Use Result vs Exceptions

| Use Result for | Use Exceptions for |
|----------------|-------------------|
| Expected business failures | Unexpected technical errors |
| Validation failures | Contract violations |
| Not-found scenarios | Unrecoverable states |
| Input validation | Programming mistakes |
| Domain rule violations | System failures |

---

## 2. Result Type Definition

### 2.1 Basic Result Structure

```typescript
// Core Result types
type Success<T> = {
  success: true;
  value: T;
};

type Failure<E extends Error = Error> = {
  success: false;
  error: E;
};

type Result<T, E extends Error = Error> = Success<T> | Failure<E>;

// Alternative tagged union (preferred for pattern matching)
type Result<T, E extends Error = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

### 2.2 Result Class Implementation

```typescript
class Result<T, E extends Error = Error> {
  private constructor(
    private readonly _ok: boolean,
    private readonly _value: T,
    private readonly _error: E | null
  ) {}

  // Static constructors
  static success<T>(value: T): Result<T, never> {
    return new Result<T, never>(true, value, null);
  }

  static failure<E extends Error>(error: E): Result<never, E> {
    return new Result<never, E>(false, null, error);
  }

  static fromNullable<T, E extends Error>(
    value: T | null | undefined,
    error: E
  ): Result<T, E> {
    return value !== null && value !== undefined
      ? Result.success(value)
      : Result.failure(error);
  }

  static try<T, E extends Error = Error>(
    fn: () => T,
    errorMapper?: (error: unknown) => E
  ): Result<T, E> {
    try {
      return Result.success(fn());
    } catch (error) {
      const mappedError = errorMapper
        ? errorMapper(error)
        : new Error(error instanceof Error ? error.message : 'Unknown error') as E;
      return Result.failure(mappedError);
    }
  }

  // Instance methods
  get isSuccess(): boolean {
    return this._ok;
  }

  get isFailure(): boolean {
    return !this._ok;
  }

  get value(): T {
    if (!this._ok) {
      throw new Error('Cannot get value from failure result');
    }
    return this._value;
  }

  get error(): E {
    if (this._ok) {
      throw new Error('Cannot get error from success result');
    }
    return this._error!;
  }

  // Pattern matching
  match<U>(handlers: {
    success: (value: T) => U;
    failure: (error: E) => U;
  }): U {
    return this._ok ? handlers.success(this._value) : handlers.failure(this._error!);
  }

  // Unwrap methods
  unwrap(): T {
    if (this._ok) return this._value;
    throw this._error;
  }

  unwrapOr(defaultValue: T): T {
    return this._ok ? this._value : defaultValue;
  }

  unwrapOrElse(fn: (error: E) => T): T {
    return this._ok ? this._value : fn(this._error!);
  }

  // Mapping
  map<U>(fn: (value: T) => U): Result<U, E> {
    return this._ok
      ? Result.success(fn(this._value))
      : Result.failure(this._error!);
  }

  mapError<F extends Error>(fn: (error: E) => F): Result<T, F> {
    return this._ok
      ? Result.success(this._value)
      : Result.failure(fn(this._error!));
  }

  // Chaining
  flatMap<U, F extends Error>(
    fn: (value: T) => Result<U, F>
  ): Result<U, F> {
    return this._ok ? fn(this._value) : Result.failure(this._error!);
  }

  // Async variants
  async mapAsync<U>(fn: (value: T) => Promise<U>): Promise<Result<U, E>> {
    if (this._ok) {
      const value = await fn(this._value);
      return Result.success(value);
    }
    return Result.failure(this._error!);
  }

  async flatMapAsync<U, F extends Error>(
    fn: (value: T) => Promise<Result<U, F>>
  ): Promise<Result<U, F>> {
    if (this._ok) {
      return fn(this._value);
    }
    return Result.failure(this._error!);
  }
}
```

### 2.3 Type Guards

```typescript
function isSuccess<T, E extends Error>(result: Result<T, E>): result is Success<T> {
  return result.isSuccess;
}

function isFailure<T, E extends Error>(result: Result<T, E>): result is Failure<E> {
  return result.isFailure;
}

// Usage with type narrowing
const result = getUser('123');
if (isSuccess(result)) {
  // TypeScript knows result is Success<User>
  console.log(result.value.name);
} else {
  // TypeScript knows result is Failure<UserNotFoundError>
  console.error(result.error.message);
}
```

---

## 3. Working with Results

### 3.1 Creating Results

```typescript
// From nullable values
const user = Result.fromNullable(
  database.findById(id),
  new UserNotFoundError(id)
);

// From try-catch
const parseResult = Result.try(
  () => JSON.parse(jsonString),
  (error) => new ParseError('Invalid JSON', error)
);

// From async operations
async function fetchUser(id: string): Promise<Result<User, NetworkError>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      return Result.failure(new NetworkError(response.statusText));
    }
    const user = await response.json();
    return Result.success(user);
  } catch (error) {
    return Result.failure(new NetworkError('Request failed', error));
  }
}
```

### 3.2 Pattern Matching

```typescript
// Simple match
const result = getUser('123');
result.match({
  success: (user) => {
    console.log(`Found user: ${user.name}`);
    return user;
  },
  failure: (error) => {
    console.error(`Failed: ${error.message}`);
    return null;
  }
});

// With return types
const displayName = result.match({
  success: (user) => user.displayName,
  failure: () => 'Anonymous',
});
```

### 3.3 Mapping and Chaining

```typescript
// Transform success value
const upperName = userResult
  .map((user) => user.name.toUpperCase())
  .unwrapOr('UNKNOWN');

// Chain dependent operations
const userWithAddress = await userResult.flatMapAsync((user) =>
  addressService.getByUserId(user.id)
);

// Handle errors
const displayResult = userResult
  .map((user) => user.name)
  .mapError((error) => new DisplayError('User unavailable', error))
  .unwrapOr('Guest');
```

### 3.4 Combining Results

```typescript
// Combine two results
function combineResults<T1, T2, E extends Error>(
  result1: Result<T1, E>,
  result2: Result<T2, E>
): Result<[T1, T2], E> {
  return result1.flatMap((value1) =>
    result2.map((value2) => [value1, value2] as [T1, T2])
  );
}

// Combine array of results
function all<T, E extends Error>(
  results: Result<T, E>[]
): Result<T[], E> {
  const successes: T[] = [];
  for (const result of results) {
    if (result.isFailure) {
      return Result.failure(result.error);
    }
    successes.push(result.value);
  }
  return Result.success(successes);
}

// First success or last failure
function firstSuccess<T, E extends Error>(
  results: Result<T, E>[]
): Result<T, E> {
  for (const result of results) {
    if (result.isSuccess) {
      return result;
    }
  }
  return results[results.length - 1] || Result.failure(
    new Error('No results provided')
  );
}
```

---

## 4. Async Result Patterns

### 4.1 Async Result Wrapper

```typescript
class AsyncResult<T, E extends Error = Error> {
  constructor(private promise: Promise<Result<T, E>>) {}

  static from<T, E extends Error>(
    promise: Promise<T>,
    errorMapper?: (error: unknown) => E
  ): AsyncResult<T, E> {
    return new AsyncResult(
      promise
        .then((value) => Result.success<T, E>(value))
        .catch((error) => {
          const mappedError = errorMapper
            ? errorMapper(error)
            : new Error(error instanceof Error ? error.message : 'Unknown error') as E;
          return Result.failure(mappedError);
        })
    );
  }

  // Instance methods mirror sync Result
  get isSuccess(): Promise<boolean> {
    return this.promise.then((r) => r.isSuccess);
  }

  async match<U>(handlers: {
    success: (value: T) => U;
    failure: (error: E) => U;
  }): Promise<U> {
    const result = await this.promise;
    return result.match(handlers);
  }

  async map<U>(fn: (value: T) => U): Promise<AsyncResult<U, E>> {
    const mapped = await this.promise.then((r) => r.map(fn));
    return new AsyncResult(mapped as Promise<Result<U, E>>);
  }

  async flatMap<U, F extends Error>(
    fn: (value: T) => AsyncResult<U, F>
  ): Promise<AsyncResult<U, E | F>> {
    const result = await this.promise;
    if (result.isFailure) {
      return new AsyncResult(Result.failure(result.error) as Result<U, E | F>);
    }
    return fn(result.value);
  }

  async unwrap(): Promise<T> {
    const result = await this.promise;
    return result.unwrap();
  }
}
```

### 4.2 Parallel Async Operations

```typescript
async function parallelFetch<T>(
  operations: Array<() => Promise<Result<T, Error>>>
): Promise<Result<T[], Error[]>> {
  const results = await Promise.all(
    operations.map(async (op) => {
      try {
        return await op();
      } catch (error) {
        return Result.failure(error as Error);
      }
    })
  );

  const errors = results.filter((r) => r.isFailure);
  if (errors.length > 0) {
    return Result.failure(errors.map((r) => r.error));
  }
  return Result.success(results.map((r) => r.value));
}

// Usage
const [users, orders, products] = await parallelFetch([
  () => fetchUsers(),
  () => fetchOrders(),
  () => fetchProducts(),
]).then((r) => r.unwrapOr([[], [], []]));
```

---

## 5. Result in Domain Layer

### 5.1 Value Object Creation with Result

```typescript
class Email {
  static create(value: string): Result<Email, InvalidEmailError> {
    if (!value || value.trim().length === 0) {
      return Result.failure(new InvalidEmailError('Email cannot be empty'));
    }

    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) {
      return Result.failure(new InvalidEmailError('Invalid email format'));
    }

    return Result.success(new Email(value.toLowerCase()));
  }

  // Private constructor - must use factory
  private constructor(public readonly value: string) {}
}

// Usage
const emailResult = Email.create('user@example.com');

emailResult.match({
  success: (email) => {
    // email is Email type
    console.log(`Created: ${email.value}`);
  },
  failure: (error) => {
    console.error(`Failed: ${error.message}`);
  },
});
```

### 5.2 Entity Factory with Result

```typescript
class User {
  private constructor(
    public readonly id: UserId,
    public readonly email: Email,
    public readonly name: string
  ) {}

  static create(input: CreateUserInput): Result<User, UserCreationError> {
    // Validate email
    const emailResult = Email.create(input.email);
    if (emailResult.isFailure) {
      return Result.failure(
        new UserCreationError('Invalid email', emailResult.error)
      );
    }

    // Validate name
    if (!input.name || input.name.trim().length < 2) {
      return Result.failure(
        new UserCreationError('Name must be at least 2 characters')
      );
    }

    // Generate ID and create user
    const id = UserId.generate();
    return Result.success(
      new User(id, emailResult.value, input.name.trim())
    );
  }
}
```

### 5.3 Repository Methods Return Result

```typescript
interface UserRepository {
  findById(id: UserId): Promise<Result<User, UserNotFoundError>>;
  findByEmail(email: Email): Promise<Result<User, UserNotFoundError>>;
  save(user: User): Promise<Result<void, RepositoryError>>;
  delete(id: UserId): Promise<Result<void, RepositoryError>>;
}

// Usage
const userResult = await repository.findById(userId);

if (userResult.isSuccess) {
  const user = userResult.value;
  // Use user
}
```

---

## 6. Result in Application Layer

### 6.1 Use Cases with Result

```typescript
interface UseCase<Input, Output, Error extends Error = Error> {
  execute(input: Input): Promise<Result<Output, Error>>;
}

class CreateOrderUseCase implements UseCase<CreateOrderInput, Order, OrderCreationError> {
  constructor(
    private orderRepository: OrderRepository,
    private inventoryService: InventoryService,
    private eventPublisher: EventPublisher
  ) {}

  async execute(input: CreateOrderInput): Promise<Result<Order, OrderCreationError>> {
    // Validate input
    if (input.items.length === 0) {
      return Result.failure(
        new OrderCreationError('Order must have at least one item')
      );
    }

    // Check inventory for all items
    const inventoryCheck = await this.checkInventory(input.items);
    if (inventoryCheck.isFailure) {
      return Result.failure(inventoryCheck.error);
    }

    // Create order
    const orderResult = Order.create({
      customerId: input.customerId,
      items: inventoryCheck.value,
    });

    if (orderResult.isFailure) {
      return Result.failure(
        new OrderCreationError('Invalid order', orderResult.error)
      );
    }

    const order = orderResult.value;

    // Save
    const saveResult = await this.orderRepository.save(order);
    if (saveResult.isFailure) {
      return Result.failure(
        new OrderCreationError('Failed to save order', saveResult.error)
      );
    }

    // Publish event
    this.eventPublisher.publish(new OrderCreatedEvent(order.id));

    return Result.success(order);
  }

  private async checkInventory(
    items: OrderItemInput[]
  ): Promise<Result<ValidatedItem[], InsufficientInventoryError>> {
    // Implementation
  }
}
```

### 6.2 Result in Controller

```typescript
class OrderController {
  constructor(private createOrderUseCase: CreateOrderUseCase) {}

  @Post('/orders')
  async createOrder(req: Request): Promise<Response> {
    const input = CreateOrderInput.fromRequest(req.body);
    const result = await this.createOrderUseCase.execute(input);

    return result.match({
      success: (order) => Response.created({ orderId: order.id }),
      failure: (error) => {
        const status = this.mapErrorToStatus(error);
        return Response.error(status, {
          code: error.code,
          message: error.message,
        });
      },
    });
  }

  private mapErrorToStatus(error: OrderCreationError): number {
    switch (error.code) {
      case 'INVALID_INPUT':
        return 400;
      case 'INSUFFICIENT_INVENTORY':
        return 422;
      default:
        return 500;
    }
  }
}
```

---

## 7. AI-Specific Guidelines

### 7.1 When to Generate Result Types

1. **Always use Result for expected failures**
   - User not found
   - Validation errors
   - Business rule violations
   - Duplicate entities

2. **Never throw exceptions for expected failures**
   - Use `Result.failure()` instead

3. **Use exceptions for unexpected errors only**
   - Database connection failures
   - Network timeouts
   - Out of memory conditions

### 7.2 Result Generation Template

```typescript
// AI-generated domain operation with Result
async function [operation]([params]): Promise<Result<[ReturnType], [DomainError]>> {
  // Validate inputs
  const [validation] = validate([input]);
  if ([validationFailed]) {
    return Result.failure(new [ValidationError]([details]));
  }

  // Perform operation
  try {
    const [result] = await [operation]();
    return Result.success([result]);
  } catch (error) {
    // Only catch unexpected errors
    if (error instanceof [DomainError]) {
      return Result.failure(error);
    }
    return Result.failure(new [UnexpectedError]('Operation failed', { originalError: error }));
  }
}
```

### 7.3 Common Mistakes to Avoid

| Mistake | Correction |
|---------|------------|
| Returning `Result` but throwing exceptions | Always use `Result.failure()` |
| Generic error types | Use specific domain error types |
| Missing error context | Include relevant identifiers |
| Ignoring Result in callers | Use pattern matching or unwrap |
| Mixing exceptions with Result | Pick one pattern per operation |

---

## 8. Integration with Existing Patterns

### 8.1 Result with Either Libraries

```typescript
// Using fp-ts Either
import { either } from 'fp-ts';

function parseJson(s: string): Either<ParseError, unknown> {
  return either.tryCatch(
    () => JSON.parse(s),
    (reason) => new ParseError('Invalid JSON', reason as Error)
  );
}

// Using neverthrow
import { Ok, Err } from 'neverthrow';

function divide(a: number, b: number): Result<number, DivisionByZeroError> {
  if (b === 0) {
    return Err(new DivisionByZeroError());
  }
  return Ok(a / b);
}
```

### 8.2 Result with Functional Programming

```typescript
// Pipeline with Result
const result = pipe(
  parseUserInput(input),
  Result.flatMap(validateUser),
  Result.flatMap(saveUser),
  Result.map(user => user.id)
);

// Effects with Result (effect-ts pattern)
class UserService extends Effect.Service<UserService>()('UserService', {
  effects: [
    findUser: (id: UserId) => Effect.try({
      try: () => db.findById(id),
      catch: () => new UserNotFoundError(id)
    })
  ]
})
```

---

## 9. References

1. doc/clean-code.md - Error handling patterns
2. doc/vertical-slice-architecture-megred.md - Result pattern usage
3. ref/CONSTITUTION.md - Error handling conventions
4. "Result types for TypeScript" - Matt Pocock
5. "typescript-result" library - GitHub
6. "neverthrow" library - GitHub
7. "fp-ts" library - Functional programming in TypeScript

---

## Appendix A: Result Pattern Variations

### A.1 Simple Result (Language-Level)

```typescript
// For languages without generics or with limited type systems
interface Result<T> {
  isSuccess: boolean;
  value?: T;
  error?: string;
}

// Usage
const result: Result<User> = { isSuccess: true, value: user };
```

### A.2 Rust-Style Result

```typescript
// Inspired by Rust's Result<T, E>
type Result<T, E = never> = 
  | { ok: true; value: T }
  | { ok: false; error: E };

// Pattern matching
function unwrap<T, E>(result: Result<T, E>): T {
  if (result.ok) return result.value;
  throw new Error(result.error);
}
```

### A.3 Either Pattern (Functional)

```typescript
// Left (error) and Right (success) - common in FP
type Either<L, R> = Left<L> | Right<R>;

class Left<L> {
  readonly _tag = 'Left';
  constructor(readonly value: L) {}
}

class Right<R> {
  readonly _tag = 'Right';
  constructor(readonly value: R) {}
}

const left = <L, R>(value: L) => new Left(value);
const right = <L, R>(value: R) => new Right(value);

// Map and flatMap
const map = <L, A, B>(ea: Either<L, A>, f: (a: A) => B): Either<L, B> =>
  ea._tag === 'Right' ? right(f(ea.value)) : ea;

const flatMap = <L, A, B, M>(ea: Either<L, A>, f: (a: A) => Either<M, B>): Either<L | M, B> =>
  ea._tag === 'Right' ? f(ea.value) : ea;
```

### A.4 Maybe Pattern (Null Handling)

```typescript
// For operations that might return null/undefined
type Maybe<T> = Some<T> | None;

class Some<T> {
  readonly _tag = 'Some';
  constructor(readonly value: T) {}
}

class None {
  readonly _tag = 'None';
}

// Usage with Result
type Result<T, E> = Either<E, T>;  // Error on Left, Success on Right
```

---

## Appendix B: Integration Patterns

### B.1 Result with Async/Await

```typescript
// Promise-based Result
type AsyncResult<T, E = Error> = Promise<Result<T, E>>;

async function findUser(id: string): AsyncResult<User, UserNotFoundError> {
  try {
    const user = await database.findById(id);
    if (!user) {
      return { ok: false, error: new UserNotFoundError(id) };
    }
    return { ok: true, value: user };
  } catch (error) {
    return { ok: false, error: new DatabaseError(error.message) };
  }
}

// Usage with await
const result = await findUser('user-123');
if (result.ok) {
  console.log('Found:', result.value.name);
} else {
  console.error('Error:', result.error.message);
}
```

### B.2 Result with Streams

```typescript
// Processing a stream of Results
async function processUsers(userIds: string[]): Promise<void> {
  for await (const result of streamUserResults(userIds)) {
    if (result.ok) {
      await notifyUser(result.value);
    } else {
      logError(result.error);
    }
  }
}

// Batch processing with Result aggregation
async function processBatch<T>(
  items: T[],
  processor: (item: T) => Promise<Result<void, Error>>
): Promise<Result<void[], Error>> {
  const results = await Promise.all(items.map(processor));
  
  const failures = results.filter(r => !r.ok);
  if (failures.length > 0) {
    return { ok: false, error: new BatchProcessingError(failures.length) };
  }
  
  return { ok: true, value: undefined as void };
}
```

### B.3 Result with Retry Logic

```typescript
async function withRetry<T, E extends Error>(
  operation: () => Promise<Result<T, E>>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<Result<T, E>> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    const result = await operation();
    if (result.ok) return result;
    
    // Don't retry on domain errors
    if (result.error instanceof DomainError) {
      return result;
    }
    
    if (attempt < maxRetries) {
      await sleep(delay * attempt);
    }
  }
  
  return { ok: false, error: new RetryExhaustedError() };
}
```

---

## Appendix C: Migration Guide

### C.1 From Exceptions to Result

**Before (Exception-based):**
```typescript
async function getUser(id: string): Promise<User> {
  const user = await db.findById(id);
  if (!user) throw new NotFoundError();
  return user;
}
```

**After (Result-based):**
```typescript
async function getUser(id: string): Promise<Result<User, NotFoundError>> {
  const user = await db.findById(id);
  if (!user) return { ok: false, error: new NotFoundError() };
  return { ok: true, value: user };
}
```

### C.2 From null Returns to Maybe

**Before (null returns):**
```typescript
function findUser(id: string): User | null {
  return users.get(id) || null;
}
```

**After (Maybe pattern):**
```typescript
function findUser(id: string): Maybe<User> {
  const user = users.get(id);
  return user ? some(user) : none();
}
```

### C.3 Mixed Approach (Gradual Migration)

```typescript
// Support both patterns during migration
async function getUser(id: string): Promise<Result<User, NotFoundError>> {
  const user = await repository.findById(id);
  if (!user) return failure(new NotFoundError(id));
  return success(user);
}

// For internal calls, use Result
// For external API, convert to exception or keep as Result
```

---

## Appendix D: Performance Considerations

### D.1 Result Object Overhead

| Implementation | Memory | Performance |
|----------------|--------|-------------|
| Union type | Minimal | Fast (native) |
| Class instance | Moderate | Good |
| Tagged union | Minimal | Fast |

### D.2 Optimization Tips

1. **Use structural typing** for Result (union types) instead of classes
2. **Reuse error instances** for known errors
3. **Avoid deep nesting** of Result operations
4. **Use early returns** instead of nested matching
5. **Cache validation results** for repeated operations

---

## Appendix E: Testing Result Patterns

### E.1 Testing Success Paths

```typescript
describe('UserService', () => {
  it('should return success when user is created', async () => {
    const result = await service.createUser(validInput);
    
    expect(result.ok).toBe(true);
    expect(result.value).toBeDefined();
    expect(result.value.id).toBeDefined();
  });
});
```

### E.2 Testing Failure Paths

```typescript
it('should return error when email is duplicate', async () => {
  const result = await service.createUser(duplicateEmailInput);
  
  expect(result.ok).toBe(false);
  expect(result.error).toBeInstanceOf(DuplicateEmailError);
  expect(result.error.code).toBe('DUPLICATE_EMAIL');
});
```

### E.3 Testing Validation

```typescript
it('should return validation error for invalid input', async () => {
  const result = await service.createUser(invalidInput);
  
  expect(result.ok).toBe(false);
  expect(result.error).toBeInstanceOf(ValidationError);
  expect(result.error.context.fields).toContain('email');
});
```
