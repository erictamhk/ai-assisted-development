# TypeScript Conventions

This document defines TypeScript coding conventions for AI-assisted development, focusing on type safety, code quality, and AI-specific patterns.

---

## TypeScript Strict Mode

Always enable strict mode in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "useUnknownInCatchVariables": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": false
  }
}
```

**AI Guidelines:**
- Never use `any` - use `unknown` if type is uncertain
- Use `noUncheckedIndexedAccess` for safer array/object access
- Always specify explicit return types for functions

---

## Naming Conventions

### General Principles

| Type | Convention | Example |
|------|------------|---------|
| Variables | camelCase | `userName`, `orderTotal` |
| Constants | UPPER_SNAKE_CASE or camelCase | `MAX_RETRY_COUNT`, `defaultTimeout` |
| Functions | camelCase | `getUserById()`, `calculateTotal()` |
| Classes | PascalCase | `OrderService`, `UserRepository` |
| Interfaces | PascalCase | `User`, `OrderRepository` |
| Type Aliases | PascalCase | `UserId`, `OrderStatus` |
| Enums | PascalCase | `OrderStatus`, `UserRole` |
| Generics | PascalCase with T prefix | `T`, `TResult`, `TKey` |

### TypeScript-Specific Naming

```typescript
// Interfaces (preferred for objects)
interface User {
  id: UserId;
  name: string;
  email: Email;
}

// Type aliases (for unions, intersections, primitives)
type UserId = string & { readonly brand: unique symbol };
type OrderStatus = 'pending' | 'processing' | 'completed' | 'cancelled';

// Generics
function findById<T extends Entity<TId>, TId>(id: TId): T | null {
  return repository.findById(id);
}
```

### Private Members

```typescript
// ✅ CORRECT - Underscore prefix for private fields
class OrderService {
  private readonly _repository: OrderRepository;
  private _defaultTimeout: number = 5000;

  constructor(repository: OrderRepository) {
    this._repository = repository;
  }
}

// ✅ ALSO CORRECT - Using TypeScript #private
class OrderService {
  #repository: OrderRepository;
  #defaultTimeout = 5000;
}
```

---

## Type Annotations

### Explicit Types Required

```typescript
// ❌ WRONG - Implicit any
function processUser(user) {
  return user.name;
}

// ✅ CORRECT - Explicit types
function processUser(user: User): string {
  return user.name;
}

// ❌ WRONG - Implicit any in variables
const data = getData();

// ✅ CORRECT - Explicit types
const data: UserResponse = await getData();
const users: User[] = [];
const config: Readonly<Config> = { ... };
```

### Prefer Interfaces over Type Aliases for Objects

```typescript
// ✅ PREFERRED - Interface for objects
interface User {
  id: UserId;
  name: string;
  email: Email;
  role: UserRole;
}

// ✅ ACCEPTABLE - Type alias for unions/primitives
type UserStatus = 'active' | 'inactive' | 'pending';
type UserId = string & { readonly brand: unique symbol };
type UserEvent = 
  | { type: 'created'; userId: UserId }
  | { type: 'updated'; userId: UserId; changes: Partial<User> };
```

### Use Readonly for Immutable Data

```typescript
interface User {
  readonly id: UserId;
  readonly name: string;
  email: string; // Mutable allowed
}

function processOrder(order: Readonly<Order>): void {
  // Cannot modify order properties
}

const users: ReadonlyArray<User> = [];
const config: Readonly<Config> = {};
```

---

## Function Signatures

### Explicit Return Types

```typescript
// ✅ CORRECT - Explicit return type
async function getUserById(id: UserId): Promise<User | null> {
  return repository.findById(id);
}

// ✅ CORRECT - Function expression with type
const calculateTotal: (items: OrderItem[]) => Money = (items) => {
  return items.reduce((sum, item) => sum.add(item.price), Money.zero());
};

// ✅ CORRECT - Predicate functions
function isActive(user: User): boolean {
  return user.status === 'active';
}
```

### Optional Parameters and Overloads

```typescript
// ✅ CORRECT - Optional parameters with default
function createOrder(customerId: CustomerId, items?: OrderItem[]): Order {
  const orderItems = items ?? [];
  return new Order(customerId, orderItems);
}

// ✅ CORRECT - Function overloads for complex signatures
function findUsers(filter: { status: UserStatus }): User[];
function findUsers(filter: { email: string }): User | null;
function findUsers(filter: { status?: UserStatus; email?: string }): User[] | User | null {
  // Implementation
}
```

---

## Error Handling

### Typed Errors

```typescript
// ✅ CORRECT - Specific error types
class UserNotFoundError extends Error {
  constructor(public readonly userId: UserId) {
    super(`User not found: ${userId}`);
    this.name = 'UserNotFoundError';
  }
}

// ✅ CORRECT - Try-catch with type narrowing
async function getUser(id: UserId): Promise<User> {
  try {
    return await repository.findById(id).orElseThrow(
      () => new UserNotFoundError(id)
    );
  } catch (error) {
    if (error instanceof UserNotFoundError) {
      throw error;
    }
    throw new UnexpectedError('Failed to fetch user', { cause: error });
  }
}
```

### Result Type Pattern

```typescript
// ✅ CORRECT - Result/Output pattern
interface Result<T> {
  isSuccess(): boolean;
  isFailure(): boolean;
  getValue(): T;
  getError(): Error | null;
}

class ResultImpl<T> implements Result<T> {
  // ... implementation
}
```

---

## Generics

### Generic Constraints

```typescript
// ✅ CORRECT - Generic with constraints
interface Repository<T extends Entity<TId>, TId extends EntityId> {
  findById(id: TId): Promise<T | null>;
  save(entity: T): Promise<void>;
}

// ✅ CORRECT - Multiple generics
function mapArray<T, U>(
  array: ReadonlyArray<T>,
  mapper: (item: T, index: number) => U
): U[] {
  return array.map(mapper);
}
```

---

## Async/Await Patterns

### Always Use Async/Await

```typescript
// ❌ WRONG - Promise.then
function getUser(id: UserId): Promise<User> {
  return repository.findById(id).then(user => {
    if (!user) throw new Error('Not found');
    return user;
  });
}

// ✅ CORRECT - Async/await
async function getUser(id: UserId): Promise<User> {
  const user = await repository.findById(id);
  if (!user) throw new UserNotFoundError(id);
  return user;
}
```

### Never Return Mixed Sync/Async

```typescript
// ❌ WRONG - Inconsistent return type
function findUser(id: UserId): User | Promise<User> {
  if (cache.has(id)) return cache.get(id)!;
  return repository.findById(id);
}

// ✅ CORRECT - Always async
function findUser(id: UserId): Promise<User | null> {
  if (cache.has(id)) return Promise.resolve(cache.get(id)!);
  return repository.findById(id);
}
```

---

## Type Guards and Narrowing

### Use Type Guards

```typescript
// ✅ CORRECT - Type guard function
function isUser(value: unknown): value is User {
  return isObject(value) && 
         hasProperty(value, 'id') && 
         hasProperty(value, 'email');
}

// ✅ CORRECT - Using type guards
if (isUser(value)) {
  // value is narrowed to User
  console.log(value.email);
}
```

---

## Import/Export Patterns

### Named Exports Preferred

```typescript
// ✅ CORRECT - Named exports
export interface UserRepository {
  findById(id: UserId): Promise<User | null>;
}

export class UserRepositoryImpl implements UserRepository {
  // implementation
}

// ✅ CORRECT - Barrel exports (index.ts)
export * from './user.repository';
export { UserRepositoryImpl } from './user.repository';
```

### Type-Only Imports

```typescript
// ✅ CORRECT - Type-only imports
import type { User, UserId, UserRepository } from './types';
import { db } from './database';
```

---

## AI-Specific Guidelines

When AI generates TypeScript code:

1. **Always use strict mode** - Never disable type checking
2. **Never use `any`** - Use `unknown` or specific types
3. **Always specify return types** - Especially for async functions
4. **Prefer interfaces for objects** - Use types for unions/primitives
5. **Use readonly** - Mark immutable properties as readonly
6. **Avoid enums** - Use union types or const objects instead
7. **Use generic constraints** - Avoid over-generic types
8. **Handle all cases** - Use exhaustive switch statements with never
9. **Use type guards** - For runtime type checking
10. **Use type-only imports** - When importing types only

---

## Code Examples

### Good Example

```typescript
export type UserId = string & { readonly brand: unique symbol };

export interface User {
  readonly id: UserId;
  readonly name: string;
  readonly email: Email;
  status: UserStatus;
}

export type UserStatus = 'active' | 'inactive' | 'pending';

export interface UserRepository {
  findById(id: UserId): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: UserId): Promise<void>;
}

export class UserService {
  constructor(
    private readonly _repository: UserRepository,
    private readonly _emailService: EmailService
  ) {}

  async activateUser(id: UserId): Promise<void> {
    const user = await this._repository.findById(id);
    if (!user) throw new UserNotFoundError(id);
    
    const updatedUser = { ...user, status: 'active' as const };
    await this._repository.save(updatedUser);
  }
}
```

### Bad Example (What AI Should Avoid)

```typescript
// ❌ WRONG - Using any, implicit types, no return type
function getUser(id: any): any {
  return db.query('SELECT * FROM users WHERE id = ?', [id]);
}

// ❌ WRONG - Mutating readonly properties
class User {
  id: string;
  constructor(id: string) {
    this.id = id;
  }
}

// ❌ WRONG - Missing error handling
async function saveUser(user: any): Promise<void> {
  await db.save(user);
}
```

---

## References

1. Google TypeScript Style Guide: https://google.github.io/styleguide/tsguide.html
2. TypeScript Deep Dive: https://basarat.gitbook.io/typescript/
3. TypeScript Handbook: https://www.typescriptlang.org/docs/
