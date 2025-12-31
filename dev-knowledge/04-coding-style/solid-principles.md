# SOLID Principles

SOLID principles are fundamental design principles that make software more maintainable, scalable, and robust. These principles help create systems that are easy to understand, extend, and maintain.

## Single Responsibility Principle (SRP)

### Class Responsibility

- **One reason to change per class**
- Clear, focused purpose
- Delegating to collaborators

**Example:**

```typescript
// ❌ BAD: Violates SRP - multiple responsibilities
class UserService {
  createUser(data: UserData): User {
    // Validates input
    // Hashes password
    // Sends email
    // Saves to database
    // Returns user
  }
}

// ✅ GOOD: Follows SRP - single responsibility
class UserCreator {
  constructor(
    private validator: UserValidator,
    private passwordHasher: PasswordHasher,
    private repository: UserRepository,
    private emailService: EmailService
  ) {}

  createUser(data: UserData): User {
    this.validator.validate(data);
    const passwordHash = this.passwordHasher.hash(data.password);
    const user = User.create(data.email, passwordHash);
    this.repository.save(user);
    this.emailService.sendWelcomeEmail(user);
    return user;
  }
}
```

### Method Responsibility

- **Do one thing well**
- Short, focused methods
- Extract private methods for complex logic

**Example:**

```typescript
// ❌ BAD: Violates SRP - method does too many things
class ReportGenerator {
  generateReport(): string {
    // 1. Fetch data from database
    // 2. Transform data
    // 3. Format as HTML
    // 4. Save to file
    // 5. Send email notification
  }
}

// ✅ GOOD: Follows SRP - each method has single responsibility
class ReportGenerator {
  private fetchData(): ReportData {
    return this.repository.fetchReportData();
  }

  private transformData(data: ReportData): TransformedData {
    return this.dataTransformer.transform(data);
  }

  private formatReport(data: TransformedData): string {
    return this.reportFormatter.formatAsHtml(data);
  }

  private saveReport(report: string): void {
    this.fileStorage.save('report.html', report);
  }

  private notifyUser(): void {
    this.emailService.sendNotification('Report generated');
  }

  generateReport(): string {
    const data = this.fetchData();
    const transformed = this.transformData(data);
    const report = this.formatReport(transformed);
    this.saveReport(report);
    this.notifyUser();
    return report;
  }
}
```

### SRP Benefits

- **Improved Maintainability**: Changes are localized
- **Better Testability**: Each responsibility can be tested in isolation
- **Reduced Coupling**: Classes are less dependent on each other
- **Enhanced Readability**: Clear, focused purpose

---

## Open/Closed Principle (OCP)

### Software entities should be open for extension but closed for modification

**Key Concepts**:

- **Open for Extension**: New behavior can be added
- **Closed for Modification**: Existing code doesn't need to change
- **Achieved through abstraction**: Use interfaces and inheritance

**Example:**

```typescript
// ❌ BAD: Modifying existing code to add new functionality
class PaymentProcessor {
  processPayment(amount: number, type: 'credit' | 'debit'): void {
    if (type === 'credit') {
      // Credit card processing
    } else if (type === 'debit') {
      // Debit card processing
    }
  }
}

// To add PayPal, you must modify processPayment

// ✅ GOOD: Extension through abstraction
interface PaymentMethod {
  process(amount: number): void;
}

class CreditCardPayment implements PaymentMethod {
  process(amount: number): void {
    // Credit card processing
  }
}

class DebitCardPayment implements PaymentMethod {
  process(amount: number): void {
    // Debit card processing
  }
}

class PayPalPayment implements PaymentMethod {
  process(amount: number): void {
    // PayPal processing
  }
}

class PaymentProcessor {
  constructor(private paymentMethods: PaymentMethod[]) {}

  processPayment(paymentMethod: PaymentMethod, amount: number): void {
    paymentMethod.process(amount);
  }
}

// To add new payment method, just implement PaymentMethod interface
```

### OCP Benefits

- **Flexibility**: Easy to add new features
- **Stability**: Existing code doesn't change
- **Maintainability**: Reduced risk of breaking existing functionality
- **Testability**: Each extension can be tested independently

---

## Liskov Substitution Principle (LSP)

### Objects of a superclass should be replaceable with objects of its subclasses

**Key Concepts**:

- **Subtype must be substitutable for its supertype**
- **Behavioral consistency**: Subclasses must honor the contract
- **No unexpected behavior**: Subtypes shouldn't break expectations

**Example:**

```typescript
// ❌ BAD: Violates LSP - Square breaks Rectangle contract
class Rectangle {
  constructor(protected width: number, protected height: number) {}

  setWidth(width: number): void {
    this.width = width;
  }

  setHeight(height: number): void {
    this.height = height;
  }

  area(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  constructor(size: number) {
    super(size, size);
  }

  setWidth(width: number): void {
    // This breaks LSP - setting width also changes height
    super.setWidth(width);
    super.setHeight(width);
  }

  setHeight(height: number): void {
    // This breaks LSP - setting height also changes width
    super.setWidth(height);
    super.setHeight(height);
  }
}

function printArea(rectangle: Rectangle): void {
  rectangle.setWidth(5);
  rectangle.setHeight(10);
  console.log(rectangle.area()); // Expects 50
}

const square = new Square(5);
printArea(square); // Returns 100, not 50! LSP violation.

// ✅ GOOD: Separate hierarchies
interface Shape {
  area(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}

  area(): number {
    return this.width * this.height;
  }
}

class Square implements Shape {
  constructor(private size: number) {}

  area(): number {
    return this.size * this.size;
  }
}

function printArea(shape: Shape): void {
  console.log(shape.area()); // Works correctly for all shapes
}
```

### LSP Guidelines

- **Preconditions cannot be strengthened**: Subclasses shouldn't require more
- **Postconditions cannot be weakened**: Subclasses shouldn't promise less
- **Invariant conditions must be maintained**: Subclasses must preserve all guarantees
- **History constraint**: Subclasses shouldn't introduce new modifying operations

---

## Interface Segregation Principle (ISP)

### Clients should not be forced to depend on interfaces they don't use

**Key Concepts**:

- **Small, focused interfaces**
- **One method per interface where appropriate**
- **Clients depend only on methods they use**

**Example:**

```typescript
// ❌ BAD: Violates ISP - clients forced to implement methods they don't need
interface UserOperations {
  createUser(user: User): void;
  deleteUser(id: string): void;
  updateUser(user: User): void;
  findById(id: string): User | null;
  findByEmail(email: string): User | null;
  findAll(): User[];
}

class UserReader implements UserOperations {
  // Reader shouldn't need to implement create/update/delete
  createUser(user: User): void {
    throw new Error('Not implemented');
  }
  deleteUser(id: string): void {
    throw new Error('Not implemented');
  }
  updateUser(user: User): void {
    throw new Error('Not implemented');
  }
  // ...
}

// ✅ GOOD: Follows ISP - small, focused interfaces
interface IUserReader {
  findById(id: string): User | null;
  findByEmail(email: string): User | null;
  findAll(): User[];
}

interface IUserWriter {
  create(user: User): void;
  update(user: User): void;
  delete(id: string): void;
}

class UserReader implements IUserReader {
  // Only implements methods needed for reading
  findById(id: string): User | null {
    return this.repository.findById(id);
  }
  findByEmail(email: string): User | null {
    return this.repository.findByEmail(email);
  }
  findAll(): User[] {
    return this.repository.findAll();
  }
}
```

### Client-Specific Interfaces

- **Interfaces designed for specific client needs**
- **No "god interfaces" with many methods**

**Example:**

```typescript
// ❌ BAD: One interface for all clients
interface UserRepository {
  findById(id: string): User | null;
  findByEmail(email: string): User | null;
  findByUsername(username: string): User | null;
  findAll(): User[];
  findByRole(role: string): User[];
  findByStatus(status: string): User[];
  create(user: User): void;
  update(user: User): void;
  delete(id: string): void;
  changePassword(id: string, newHash: string): void;
  updateLastLogin(id: string): void;
}

// ✅ GOOD: Client-specific interfaces
interface IUserIdFinder {
  findById(id: string): User | null;
}

interface IUserEmailFinder {
  findByEmail(email: string): User | null;
}

interface IUserRepository extends IUserIdFinder, IUserEmailFinder {
  create(user: User): void;
  update(user: User): void;
  delete(id: string): void;
}

class AuthenticationService {
  constructor(private emailFinder: IUserEmailFinder) {
    // Only depends on email finding, not full repository
  }

  async authenticate(email: string, password: string): Promise<User | null> {
    const user = await this.emailFinder.findByEmail(email);
    if (user && this.passwordVerifier.verify(password, user.passwordHash)) {
      return user;
    }
    return null;
  }
}
```

---

## Dependency Inversion Principle (DIP)

### High-Level Modules Don't Depend on Low-Level Modules

**Key Concepts**:

- **Both depend on abstractions (interfaces)**
- **Never depend on concrete implementations**
- **Abstractions don't depend on details, details depend on abstractions**

**Example:**

```typescript
// ❌ BAD: High-level module depends on low-level module
class UserService {
  // Direct dependency on concrete implementation
  private repository = new SqlUserRepository();

  createUser(email: string, password: string): void {
    this.repository.save(email, password);
  }
}

// ✅ GOOD: High-level module depends on abstraction
interface IUserRepository {
  save(email: string, password: string): void;
}

class UserService {
  constructor(private repository: IUserRepository) {
    // Dependency on interface, not implementation
  }

  createUser(email: string, password: string): void {
    this.repository.save(email, password);
  }
}
```

### Both Depend on Abstractions

- **High-level modules define abstractions**
- **Low-level modules implement abstractions**
- **Abstractions don't depend on details**

**Example:**

```typescript
// Abstraction defined by high-level module
interface IEmailSender {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
}

// Low-level module implements abstraction
class SmtpEmailSender implements IEmailSender {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // SMTP-specific implementation
    await this.smtpClient.send({
      to,
      subject,
      body
    });
  }
}

class SesModuleEmailSender implements IEmailSender {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // AWS SES-specific implementation
    await this.sesClient.sendEmail({
      Destination: { ToAddresses: [to] },
      Message: { Subject: { Data: subject }, Body: { Data: body } }
    });
  }
}

// High-level module uses abstraction
class UserRegistrationService {
  constructor(
    private userRepository: IUserRepository,
    private emailSender: IEmailSender
  ) {
    // Depends on abstractions only
  }

  async registerUser(email: string, password: string): Promise<void> {
    const user = await this.userRepository.create(email, password);
    await this.emailSender.sendEmail(
      email,
      'Welcome!',
      'Thank you for registering.'
    );
  }
}
```

### Abstraction Independence

- **Interfaces don't depend on implementation details**
- **Details change without affecting abstractions**

**Example:**

```typescript
// ✅ GOOD: Abstraction independent of details
interface ICacheProvider {
  get<T>(key: string): T | null;
  set<T>(key: string, value: T, ttl: number): void;
  delete(key: string): void;
}

class InMemoryCacheProvider implements ICacheProvider {
  private cache = new Map<string, any>();

  get<T>(key: string): T | null {
    return this.cache.get(key) || null;
  }

  set<T>(key: string, value: T, ttl: number): void {
    this.cache.set(key, value);
    setTimeout(() => this.cache.delete(key), ttl * 1000);
  }

  delete(key: string): void {
    this.cache.delete(key);
  }
}

class RedisCacheProvider implements ICacheProvider {
  async get<T>(key: string): Promise<T | null> {
    const value = await this.redisClient.get(key);
    return value ? JSON.parse(value) : null;
  }

  async set<T>(key: string, value: T, ttl: number): Promise<void> {
    await this.redisClient.setex(key, ttl, JSON.stringify(value));
  }

  async delete(key: string): Promise<void> {
    await this.redisClient.del(key);
  }
}

// Usage - abstraction independent of implementation
class ProductService {
  constructor(
    private repository: IProductRepository,
    private cache: ICacheProvider
  ) {}

  async getProduct(id: string): Promise<Product | null> {
    const cached = this.cache.get<Product>(`product:${id}`);
    if (cached) return cached;

    const product = await this.repository.findById(id);
    if (product) {
      this.cache.set(`product:${id}`, product, 3600); // 1 hour TTL
    }
    return product;
  }
}
```

---

## Enables Testability and Flexibility

### Testability

- **Dependencies can be mocked easily**
- **Unit tests isolate behavior**
- **No external system dependencies in tests**

**Example:**

```typescript
class UserServiceTest {
  @Test
  async createUser_saves_to_repository(): Promise<void> {
    // Arrange - create mock repository
    const mockRepository = {
      save: jest.fn().mockResolvedValue({ id: '123', email: 'test@example.com' })
    } as jest.Mocked<IUserRepository>;

    const mockEmailSender = {
      sendEmail: jest.fn().mockResolvedValue(undefined)
    } as jest.Mocked<IEmailSender>;

    const service = new UserService(mockRepository, mockEmailSender);

    // Act
    await service.registerUser('test@example.com', 'password123');

    // Assert - verify dependencies were called correctly
    expect(mockRepository.save).toHaveBeenCalledWith(
      'test@example.com',
      expect.any(String) // password hash
    );
    expect(mockEmailSender.sendEmail).toHaveBeenCalledWith(
      'test@example.com',
      'Welcome!',
      expect.any(String)
    );
  }
}
```

### Flexibility

- **Easy to swap implementations**
- **Configuration changes without code changes**
- **Multiple implementations for different environments**

**Example:**

```typescript
// Development environment
const developmentConfig = {
  repository: new InMemoryUserRepository(),
  emailSender: new ConsoleEmailSender() // Log emails to console
};

// Production environment
const productionConfig = {
  repository: new PostgreSqlUserRepository(process.env.DATABASE_URL),
  emailSender: new SesModuleEmailSender(process.env.AWS_REGION)
};

// Same service works with both
const userService = new UserService(
  config.repository,
  config.emailService
);
```

---

## SOLID in Practice

### Example: Shopping Cart

```typescript
// SRP: Each class has single responsibility
class Cart {
  // Manages cart state and business rules
}

class CartValidator {
  // Validates cart operations
}

class CartCalculator {
  // Calculates totals and discounts
}

// ISP: Small, focused interfaces
interface ICartReader {
  getItems(): CartItem[];
  getTotal(): Money;
}

interface ICartWriter {
  addItem(item: CartItem): void;
  removeItem(itemId: string): void;
  clear(): void;
}

interface ICartValidator {
  canAddItem(item: CartItem): boolean;
}

interface ICartCalculator {
  calculateTotal(items: CartItem[]): Money;
  calculateDiscount(items: CartItem[], coupon: Coupon): Money;
}

// DIP: All depend on abstractions
class CartService {
  constructor(
    private reader: ICartReader,
    private writer: ICartWriter,
    private validator: ICartValidator,
    private calculator: ICartCalculator
  ) {}

  async addItem(item: CartItem): Promise<AddItemResult> {
    if (!this.validator.canAddItem(item)) {
      return AddItemResult.failure('Item cannot be added');
    }

    this.writer.addItem(item);
    const total = this.reader.getTotal();

    return AddItemResult.success(total);
  }
}
```

---

## SOLID Principles Summary

| Principle | Description | Key Benefit |
|-----------|-------------|-------------|
| **SRP** | Single Responsibility | Maintainability |
| **OCP** | Open/Closed | Extensibility |
| **LSP** | Liskov Substitution | Reliability |
| **ISP** | Interface Segregation | Flexibility |
| **DIP** | Dependency Inversion | Testability |

### When to Apply

- **New Code**: Apply SOLID from the start
- **Refactoring**: Gradually introduce SOLID principles
- **Growing Codebases**: When complexity increases
- **Team Collaboration**: When multiple developers work together

### Common Signs of Violation

- **God Classes**: Classes that do too much
- **Rigidity**: Changes require widespread modifications
- **Fragility**: Small changes break unrelated functionality
- **Immobility**: Code can't be reused
- **Dependency Cycles**: Circular dependencies between modules

---

## References

- [Clean Architecture](03-architecture/clean-architecture.md) - Architecture patterns
- [Design Patterns](../06-design-patterns/gof-catalog.md) - GoF patterns catalog
- [Coding Conventions](coding-conventions.md) - Project coding standards
- [AI Code Review](../07-review-checklists/ai-code-review.md) - Code review checklists
