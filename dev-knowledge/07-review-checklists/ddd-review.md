# DDD Review Checklist

This checklist provides a comprehensive guide for reviewing Domain-Driven Design implementation, ensuring proper application of tactical DDD patterns and strategic design principles.

---

## 1. Ubiquitous Language

### 1.1 Language Consistency

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Domain terms used consistently in code | ☐ | | |
| No technical jargon in domain layer | ☐ | | |
| Business terminology matches domain expert vocabulary | ☐ | | |
| Domain events named in past tense | ☐ | | |
| No translation between bounded contexts | ☐ | | |

**Examples:**
```typescript
// ✅ GOOD - Ubiquitous language
class OrderSubmissionService {
  async submitOrder(order: Order): Promise<SubmitOrderResult> {
    if (!order.canBeSubmitted()) {
      throw new OrderCannotBeSubmittedError(order.status);
    }
    order.submit();
    // ...
  }
}

// ❌ BAD - Technical jargon
class OrderProcessor {
  async processOrder(orderObj: any) {  // "orderObj" is technical
    if (orderObj.status !== 'PENDING') {
      throw new Error('Invalid state');  // Technical error
    }
    // ...
  }
}
```

### 1.2 Naming in Domain Layer

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Aggregate names are nouns | ☐ | | |
| Entity names are meaningful domain terms | ☐ | | |
| Value object names are descriptive | ☐ | | |
| Domain service names are verbs | ☐ | | |
| Event names are in past tense | ☐ | | |

---

## 2. Entities

### 2.1 Identity

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Entity has unique identity (ID) | ☐ | | |
| Identity is immutable | ☐ | | |
| Equality based on identity, not attributes | ☐ | | |
| Proper implementation of equals/hashCode | ☐ | | |
| ID type is a value object or primitive | ☐ | | |

```typescript
// ✅ GOOD - Proper entity identity
class Order implements Entity<OrderId> {
  private readonly id: OrderId;  // Immutable identity
  private status: OrderStatus;
  
  constructor(id: OrderId, items: OrderItem[]) {
    this.id = id;
    this.status = OrderStatus.DRAFT;
    // ...
  }

  get id(): OrderId { return this.id; }

  equals(other: Order): boolean {
    return other instanceof Order && this.id.equals(other.id);
  }
}
```

### 2.2 Entity Behavior

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Entities contain business logic | ☐ | | |
| No anemic entities (data only) | ☐ | | |
| State transitions are encapsulated | ☐ | | |
| Invariants enforced within entity | ☐ | | |
| Business rules in entity methods | ☐ | | |

---

## 3. Value Objects

### 3.1 Immutability

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Value objects are immutable | ☐ | | |
| No setters on value objects | ☐ | | |
| All properties are readonly | ☐ | | |
| Operations return new instances | ☐ | | |

```typescript
// ✅ GOOD - Immutable value object
class Money implements ValueObject {
  private readonly amount: number;
  private readonly currency: Currency;

  constructor(amount: number, currency: Currency) {
    if (amount < 0) throw new MoneyCannotBeNegativeError();
    this.amount = Math.round(amount * 100) / 100;
    this.currency = currency;
  }

  add(other: Money): Money {
    return new Money(this.amount + other.amount, this.currency);
  }

  equals(other: ValueObject): boolean {
    return other instanceof Money && 
           this.amount === other.amount && 
           this.currency.equals(other.currency);
  }
}

// ❌ BAD - Mutable value object
class Money {
  amount: number;  // Mutable!
  currency: Currency;
  
  add(other: Money) {  // Mutates!
    this.amount += other.amount;
  }
}
```

### 3.2 Self-Validation

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Validation in constructor | ☐ | | |
| Throws specific domain errors | ☐ | | |
| Validation logic in value object | ☐ | | |
| No invalid instances can exist | ☐ | | |
| Validation uses Objects.requireNonNull, not Contract | ☐ | | |

### 3.3 Equality

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Equals based on all attributes | ☐ | | |
| No identity in equality check | ☐ | | |
| hashCode consistent with equals | ☐ | | |
| Proper implementation for collections | ☐ | | |

---

## 4. Aggregates

### 4.1 Aggregate Root

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Single aggregate root per aggregate | ☐ | | |
| Root controls access to internals | ☐ | | |
| Root enforces invariants | ☐ | | |
| Root manages internal entities | ☐ | | |
| Root records domain events | ☐ | | |

```typescript
// ✅ GOOD - Proper aggregate root
class Order implements AggregateRoot<OrderId> {
  private readonly id: OrderId;
  private _status: OrderStatus;
  private _lineItems: OrderLineItem[];
  private _domainEvents: DomainEvent[] = [];

  get status(): OrderStatus { return this._status; }
  get lineItems(): ReadonlyArray<OrderLineItem> { 
    return [...this._lineItems];  // Defensive copy
  }

  submit(): void {
    if (this._lineItems.length === 0) {
      throw new OrderMustHaveItemsError();
    }
    this._status = OrderStatus.SUBMITTED;
    this._domainEvents.push(new OrderSubmittedEvent(this.id));
  }
}
```

### 4.2 Aggregate Boundaries

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Only root is accessed from outside | ☐ | | |
| No direct references to internal entities | ☐ | | |
| External references use ID only | ☐ | | |
| Aggregate is small and focused | ☐ | | |
| Transaction boundary aligns with aggregate | ☐ | | |

### 4.3 Invariants

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| All invariants enforced in root | ☐ | | |
| Invariants checked after modifications | ☐ | | |
| Consistent state after each operation | ☐ | | |
| No partially valid states | ☐ | | |
| Business rules in aggregate, not service | ☐ | | |

---

## 5. Domain Services

### 5.1 Service Necessity

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Created only when logic doesn't fit entity/value object | ☐ | | |
| Not used to avoid entity complexity | ☐ | | |
| Service is stateless | ☐ | | |
| Domain language in service name | ☐ | | |

```typescript
// ✅ GOOD - Domain service for cross-aggregate logic
class MoneyTransferService {
  constructor(
    private readonly accountRepository: AccountRepository,
    private readonly eventPublisher: EventPublisher
  ) {}

  async transfer(
    fromId: AccountId, 
    toId: AccountId, 
    amount: Money
  ): Promise<TransferResult> {
    const from = await this.accountRepository.findById(fromId);
    const to = await this.accountRepository.findById(toId);
    
    if (from.balance.lessThan(amount)) {
      throw new InsufficientFundsError(fromId);
    }

    from.withdraw(amount);
    to.deposit(amount);

    await this.accountRepository.save(from);
    await this.accountRepository.save(to);

    this.eventPublisher.publish(
      new MoneyTransferredEvent(fromId, toId, amount)
    );

    return TransferResult.success();
  }
}
```

### 5.2 Service Design

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Stateless (no instance fields) | ☐ | | |
| Uses dependency injection | ☐ | | |
| Operates on domain objects | ☐ | | |
| Named with domain language | ☐ | | |
| Not a wrapper around CRUD | ☐ | | |

---

## 6. Repositories

### 6.1 Repository Interface

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Interface in domain layer | ☐ | | |
| Works with aggregate roots only | ☐ | | |
| Returns domain objects, not DTOs | ☐ | | |
| Method names use domain language | ☐ | | |
| No CRUD methods not in domain | ☐ | | |

```typescript
// ✅ GOOD - Domain repository interface
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  findByCustomerId(customerId: CustomerId): Promise<Order[]>;
  findByStatus(status: OrderStatus): Promise<Order[]>;
  save(order: Order): Promise<void>;
  delete(order: Order): Promise<void>;
}

// ❌ BAD - Non-domain repository interface
interface OrderRepository {
  getOrder(id: string): Order;
  getAllOrders(): Order[];
  createOrder(order: Order): void;
  updateOrder(order: Order): void;
  deleteOrder(id: string): void;
  findByCustomer(customerId: string): Order[];  // Technical naming
}
```

### 6.2 Repository Implementation

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Implementation in infrastructure | ☐ | | |
| No business logic in repository | ☐ | | |
| Maps between domain and persistence | ☐ | | |
| Handles transaction management | ☐ | | |
| Throws domain exceptions | ☐ | | |

### 6.3 Common AI Mistakes

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| No custom `findBy` methods (use framework) | ☐ | | |
| No separate Input class file (use inner class) | ☐ | | |
| No repository implementation in domain | ☐ | | |
| Proper use of framework GenericInMemoryRepository | ☐ | | |

---

## 7. Domain Events

### 7.1 Event Definition

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Named in past tense | ☐ | | |
| Immutable (frozen) | ☐ | | |
| Contains all necessary data | ☐ | | |
| Self-contained (no lazy loading) | ☐ | | |
| Serializable | ☐ | | |

```typescript
// ✅ GOOD - Domain event
class OrderSubmitted implements DomainEvent {
  readonly orderId: OrderId;
  readonly customerId: CustomerId;
  readonly totalAmount: Money;
  readonly submittedAt: DateTime;
  readonly eventId: EventId;

  constructor(orderId: OrderId, customerId: CustomerId, totalAmount: Money) {
    this.orderId = orderId;
    this.customerId = customerId;
    this.totalAmount = totalAmount;
    this.submittedAt = DateTime.now();
    this.eventId = EventId.generate();
  }
}
```

### 7.2 Event Publishing

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Events raised by aggregates | ☐ | | |
| Events published through ports | ☐ | | |
| Uncommitted events tracked | ☐ | | |
| Events published after persistence | ☐ | | |
| Event handlers are idempotent | ☐ | | |

---

## 8. Factories

### 8.1 Factory Usage

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Used for complex creation | ☐ | | |
| Ensures valid object creation | ☐ | | |
| Encapsulates creation logic | ☐ | | |
| Returns fully initialized object | ☐ | | |
| Used when construction has rules | ☐ | | |

```typescript
// ✅ GOOD - Factory for complex creation
class OrderFactory {
  constructor(
    private readonly productRepository: ProductRepository,
    private readonly pricingService: PricingService
  ) {}

  createOrder(customerId: CustomerId, items: OrderItemInput[]): Order {
    const lineItems = items.map(input => {
      const product = this.productRepository.findById(input.productId)
        .orElseThrow(() => new ProductNotFoundError(input.productId));
      const price = this.pricingService.calculatePrice(product, input.quantity);
      return new OrderLineItem(product, input.quantity, price);
    });

    return new Order(OrderId.generate(), customerId, lineItems);
  }
}
```

---

## 9. Bounded Context Integration

### 9.1 Context Map

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Bounded contexts are defined | ☐ | | |
| Relationships documented | ☐ | | |
| Anticorruption layers where needed | ☐ | | |
| Shared kernel identified | ☐ | | |
| Published language used | ☐ | | |

### 9.2 Integration Patterns

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Domain events for loose coupling | ☐ | | |
| No shared databases between contexts | ☐ | | |
| Proper adapter implementation | ☐ | | |
| Data transformation at boundaries | ☐ | | |
| No direct database coupling | ☐ | | |

---

## 10. Scoring Guide

| Category | Weight | Score (1-5) |
|----------|--------|-------------|
| Ubiquitous Language | 10% | |
| Entities | 15% | |
| Value Objects | 15% | |
| Aggregates | 20% | |
| Domain Services | 10% | |
| Repositories | 10% | |
| Domain Events | 10% | |
| Factories | 5% | |
| Bounded Contexts | 5% | |

**Total Score:** _____ / 5

---

## 13. AI-Specific DDD Checks

### 13.1 Common AI Mistakes in DDD Implementation

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| No custom repository methods | ☐ | | |
| No separate Input class file | ☐ | | |
| Value objects use Objects.requireNonNull | ☐ | | |
| No instanceof chains | ☐ | | |
| No primitive obsession | ☐ | | |
| Repository interface in domain, impl in infra | ☐ | | |

### 13.2 Framework Convention Compliance

| Check Item | Result | Location | Notes |
|------------|--------|----------|-------|
| Uses GenericInMemoryRepository | ☐ | | |
| Repository methods match framework conventions | ☐ | | |
| No @Component on services | ☐ | | |
| Services use @Bean registration | ☐ | | |
| Domain events include metadata | ☐ | | |

---

## 14. Common DDD Anti-Patterns

| Anti-Pattern | Severity | Correct Approach |
|--------------|----------|------------------|
| Anemic Domain Model | Critical | Move logic to entities |
| God Aggregate | High | Split into multiple aggregates |
| Repository in Domain | Critical | Interface in domain, impl in infra |
| Mutable Value Object | High | Make immutable |
| Missing Aggregate Invariants | Critical | Enforce in aggregate root |
| Event in Wrong Layer | Medium | Events in domain, publish in app |
| No Identity on Entity | Critical | Add ID field |
| Service with State | Medium | Make stateless |
| Value Object Without Validation | High | Add validation in constructor |
| Technical Naming | Medium | Use domain language |
| Separate Input.java | Critical | Use inner class |
| Custom Repository Methods | High | Use framework methods |

---

## 15. Review Output Format

```markdown
## DDD Review Report

### Aggregate: Order
| Check Item | Result | Location | Issue |
|------------|--------|----------|-------|
| Aggregate Root | ✅ | Order.java | Correct |
| Invariants | ✅ | - | Enforced |
| ID Immutability | ❌ | Line 23 | setId() method found |

### Value Objects
| Check Item | Result | Location | Issue |
|------------|--------|----------|-------|
| Immutability | ✅ | Money.java | Correct |
| Validation | ❌ | Email.java | Uses Contract.requireNotNull |

### Repositories
| Check Item | Result | Location | Issue |
|------------|--------|----------|-------|
| Interface Location | ✅ | - | Domain layer |
| Method Naming | ⚠️ | Line 45 | Uses getByStatus instead of findByStatus |

### Summary
- **Critical Issues**: 1
- **Must Fix Issues**: 2
- **Score**: 3.5/5 ⭐⭐⭐
```

---

## References

1. Domain-Driven Design - Eric Evans
2. Implementing Domain-Driven Design - Vaughn Vernon
3. doc/domain-driven-design-megred.md
4. 02-ddd/tactical-patterns.md
5. ref/ai-coding-exercise-analysis.md
6. ref/ai-coding-exercise/CLAUDE.md - Code Review Process
7. doc/clean-code.md - AI Code Review Checklist
