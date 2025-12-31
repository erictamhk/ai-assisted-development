# Clean Code

## Overview

Clean Code is a software development philosophy articulated by Robert C. Martin (Uncle Bob) that emphasizes writing code that is easy to understand, maintain, and extend. The core premise is that code is read far more often than it is written, and therefore readability should be a primary concern.

## Core Principles

### Meaningful Names

```typescript
// ❌ Bad: Cryptic names
function p(d: Date): number {
  return d.getTime() / 86400000;
}

// ✅ Good: Descriptive names
function daysSinceEpoch(epochDate: Date): number {
  const millisecondsPerDay = 86_400_000;
  return epochDate.getTime() / millisecondsPerDay;
}

// ❌ Bad: Ambiguous abbreviations
interface SvcCfg {
  dmn: string;
  ttl: number;
  rtry: boolean;
}

// ✅ Good: Self-explanatory names
interface ServiceConfig {
  domainName: string;
  timeToLive: number;
  shouldRetry: boolean;
}

// ❌ Bad: Magic numbers
if (user.status === 2 && user.score > 75) {
  activatePremium(user);
}

// ✅ Good: Named constants
enum UserStatus {
  ACTIVE = 1,
  INACTIVE = 2,
  SUSPENDED = 3
}

const PREMIUM_ACTIVATION_SCORE_THRESHOLD = 75;

if (user.status === UserStatus.INACTIVE && user.score > PREMIUM_ACTIVATION_SCORE_THRESHOLD) {
  activatePremium(user);
}
```

### Functions

```typescript
// ❌ Bad: Long function with multiple responsibilities
function processUserData(user: User): void {
  // Validate user
  if (!user.email || !user.name) throw new Error('Invalid user');
  
  // Save to database
  database.save(user);
  
  // Send notification
  emailService.send(user.email, 'Welcome!');
  
  // Update analytics
  analytics.track('user_created', user.id);
}

// ✅ Good: Single responsibility, composed functions
function validateUser(user: User): void {
  if (!user.email || !user.name) {
    throw new ValidationError('User must have email and name');
  }
}

async function processUserRegistration(user: User): Promise<void> {
  validateUser(user);
  
  await saveUser(user);
  await sendWelcomeEmail(user);
  await trackUserCreation(user.id);
}

async function saveUser(user: User): Promise<void> {
  await userRepository.save(user);
}

async function sendWelcomeEmail(user: User): Promise<void> {
  await emailService.send(user.email, 'Welcome!', 'welcome-template');
}

async function trackUserCreation(userId: string): Promise<void> {
  analytics.track('user_created', userId);
}

// ✅ Good: Small, focused function
function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

// ✅ Good: Parameter objects reduce parameter count
interface FetchOptions {
  url: string;
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  headers?: Record<string, string>;
  body?: unknown;
  timeout?: number;
}

async function fetchWithDefaults(options: FetchOptions): Promise<Response> {
  const response = await fetch(options.url, {
    method: options.method ?? 'GET',
    headers: options.headers ?? {},
    body: options.body ? JSON.stringify(options.body) : undefined
  });
  return response;
}
```

### Comments

```typescript
// ❌ Bad: Redundant comments
// This function calculates the total
function calculateTotal(items: Item[]): number {
  let total = 0; // Start with zero
  for (const item of items) { // Loop through items
    total += item.price; // Add each price
  }
  return total; // Return the total
}

// ✅ Good: Explain "why", not "what"
// The product catalog is priced in cents to avoid floating-point errors
const PRICE_CENTS_MULTIPLIER = 100;

function calculateOrderTotal(items: OrderItem[]): Money {
  return items.reduce(
    (total, item) => total.add(item.unitPrice.multiply(item.quantity)),
    Money.zero()
  );
}

// ✅ Good: Public API documentation
/**
 * Calculates the total price for an order including applicable discounts.
 * 
 * @param items - The line items in the order
 * @param discountCode - Optional discount code to apply
 * @returns The total price as a Money object
 * @throws InvalidDiscountError if the discount code is expired or invalid
 * 
 * @example
 * ```typescript
 * const total = calculateOrderTotal([
 *   { name: 'Widget', price: Money.fromAmount(9.99) }
 * ], 'SAVE10');
 * ```
 */
function calculateOrderTotal(
  items: OrderItem[],
  discountCode?: string
): Money {
  // Implementation
}
```

### Formatting

```typescript
// ✅ Good: Consistent spacing and alignment
const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
  { id: 3, name: 'Charlie', email: 'charlie@example.com' }
];

// ✅ Good: Related concepts grouped
interface UserRegistration {
  personalInfo: {
    firstName: string;
    lastName: string;
    email: string;
  };
  address: {
    street: string;
    city: string;
    zipCode: string;
    country: string;
  };
  preferences: {
    newsletter: boolean;
    notifications: boolean;
    language: string;
  };
}

// ✅ Good: Vertical spacing between concepts
class UserService {
  // Public methods first
  async createUser(data: CreateUserInput): Promise<User> {
    const user = User.create(data);
    await this.repository.save(user);
    await this.eventBus.publish(new UserCreatedEvent(user));
    return user;
  }

  async getUser(id: string): Promise<User | null> {
    return this.repository.findById(id);
  }

  // Private methods after public
  private validateInput(data: CreateUserInput): void {
    if (!data.email || !data.name) {
      throw new ValidationError('Email and name are required');
    }
  }

  private generateUsername(email: string): string {
    return email.split('@')[0];
  }
}
```

### Error Handling

```typescript
// ❌ Bad: Silent failures and generic errors
try {
  const user = await fetchUser(id);
} catch (e) {
  console.log('Error');
  return null;
}

// ✅ Good: Specific error handling with context
async function getUserProfile(userId: string): Promise<UserProfile> {
  try {
    const user = await userRepository.findById(userId);
    if (!user) {
      throw new UserNotFoundError(`User ${userId} not found`);
    }
    
    const profile = await profileService.buildProfile(user);
    return profile;
  } catch (error) {
    if (error instanceof UserNotFoundError) {
      logger.warn('User not found', { userId });
      throw error;
    }
    if (error instanceof DatabaseConnectionError) {
      logger.error('Database connection failed', { userId, error });
      throw new ServiceUnavailableError('Profile service temporarily unavailable');
    }
    logger.error('Unexpected error fetching profile', { userId, error });
    throw new ProfileFetchError('Failed to fetch user profile', { cause: error });
  }
}

// ✅ Good: Result type for expected failures
type Result<T, E = Error> = 
  | { ok: true; value: T }
  | { ok: false; error: E };

function safeParse<T>(json: string, schema: Schema<T>): Result<T, ParseError> {
  try {
    const value = JSON.parse(json);
    const result = schema.validate(value);
    if (result.success) {
      return { ok: true, value: result.value };
    }
    return { ok: false, error: result.error };
  } catch (error) {
    return { ok: false, error: new ParseError(error.message) };
  }
}
```

### Object Orientation

```typescript
// ❌ Bad: Data classes with behavior elsewhere
class User {
  constructor(
    public id: string,
    public email: string,
    public name: string,
    public status: string
  ) {}
}

function activateUser(user: User): void {
  if (user.status === 'ACTIVE') return;
  user.status = 'ACTIVE';
  emailService.send(user.email, 'Your account is activated!');
}

function deactivateUser(user: User): void {
  user.status = 'INACTIVE';
  auditService.log(`User ${user.id} was deactivated`);
}

// ✅ Good: Encapsulated behavior
class User {
  private constructor(
    private readonly id: string,
    private readonly email: string,
    private readonly name: string,
    private status: UserStatus,
    private readonly events: UserEvent[] = []
  ) {}

  static register(input: RegisterUserInput): User {
    const user = new User(
      UserId.generate(),
      input.email,
      input.name,
      UserStatus.PENDING,
      [new UserRegisteredEvent(input.email)]
    );
    return user;
  }

  activate(): void {
    if (this.status === UserStatus.ACTIVE) {
      throw new Error('User is already active');
    }
    
    this.status = UserStatus.ACTIVE;
    this.events.push(new UserActivatedEvent(this.id));
  }

  deactivate(reason: DeactivationReason): void {
    if (this.status === UserStatus.INACTIVE) {
      throw new Error('User is already inactive');
    }
    
    this.status = UserStatus.INACTIVE;
    this.events.push(new UserDeactivatedEvent(this.id, reason));
  }

  getEvents(): UserEvent[] {
    return [...this.events];
  }
}
```

### SOLID Principles in Practice

```typescript
// Single Responsibility Principle
// ❌ Bad: Multiple responsibilities
class ReportGenerator {
  generate(data: Data): string { /* ... */ }
  saveToFile(content: string, path: string): void { /* ... */ }
  sendEmail(to: string, content: string): void { /* ... */ }
}

// ✅ Good: Single responsibility each
class ReportGenerator {
  constructor(
    private readonly templateEngine: TemplateEngine,
    private readonly dataProvider: DataProvider
  ) {}

  generate(reportType: string, parameters: Record<string, unknown>): string {
    const template = this.templateEngine.getTemplate(reportType);
    const data = this.dataProvider.fetchData(parameters);
    return template.render(data);
  }
}

class FileStorage {
  save(content: string, path: string): void { /* ... */ }
}

class EmailSender {
  send(to: string, subject: string, content: string): void { /* ... */ }
}

// Open/Closed Principle
// ✅ Good: Open for extension, closed for modification
interface DiscountStrategy {
  calculateDiscount(order: Order): Money;
}

class PercentageDiscount implements DiscountStrategy {
  constructor(private readonly percentage: number) {}

  calculateDiscount(order: Order): Money {
    return order.subtotal.multiply(this.percentage / 100);
  }
}

class FixedDiscount implements DiscountStrategy {
  constructor(private readonly amount: Money) {}

  calculateDiscount(order: Order): Money {
    return this.amount;
  }
}

class DiscountCalculator {
  constructor(private readonly strategies: DiscountStrategy[]) {}

  calculateBestDiscount(order: Order): Money {
    return this.strategies.reduce(
      (best, strategy) => {
        const discount = strategy.calculateDiscount(order);
        return discount.amount > best.amount ? discount : best;
      },
      Money.zero()
    );
  }
}

// Liskov Substitution Principle
// ✅ Good: Subtypes can replace base types
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<void>;
}

class UserRepository implements Repository<User> {
  async findById(id: string): Promise<User | null> {
    // User-specific implementation
  }

  async findAll(): Promise<User[]> {
    // User-specific implementation
  }

  async save(user: User): Promise<void> {
    // User-specific implementation
  }
}
```

## Code Review Checklist

```markdown
## Clean Code Review Checklist

### Naming
- [ ] Names reveal intent
- [ ] Names are pronounceable
- [ ] Names are searchable
- [ ] Constants have meaningful names
- [ ] No magic numbers or strings

### Functions
- [ ] Functions do one thing
- [ ] Functions are small (ideally < 20 lines)
- [ ] Functions have few parameters (< 4)
- [ ] No side effects
- [ ] Try/Error handling separated from main logic

### Comments
- [ ] Comments explain "why", not "what"
- [ ] No commented-out code
- [ ] Commented code removed
- [ ] Public API documented

### Formatting
- [ ] Consistent indentation
- [ ] Logical grouping of code
- [ ] Vertical spacing between concepts
- [ ] Line length reasonable (< 120 chars)

### Objects & Data
- [ ] Data structures have behavior
- [ ] Objects hide their internals
- [ ] Law of Demeter followed
- [ ] No public mutable fields

### Error Handling
- [ ] Errors are specific
- [ ] Errors include context
- [ ] No silent failures
- [ ] No empty catch blocks
```

## AI-Assisted Clean Code

### Common AI Coding Anti-Patterns

Based on analysis of AI-generated code patterns, these anti-patterns frequently occur and should be explicitly avoided:

```typescript
// ANTI-PATTERN: Nested Input/Output Classes
// ❌ AI creates: Separate Input.java, Output.java files
// ✅ CORRECT: Inner classes within UseCase interface
interface CreateProductUseCase {
    class CreateProductInput implements Input {
        public productId: string;
        public name: string;
        public userId: string;
    }
}

// ANTI-PATTERN: if-else instanceof Chain
// ❌ AI writes: Multiple if-else checks with instanceof
// ✅ CORRECT: Switch expression with pattern matching
when(event: ProductEvent): void {
    switch (event) {
        case ProductEvents.ProductCreated e -> {
            this.id = e.productId();
            this.name = e.name();
        }
        case ProductEvents.ProductRenamed e -> {
            this.name = e.newName();
        }
    }
}

// ANTI-PATTERN: Automatic Repository Implementation
// ❌ AI generates: Custom repository interface
// ✅ CORRECT: Use framework GenericInMemoryRepository
interface ProductRepository extends Repository<Product, ProductId> {
    List<Product> findBySprintId(SprintId id);  // WRONG: Custom query
}

// ANTI-PATTERN: Contract Validation in Value Objects
// ❌ AI uses: Contract.requireNotNull() in Value Object
// ✅ CORRECT: Use Objects.requireNonNull()
record ProductId(String value) implements ValueObject {
    public ProductId {
        Objects.requireNonNull(value, "Product ID cannot be null");
    }
}
```

### AI Prompt for Clean Code Refactoring

Refactor the following code following Clean Code principles:

```typescript
// Original code
function p(o) {
  let r = { s: [], t: 0 };
  for (let i = 0; i < o.items.length; i++) {
    let item = o.items[i];
    if (item.status === 'A' && item.qty > 0) {
      r.s.push(item);
      r.t += item.price * item.qty;
    }
  }
  return r;
}
```

Requirements:
1. Use meaningful names
2. Extract into small, focused functions
3. Add proper TypeScript types
4. Add JSDoc documentation
5. Include error handling
6. Make it testable

Output:
1. Refactored TypeScript code
2. Explanation of changes made
3. Unit tests for the function
```

### AI Code Review Checklist

When reviewing AI-generated code, use this enhanced checklist:

```markdown
## AI Code Review Checklist

### 1. Architecture Compliance
- [ ] Package location follows layer structure
- [ ] Clean Architecture boundaries respected
- [ ] Dependency direction is correct (inward)

### 2. Coding Standards
- [ ] Input class is inner class of UseCase
- [ ] Value Objects use Objects.requireNonNull()
- [ ] Repository pattern follows framework conventions
- [ ] No nested Input/Output files

### 3. Business Logic
- [ ] Contract validation in constructors
- [ ] Domain events properly applied
- [ ] Aggregate invariants enforced
- [ ] Soft delete pattern implemented

### 4. Testability
- [ ] Dependencies injected via constructor
- [ ] No static references to framework
- [ ] Can be unit tested in isolation
```

## Testability

```typescript
// ✅ Good: Testable code
class PaymentProcessor {
  constructor(
    private readonly paymentGateway: PaymentGateway,
    private readonly logger: Logger
  ) {}

  async processPayment(
    orderId: string,
    amount: Money,
    paymentMethod: PaymentMethod
  ): Promise<PaymentResult> {
    try {
      this.logger.info(`Processing payment for order ${orderId}`);
      
      const result = await this.paymentGateway.charge(
        paymentMethod,
        amount
      );

      if (result.success) {
        this.logger.info(`Payment successful for order ${orderId}`);
        return PaymentResult.success(result.transactionId);
      }

      this.logger.warn(`Payment failed for order ${orderId}: ${result.error}`);
      return PaymentResult.failure(result.error);
    } catch (error) {
      this.logger.error(`Payment error for order ${orderId}`, { error });
      return PaymentResult.error(error.message);
    }
  }
}

// ✅ Good: Dependencies injectable for testing
describe('PaymentProcessor', () => {
  let processor: PaymentProcessor;
  let mockGateway: jest.Mocked<PaymentGateway>;
  let mockLogger: jest.Mocked<Logger>;

  beforeEach(() => {
    mockGateway = {
      charge: jest.fn().mockResolvedValue({ success: true, transactionId: 'txn_123' })
    };
    mockLogger = {
      info: jest.fn(),
      warn: jest.fn(),
      error: jest.fn()
    };
    processor = new PaymentProcessor(mockGateway, mockLogger);
  });

  it('processes payment successfully', async () => {
    const result = await processor.processPayment(
      'order_123',
      Money.fromAmount(99.99),
      { type: 'credit_card', last4: '4242' }
    );

    expect(result.success).toBe(true);
    expect(mockGateway.charge).toHaveBeenCalled();
    expect(mockLogger.info).toHaveBeenCalled();
  });
});
```

## References

- "Clean Code" by Robert C. Martin
- "The Pragmatic Programmer" by Andrew Hunt and David Thomas
- "Code Complete" by Steve McConnell
- ESLint rules for code quality
