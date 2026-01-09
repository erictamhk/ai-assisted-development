---
name: refactorer
description: Improve code quality while preserving functionality through systematic refactoring with tests, SOLID principles, and design patterns
license: MIT
compatibility: claude-code, opencode, agents.md
metadata:
  audience: developers
  workflow: code-refactoring, technical-debt-reduction
  version: "1.0"
---

# Refactorer Skill

**Purpose:** Improve code quality while preserving functionality through systematic refactoring with tests, SOLID principles, and design patterns

**What This Skill Delivers:** Refactored code with improved maintainability, reduced complexity, and preserved behavior verified by tests

**When to Use:** When improving existing code without changing functionality, applying design patterns, or reducing technical debt

---

## Three Agent Laws

### Law 1: CLARIFY
> Never assume intent. Always ask for clarification when uncertain.

- If refactoring scope is unclear → Ask
- If behavior preservation is uncertain → Ask
- If priority of improvements is unknown → Ask

### Law 2: FOLLOW PROCESS
> Always use the defined workflow for your role.

- Cover with tests before any refactoring
- Make small changes one at a time
- Run tests after each change
- Never skip validation steps

### Law 3: PROTECT QUALITY
> Never skip rules, checks, or quality gates.

- Never refactor without test coverage
- Never change behavior (tests must pass)
- Never skip SOLID principles
- Never skip design pattern application

### Law 4: ENFORCE DISCIPLINE
> Hardened discipline prevents rule violations.

**Pre-Action Checkpoint (HARD STOP):**
```
□ Does this require approval? (Refactoring scope, behavior change = YES)
□ If YES → Did I cover with tests?
    - YES → Proceed
    - NO → Write tests first
```

**Self-Audit Trail (Before Any Refactoring):**
```
"[REFACTOR] - Tests exist? [Y/N] - Tests pass? [Y/N] - Scope clear? [Y/N]"
```

---

## What I Do

1. **Code Quality Improvement** - Reduce complexity, improve readability, apply clean code principles
2. **SOLID Refactoring** - Apply single responsibility, open-closed, Liskov, interface segregation, dependency inversion
3. **Design Pattern Application** - Replace conditional logic with appropriate GoF patterns
4. **Extract Method/Class** - Break down large functions and god classes into smaller, focused units
5. **Rename Refactoring** - Improve naming for clarity and self-documentation
6. **Safe Refactoring with Tests** - Ensure tests pass before and after every change
7. **Technical Debt Reduction** - Identify and address code smells systematically
8. **Code Smell Elimination** - Apply appropriate refactoring techniques to known patterns

---

## When to Use Me

**Use when:**
- Improving code maintainability without changing behavior
- Reducing technical debt in existing codebase
- Applying design patterns to replace conditional logic
- Breaking down large functions/classes
- Improving naming and code clarity
- Refactoring toward SOLID principles

**Don't use when:**
- Changing functionality (use CODER instead)
- Adding new features (use CODER instead)
- No test coverage exists (write tests first)
- Only code review needed (use REVIEWER instead)

---

## Refactoring Process

### 1. Cover with Tests (First!)

**Before any refactoring:**

```typescript
// Ensure tests exist and pass
describe('Before Refactoring', () => {
  it('should preserve existing behavior', () => {
    const result = subject.operate(input);
    expect(result).toEqual(expectedOutput);
  });

  it('should handle edge cases', () => {
    // Test all edge cases
  });
});

// Run tests - all must pass
$ npm test  // All tests pass ✓
```

### 2. Small Steps (One Change at a Time)

**Refactoring mantra:**
```
One change → Run tests → Commit → Repeat
```

**Small step examples:**
- Extract one method (not all methods)
- Rename one variable (not all variables)
- Apply one pattern (not all patterns)

### 3. Preserve Behavior

**Golden rule:**
```
Tests must pass BEFORE and AFTER refactoring
```

**If tests fail:**
- Revert the change
- Fix the refactoring approach
- Never change tests to make refactoring "work"

### 4. Commit Often

**Commit after each small step:**
```
refactor: Extract validateEmail() method
refactor: Rename 'data' to 'userInput'
refactor: Apply Strategy pattern for payment processing
```

---

## Code Smells and Refactoring Techniques

### Smell 1: Long Method

**Symptoms:**
- Function exceeds 20 lines
- Hard to understand logic
- Multiple responsibilities

**Refactoring Technique: Extract Method**

**Before:**
```typescript
class OrderProcessor {
  processOrder(order: Order): void {
    // 50 lines of code...
    const subtotal = order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    const tax = subtotal * 0.08;
    const shipping = this.calculateShipping(order);
    const total = subtotal + tax + shipping;
    // 40 more lines...
  }
}
```

**After:**
```typescript
class OrderProcessor {
  processOrder(order: Order): void {
    const subtotal = this.calculateSubtotal(order);
    const tax = this.calculateTax(subtotal);
    const shipping = this.calculateShipping(order);
    const total = this.calculateTotal(subtotal, tax, shipping);
    this.applyDiscountIfApplicable(order, total);
    this.updateInventory(order);
    this.sendConfirmation(order);
  }

  private calculateSubtotal(order: Order): number {
    return order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  private calculateTax(subtotal: number): number {
    return subtotal * 0.08;
  }

  private calculateTotal(subtotal: number, tax: number, shipping: number): number {
    return subtotal + tax + shipping;
  }

  private applyDiscountIfApplicable(order: Order, total: number): void {
    // Discount logic
  }

  private updateInventory(order: Order): void {
    // Inventory update logic
  }

  private sendConfirmation(order: Order): void {
    // Confirmation logic
  }
}
```

### Smell 2: Large Class (God Class)

**Symptoms:**
- Class has many responsibilities
- More than 200-300 lines
- Many instance variables

**Refactoring Technique: Extract Class**

**Before:**
```typescript
class User {
  constructor(
    public id: string,
    public email: string,
    public passwordHash: string,
    public name: string,
    public address: string,
    public phone: string,
    public preferences: UserPreferences,
    public statistics: UserStatistics
  ) {}

  // User authentication methods
  authenticate(password: string): boolean { /* ... */ }
  changePassword(oldPassword: string, newPassword: string): void { /* ... */ }

  // User profile methods
  updateProfile(profile: ProfileData): void { /* ... */ }
  updateAddress(address: Address): void { /* ... */ }
  updatePhone(phone: string): void { /* ... */ }

  // User preferences methods
  getPreference(key: string): unknown { /* ... */ }
  setPreference(key: string, value: unknown): void { /* ... */ }

  // User statistics methods
  getLoginCount(): number { /* ... */ }
  getLastLogin(): Date { /* ... */ }
  incrementLoginCount(): void { /* ... */ }
}
```

**After:**
```typescript
// Value objects
class UserId {
  constructor(public readonly value: string) {}
}

class UserEmail {
  private readonly regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  constructor(public readonly value: string) {
    if (!this.regex.test(value)) {
      throw new InvalidEmailError(value);
    }
  }
}

class UserPassword {
  constructor(public readonly hash: string) {}
}

// Core entity
class User {
  constructor(
    public readonly id: UserId,
    public readonly email: UserEmail,
    private readonly password: UserPassword,
    public readonly profile: UserProfile,
    public readonly preferences: UserPreferences,
    private readonly statistics: UserStatistics
  ) {}

  authenticate(password: string): boolean {
    return this.password.verify(password);
  }

  updateProfile(data: ProfileUpdateData): User {
    return new User(
      this.id,
      this.email,
      this.password,
      this.profile.update(data),
      this.preferences,
      this.statistics
    );
  }
}

// Extracted classes
class UserPreferences {
  private readonly preferences = new Map<string, unknown>();

  get(key: string): unknown {
    return this.preferences.get(key);
  }

  set(key: string, value: unknown): void {
    this.preferences.set(key, value);
  }
}

class UserStatistics {
  private loginCount = 0;
  private lastLogin: Date | null = null;

  recordLogin(): void {
    this.loginCount++;
    this.lastLogin = new Date();
  }

  getLoginCount(): number {
    return this.loginCount;
  }

  getLastLogin(): Date | null {
    return this.lastLogin;
  }
}
```

### Smell 3: Feature Envy

**Symptoms:**
- Method uses more data from other class than its own
- Data coupling across class boundaries

**Refactoring Technique: Move Method**

**Before:**
```typescript
class User {
  constructor(public name: string, public department: string) {}
}

class Department {
  constructor(public name: string, public budget: number) {}
}

class PayrollCalculator {
  calculatePayroll(users: User[]): number {
    let total = 0;
    for (const user of users) {
      // Feature envy: Using Department data from User
      if (user.department === 'Engineering') {
        total += 100000;
      } else if (user.department === 'Sales') {
        total += 80000;
      } else {
        total += 60000;
      }
    }
    return total;
  }
}
```

**After:**
```typescript
class Department {
  private readonly salaryByDepartment = new Map<string, number>([
    ['Engineering', 100000],
    ['Sales', 80000],
    ['Default', 60000],
  ]);

  getSalaryForDepartment(): number {
    return this.salaryByDepartment.get(this.name) ?? this.salaryByDepartment.get('Default')!;
  }
}

class PayrollCalculator {
  calculatePayroll(users: User[]): number {
    return users.reduce((total, user) => {
      return total + user.department.getSalaryForDepartment();
    }, 0);
  }
}
```

### Smell 4: Data Clumps

**Symptoms:**
- Same group of variables appear together in many places
- Passed as parameters between methods

**Refactoring Technique: Extract Class**

**Before:**
```typescript
class Rectangle {
  constructor(
    public x1: number,
    public y1: number,
    public x2: number,
    public y2: number
  ) {}

  getArea(): number {
    return Math.abs((this.x2 - this.x1) * (this.y2 - this.y1));
  }

  getWidth(): number {
    return Math.abs(this.x2 - this.x1);
  }

  getHeight(): number {
    return Math.abs(this.y2 - this.y1);
  }

  containsPoint(x: number, y: number): boolean {
    return x >= this.x1 && x <= this.x2 && y >= this.y1 && y <= this.y2;
  }
}
```

**After:**
```typescript
class Point {
  constructor(public readonly x: number, public readonly y: number) {}
}

class Rectangle {
  constructor(
    public readonly topLeft: Point,
    public readonly bottomRight: Point
  ) {}

  getArea(): number {
    return Math.abs(
      (this.bottomRight.x - this.topLeft.x) *
      (this.bottomRight.y - this.topLeft.y)
    );
  }

  getWidth(): number {
    return Math.abs(this.bottomRight.x - this.topLeft.x);
  }

  getHeight(): number {
    return Math.abs(this.bottomRight.y - this.topLeft.y);
  }

  containsPoint(point: Point): boolean {
    return (
      point.x >= this.topLeft.x &&
      point.x <= this.bottomRight.x &&
      point.y >= this.topLeft.y &&
      point.y <= this.bottomRight.y
    );
  }
}
```

### Smell 5: Switch Statement / Polymorphism

**Symptoms:**
- Large switch or if-else chains
- Adding cases requires modifying existing code
- Violates Open/Closed Principle

**Refactoring Technique: Replace Conditional with Polymorphism**

**Before:**
```typescript
class PaymentProcessor {
  processPayment(payment: Payment): PaymentResult {
    switch (payment.type) {
      case 'credit_card':
        // Credit card processing logic (50 lines)
        return { status: 'success', transactionId: 'xxx' };
      case 'paypal':
        // PayPal processing logic (40 lines)
        return { status: 'success', transactionId: 'yyy' };
      case 'bank_transfer':
        // Bank transfer logic (45 lines)
        return { status: 'success', transactionId: 'zzz' };
      default:
        throw new UnknownPaymentTypeError(payment.type);
    }
  }
}
```

**After:**
```typescript
// Strategy pattern implementation
interface PaymentStrategy {
  process(payment: Payment): PaymentResult;
}

class CreditCardPayment implements PaymentStrategy {
  process(payment: Payment): PaymentResult {
    // Credit card specific logic
    return { status: 'success', transactionId: this.generateTransactionId() };
  }

  private generateTransactionId(): string {
    return `CC-${Date.now()}`;
  }
}

class PayPalPayment implements PaymentStrategy {
  process(payment: Payment): PaymentResult {
    // PayPal specific logic
    return { status: 'success', transactionId: `PP-${Date.now()}` };
  }
}

class BankTransferPayment implements PaymentStrategy {
  process(payment: Payment): PaymentResult {
    // Bank transfer specific logic
    return { status: 'success', transactionId: `BT-${Date.now()}` };
  }
}

// Factory
class PaymentProcessor {
  private readonly strategies = new Map<string, PaymentStrategy>([
    ['credit_card', new CreditCardPayment()],
    ['paypal', new PayPalPayment()],
    ['bank_transfer', new BankTransferPayment()],
  ]);

  processPayment(payment: Payment): PaymentResult {
    const strategy = this.strategies.get(payment.type);
    if (!strategy) {
      throw new UnknownPaymentTypeError(payment.type);
    }
    return strategy.process(payment);
  }

  registerStrategy(type: string, strategy: PaymentStrategy): void {
    this.strategies.set(type, strategy);
  }
}
```

### Smell 6: Speculative Generality

**Symptoms:**
- Code written for "future" features that don't exist
- Unused abstraction layers
- Over-engineered solutions

**Refactoring Technique: Collapse Hierarchy / Inline Class**

**Before:**
```typescript
// Over-engineered abstraction
interface IRepository<T, TId> {
  findById(id: TId): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: TId): Promise<void>;
}

abstract class AbstractRepository<T, TId> implements IRepository<T, TId> {
  protected readonly collection: Collection<T>;

  async findById(id: TId): Promise<T | null> {
    return this.collection.findOne({ id });
  }

  async findAll(): Promise<T[]> {
    return this.collection.find().toArray();
  }

  abstract save(entity: T): Promise<T>;
  abstract delete(id: TId): Promise<void>;
}

class UserRepository extends AbstractRepository<User, string> {
  protected readonly collection = getCollection<User>('users');

  async save(entity: User): Promise<User> {
    await this.collection.insertOne(entity);
    return entity;
  }

  async delete(id: string): Promise<void> {
    await this.collection.deleteOne({ id });
  }
}
```

**After:**
```typescript
// Simple, concrete implementation
class UserRepository {
  private readonly collection = getCollection<User>('users');

  async findById(id: string): Promise<User | null> {
    return this.collection.findOne({ id });
  }

  async findAll(): Promise<User[]> {
    return this.collection.find().toArray();
  }

  async save(user: User): Promise<User> {
    await this.collection.insertOne(user);
    return user;
  }

  async delete(id: string): Promise<void> {
    await this.collection.deleteOne({ id });
  }
}
```

---

## SOLID Refactoring

### S - Single Responsibility Principle

**Before (Multiple Responsibilities):**
```typescript
class UserService {
  authenticate(email: string, password: string): User { /* ... */ }
  sendEmail(user: User, message: string): void { /* ... */ }
  generateReport(users: User[]): Report { /* ... */ }
  backupDatabase(): void { /* ... */ }
}
```

**After (Single Responsibility):**
```typescript
class AuthenticationService {
  constructor(private readonly userRepository: IUserRepository) {}
  authenticate(email: string, password: string): User { /* ... */ }
}

class EmailService {
  sendEmail(user: User, message: string): void { /* ... */ }
}

class UserReportService {
  constructor(private readonly userRepository: IUserRepository) {}
  generateReport(users: User[]): Report { /* ... */ }
}

class DatabaseService {
  backupDatabase(): void { /* ... */ }
}
```

### O - Open/Closed Principle

**Before (Modify for Extension):**
```typescript
class DiscountCalculator {
  calculateDiscount(user: User, order: Order): number {
    if (user.isPremium) {
      return 0.15;
    }
    if (user.isEmployee) {
      return 0.20;
    }
    if (order.total > 1000) {
      return 0.10;
    }
    return 0;
  }
}
```

**After (Extend without Modification):**
```typescript
interface DiscountStrategy {
  calculateDiscount(user: User, order: Order): number;
}

class PremiumDiscount implements DiscountStrategy {
  calculateDiscount(user: User, order: Order): number {
    return user.isPremium ? 0.15 : 0;
  }
}

class EmployeeDiscount implements DiscountStrategy {
  calculateDiscount(user: User, order: Order): number {
    return user.isEmployee ? 0.20 : 0;
  }
}

class HighValueDiscount implements DiscountStrategy {
  calculateDiscount(user: User, order: Order): number {
    return order.total > 1000 ? 0.10 : 0;
  }
}

class DiscountCalculator {
  private readonly strategies: DiscountStrategy[] = [
    new PremiumDiscount(),
    new EmployeeDiscount(),
    new HighValueDiscount(),
  ];

  calculateDiscount(user: User, order: Order): number {
    return this.strategies.reduce(
      (total, strategy) => total + strategy.calculateDiscount(user, order),
      0
    );
  }

  addStrategy(strategy: DiscountStrategy): void {
    this.strategies.push(strategy);
  }
}
```

### L - Liskov Substitution Principle

**Before (Violates LSP):**
```typescript
class Rectangle {
  constructor(public width: number, public height: number) {}
  setWidth(width: number): void { this.width = width; }
  setHeight(height: number): void { this.height = height; }
}

class Square extends Rectangle {
  constructor(size: number) {
    super(size, size);
  }

  setWidth(width: number): void {
    this.width = width;
    this.height = width; // Violates Liskov
  }

  setHeight(height: number): void {
    this.width = height; // Violates Liskov
    this.height = height;
  }
}
```

**After (Follows LSP):**
```typescript
interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(
    public readonly width: number,
    public readonly height: number
  ) {}

  getArea(): number {
    return this.width * this.height;
  }
}

class Square implements Shape {
  constructor(public readonly size: number) {}

  getArea(): number {
    return this.size * this.size;
  }
}
```

### I - Interface Segregation Principle

**Before (Fat Interface):**
```typescript
interface Machine {
  print(document: Document): void;
  staple(document: Document): void;
  scan(document: Document): void;
  fax(document: Document): void;
}

class AllInOnePrinter implements Machine {
  print(document: Document): void { /* ... */ }
  staple(document: Document): void { /* ... */ }
  scan(document: Document): void { /* ... */ }
  fax(document: Document): void { /* ... */ }
}

class SimplePrinter implements Machine {
  print(document: Document): void { /* ... */ }
  // Must implement unused methods (throw UnsupportedError)
  staple(document: Document): void { throw new Error('Not supported'); }
  scan(document: Document): void { throw new Error('Not supported'); }
  fax(document: Document): void { throw new Error('Not supported'); }
}
```

**After (Segregated Interfaces):**
```typescript
interface Printer {
  print(document: Document): void;
}

interface Stapler {
  staple(document: Document): void;
}

interface Scanner {
  scan(document: Document): void;
}

interface Fax {
  fax(document: Document): void;
}

class SimplePrinter implements Printer {
  print(document: Document): void { /* ... */ }
}

class AllInOnePrinter implements Printer, Stapler, Scanner, Fax {
  print(document: Document): void { /* ... */ }
  staple(document: Document): void { /* ... */ }
  scan(document: Document): void { /* ... */ }
  fax(document: Document): void { /* ... */ }
}
```

### D - Dependency Inversion Principle

**Before (Depends on Concrete):**
```typescript
class MySQLUserRepository {
  findById(id: string): User | null { /* MySQL specific */ }
  save(user: User): void { /* MySQL specific */ }
}

class UserService {
  private readonly repository = new MySQLUserRepository(); // Direct dependency

  getUser(id: string): User | null {
    return this.repository.findById(id);
  }
}
```

**After (Depends on Abstractions):**
```typescript
interface IUserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

class MySQLUserRepository implements IUserRepository {
  async findById(id: string): Promise<User | null> {
    // MySQL implementation
  }

  async save(user: User): Promise<void> {
    // MySQL implementation
  }
}

class UserService {
  constructor(private readonly repository: IUserRepository) {} // Injection

  async getUser(id: string): Promise<User | null> {
    return this.repository.findById(id);
  }
}

// Usage with dependency injection
const repository = new MySQLUserRepository();
const service = new UserService(repository);
```

---

## Safe Refactoring Checklist

### Before Refactoring
- [ ] Test coverage exists (> 80%)
- [ ] All tests pass
- [ ] Refactoring scope defined
- [ ] Backup/version control ready

### During Refactoring
- [ ] One change at a time
- [ ] Tests pass after each change
- [ ] Commit after each successful change
- [ ] No behavior change (tests verify)

### After Refactoring
- [ ] All tests still pass
- [ ] Code coverage maintained or improved
- [ ] No new code smells introduced
- [ ] Documentation updated

---

## Self-Quality Gate (Before Output)

**Before delivering refactored code, verify:**

```
[ ] All tests pass (before and after)
[ ] Behavior preserved (tests verify)
[ ] No new code smells introduced
[ ] SOLID principles applied
[ ] Design patterns used appropriately
[ ] Naming improved for clarity
[ ] Code complexity reduced
[ ] No unused code introduced
[ ] Documentation updated
```

**If any check fails → Fix before delivery**

---

## Knowledge Access

**Always access dev-knowledge/ before acting:**

1. Classify refactoring type → Identify code smell
2. grep category → Find refactoring patterns
3. read patterns → Understand techniques
4. apply → Implement refactoring

**Required Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 04-coding-style | `04-coding-style/clean-code.md` | Required |
| 04-coding-style | `04-coding-style/solid-principles.md` | Required |
| 03-architecture | `03-architecture/refactoring-journey.md` | Required |
| 06-design-patterns | `06-design-patterns/gof-catalog.md` | Required |
| 06-design-patterns | `06-design-patterns/pattern-language.md` | Required |
| 06-design-patterns | `06-design-patterns/clean-architecture-patterns.md` | Required |
| 11-error-handling | `11-error-handling/result-pattern.md` | Required |
| 11-error-handling | `11-error-handling/design-by-contract.md` | Required |
| 07-review-checklists | `07-review-checklists/code-review.md` | Required |

**Recommended Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 05-testing | `05-testing/tdd-workflow.md` | Recommended |

---

## Output Format

```markdown
# Refactoring Report: [File/Module Name]

## Before Refactoring

### Code Smells Identified
| Smell | Location | Severity |
|-------|----------|----------|
| Long Method | `ClassName:Line` | HIGH/MEDIUM/LOW |
| Large Class | `ClassName` | HIGH/MEDIUM/LOW |

### Complexity Metrics
| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Lines of Code | XXX | XXX | -XX% |
| Cyclomatic Complexity | XX | XX | -XX% |
| Number of Methods | XX | XX | -XX% |

## Refactoring Applied

### 1. [First Refactoring]
**Smell:** [Smell Name]
**Technique:** [Technique Name]

**Before:**
```typescript
// Code before refactoring
```

**After:**
```typescript
// Code after refactoring
```

### 2. [Second Refactoring]
...

## Verification

### Tests
- [x] All existing tests pass
- [x] No behavior change
- [x] Coverage maintained

### Quality
- [x] SOLID principles applied
- [x] Design patterns used
- [x] Naming improved
- [x] Complexity reduced

## Summary
- **Files Changed:** [N]
- **Lines Removed:** [N]
- **New Code Smells:** [N]
```

---

## Constraints

1. **Never refactor without test coverage** - Write tests first
2. **Never change behavior** - Tests must pass before and after
3. **Never skip small steps** - One change at a time
4. **Never skip commit** - Commit after each small change
5. **Never skip SOLID** - Apply SOLID principles
6. **Never skip design patterns** - Use appropriate patterns
7. **Never skip naming improvement** - Improve clarity
8. **Never skip self-quality gate** - Verify before output
9. **Never skip REVIEWER** - Always route through REVIEWER
10. **Never skip documentation** - Document changes

---

## Anti-Patterns (Never Do These)

| Anti-Pattern | Instead |
|--------------|---------|
| Refactor without tests | Write tests first |
| Change behavior | Preserve behavior |
| Big refactor | Small steps, commit often |
| Skip commit | Commit after each change |
| Skip SOLID | Apply SOLID principles |
| Skip patterns | Use design patterns |
| Poor naming | Improve naming for clarity |
| Skip review | Always route through REVIEWER |
| Skip verification | Complete quality gate |
| Introduce new smells | Maintain code quality |

---

## Related Skills

- **CODER** - Implements new features, uses refactoring techniques
- **REVIEWER** - Validates refactored code quality
- **ARCHITECT** - Designs architecture for refactoring
- **PLANNER** - Breaks down refactoring into tasks

**Refactoring Flow:**
```
REFACTORER (Refactor with Tests) → REVIEWER (Validate) → Human Approval
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-09 | Initial REFACTORER skill with code smells and SOLID refactoring |

---

**Skill Version:** 1.0
**Compatibility:** claude-code, opencode, agents.md
