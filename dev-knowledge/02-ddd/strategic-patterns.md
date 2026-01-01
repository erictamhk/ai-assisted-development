# Strategic Domain-Driven Design

Strategic design addresses the "big picture" of domain architecture, focusing on bounded contexts, ubiquitous language, and context mapping. These patterns help organize complex systems into manageable components with clear boundaries.

## Domain Landscape

```
┌─────────────────────────────────────────────────────────────────┐
│                    DOMAIN LANDSCAPE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐         ┌─────────────┐         ┌──────────┐ │
│   │   CORE      │         │   CORE      │         │ SUPPORTED│ │
│   │   DOMAIN    │◄────────┤   DOMAIN    │────────►│  DOMAIN  │ │
│   │             │  Shared │             │ Conformist│          │ │
│   │  Complex &  │  Kernel │ Complex &  │         │ Simple & │ │
│   │  Valuable   │         │  Valuable  │         │ Commodity│ │
│   └─────────────┘         └─────────────┘         └──────────┘ │
│                                                                 │
│   ┌───────────────────────────────────────────────────────┐   │
│   │                    BOUNDED CONTEXTS                   │   │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │   │
│   │  │OrderCtx │  │InvCtx   │  │PayCtx   │  │UserCtx  │  │   │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │   │
│   └───────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## The Ubiquitous Language

The Ubiquitous Language is the shared vocabulary used by all team members, bridging the gap between domain experts and developers. It must be used consistently in code, documentation, and conversations.

```typescript
// Anti-pattern: Mixing technical and domain language
class OrderService {
  async processOrd() {
    const ord = await this.repo.findById(id);
    if (ord.status !== 'PENDING') throw new Error('Invalid');
  }
}

// DDD Pattern: Ubiquitous Language
class OrderApplicationService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly paymentGateway: PaymentGateway,
    private readonly inventoryService: InventoryService
  ) {}

  async submitOrder(orderId: OrderId): Promise<SubmitOrderResult> {
    const order = await this.orderRepository.findById(orderId);
    
    if (!order.canBeSubmitted()) {
      throw new OrderCannotBeSubmittedError(order.status);
    }
    
    order.submit();
    
    await this.paymentGateway.charge(order.paymentMethod, order.total);
    await this.inventoryService.reserveItems(order.lineItems);
    
    order.confirm();
    await this.orderRepository.save(order);
    
    return SubmitOrderResult.success(order.confirmationNumber);
  }
}
```

**Key Principles:**
- Same terms in code and domain conversations
- No translation between technical and business language
- Evolves as understanding deepens
- Validated through conversation with domain experts

## Bounded Contexts

A Bounded Context is a boundary within which a particular domain model applies. Each bounded context has its own ubiquitous language and explicit boundaries.

```typescript
// Order Bounded Context
namespace OrderContext {
  export interface Order {
    id: OrderId;
    status: OrderStatus;
    lineItems: OrderLineItem[];
    customer: Customer;
    shippingAddress: Address;
    paymentMethod: PaymentMethod;
    total: Money;
    confirmationNumber: ConfirmationNumber;
  }

  export enum OrderStatus {
    DRAFT = 'DRAFT',
    PENDING = 'PENDING',
    SUBMITTED = 'SUBMITTED',
    CONFIRMED = 'CONFIRMED',
    SHIPPED = 'SHIPPED',
    DELIVERED = 'DELIVERED',
    CANCELLED = 'CANCELLED'
  }
}

// Customer Bounded Context - Different model for same concept
namespace CustomerContext {
  export interface CustomerProfile {
    id: CustomerId;
    contactInfo: ContactInformation;
    preferences: CustomerPreferences;
    loyaltyTier: LoyaltyTier;
    accountBalance: Money;
  }
}
```

**Key Principles:**
- Each bounded context has its own ubiquitous language
- Contexts communicate through well-defined interfaces
- No sharing of domain logic between contexts
- Explicit boundaries prevent conceptual confusion

## Context Mapping

Context Map showing relationships between bounded contexts:

```typescript
const contextMap = {
  upstream: {
    name: 'OrderManagement',
    responsibilities: [
      'Order creation',
      'Order lifecycle management',
      'Order history'
    ]
  },
  
  downstream: {
    name: 'Inventory',
    relationship: 'Customer-Supplier',
    protocol: 'Domain Events'
  },
  
  inventoryACL: {
    name: 'InventoryACL',
    role: 'Translate between contexts',
    implementations: [
      'Event consumer from Inventory context',
      'Adapter for inventory data format',
      'Translator for Inventory-specific concepts'
    ]
  },
  
  shared: {
    name: 'CoreTypes',
    sharedBy: ['OrderContext', 'CustomerContext', 'InventoryContext'],
    artifacts: [
      'Money (value object)',
      'Address (value object)',
      'Event types (domain events)'
    ]
  }
};
```

### Context Mapping Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Shared Kernel** | Shared domain code between contexts | Common types, utilities |
| **Customer-Supplier** | Upstream provides, downstream consumes | Service relationships |
| **Conformist** | Downstream adapts to upstream model | Legacy system integration |
| **Anticorruption Layer** | Translation layer between contexts | Legacy migration |
| **Open Host Service** | Published language for integration | API design |
| **Published Language** | Documented exchange format | External integration |

## Subdomains

Identifying and classifying subdomains helps prioritize development effort:

| Subdomain Type | Description | Example |
|----------------|-------------|---------|
| **Core Domain** | Unique business value, competitive advantage | Booking engine |
| **Supporting Domain** | Necessary but not differentiating | Reporting |
| **Generic Domain** | Common solutions, commodity | Authentication |

## Strategic Design Process

1. **Domain Exploration**: Collaborate with domain experts to understand the business
2. **Context Identification**: Identify bounded contexts and their boundaries
3. **Language Definition**: Establish ubiquitous language for each context
4. **Context Mapping**: Define relationships between contexts
5. **Integration Design**: Design interfaces and event flows between contexts

## Tactical Design Patterns

### Entity Pattern

Entities are domain objects that have a distinct identity that persists through time and state changes. Unlike value objects, entities are compared by identity, not by their attributes.

**Key Characteristics:**
- Has a unique identity (ID) that distinguishes it from other entities
- Equality is based on identity, not attributes
- Has a lifecycle (creation, modification, deletion)
- Encapsulates business behavior and rules
- Can change state over time

**Implementation:**

```typescript
interface Entity<TId> {
  getId(): TId;
}

class Order implements Entity<OrderId> {
  private readonly id: OrderId;
  private status: OrderStatus;
  private lineItems: OrderLineItem[];
  private readonly createdAt: DateTime;
  private version: number;

  constructor(id: OrderId, lineItems: OrderLineItem[]) {
    this.id = id;
    this.lineItems = lineItems;
    this.status = OrderStatus.DRAFT;
    this.createdAt = DateTime.now();
    this.version = 1;
  }

  get id(): OrderId { return this.id; }
  get total(): Money { return this.lineItems.reduce((sum, item) => sum.add(item.subtotal), Money.zero()); }
  
  isSameIdentity(other: Order): boolean {
    return this.id.equals(other.id);
  }
}
```

**When to Use Entities:**
- When the object has a lifecycle (can be created, modified, deleted)
- When the object needs to be tracked and referenced
- When equality should be based on identity, not attributes
- When the object has behavior that changes its state

---

### Value Object Pattern

Value Objects are immutable objects that are defined solely by their attributes. They have no identity and are compared by their attribute values.

**Key Characteristics:**
- Immutable (once created, cannot be changed)
- No identity (equality is based on attributes)
- Self-validating in constructor
- Represents a descriptive aspect of the domain
- Replaces primitive obsession

**Implementation:**

```typescript
class Money implements ValueObject {
  private readonly amount: number;
  private readonly currency: Currency;

  constructor(amount: number, currency: Currency) {
    if (amount < 0) throw new MoneyCannotBeNegativeError();
    this.amount = Math.round(amount * 100) / 100; // Precision
    this.currency = currency;
  }

  static zero(currency: Currency): Money {
    return new Money(0, currency);
  }

  add(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  private ensureSameCurrency(other: Money): void {
    if (!this.currency.equals(other.currency)) {
      throw new CurrencyMismatchError();
    }
  }
}

record ProjectName implements ValueObject {
  private readonly value: string;

  constructor(value: string) {
    if (value === null || value.trim().length === 0) {
      throw new DomainError('Project name cannot be empty');
    }
    if (value.length > 100) {
      throw new DomainError('Project name cannot exceed 100 characters');
    }
    this.value = value.trim();
  }

  getValue(): string {
    return this.value;
  }

  equals(other: ValueObject): boolean {
    return other instanceof ProjectName && this.value === other.value;
  }

  containsKeyword(keyword: string): boolean {
    return this.value.toLowerCase().includes(keyword.toLowerCase());
  }
}
```

**When to Use Value Objects:**
- When the object represents a descriptive aspect without identity
- When objects with the same attributes should be considered equal
- When the object should be immutable
- When validation belongs with the data
- When replacing primitive types with meaningful domain concepts

---

### Aggregate Root Pattern

An Aggregate Root is a special Entity that serves as the entry point to a group of related entities and value objects (the Aggregate). The aggregate root controls access to all other objects within the aggregate and enforces consistency rules.

**Key Characteristics:**
- Is an Entity with additional responsibilities
- Controls consistency boundaries
- Is the only entry point to the aggregate
- Manages references to other objects in the aggregate
- Ensures invariants are maintained

**Implementation:**

```typescript
class Order implements AggregateRoot<OrderId> {
  private _id: OrderId;
  private _status: OrderStatus;
  private _lineItems: OrderLineItem[];
  private _customer: CustomerReference;
  private _shippingAddress: Address;
  private _paymentInfo: PaymentInfo;
  private _domainEvents: DomainEvent[];

  get id(): OrderId { return this._id; }
  get lineItems(): OrderLineItem[] { return [...this._lineItems]; }
  get status(): OrderStatus { return this._status; }

  // Public interface for modifying aggregate
  addLineItem(product: Product, quantity: Quantity): void {
    if (this._status !== OrderStatus.DRAFT) {
      throw new CannotModifyOrderError();
    }
    
    const existing = this._lineItems.find(li => li.productId.equals(product.id));
    if (existing) {
      existing.increaseQuantity(quantity);
    } else {
      this._lineItems.push(OrderLineItem.create(product, quantity));
    }
  }

  submitForFulfillment(): void {
    this.validateBusinessRules();
    this._status = OrderStatus.SUBMITTED;
    this._domainEvents.push(new OrderSubmittedEvent(this._id));
  }

  private validateBusinessRules(): void {
    if (this._lineItems.length === 0) {
      throw new OrderMustHaveAtLeastOneLineItemError();
    }
    if (!this._shippingAddress.isValid()) {
      throw new InvalidShippingAddressError();
    }
  }
}
```

**Aggregate Design Rules:**
1. **Reference by ID only**: External references should use identity, not direct object references
2. **Single entry point**: Only the aggregate root can be accessed from outside
3. **Consistency boundary**: All invariants are enforced within the aggregate
4. **Small aggregates**: Keep aggregates small to avoid performance issues

---

### Domain Events Pattern

Domain events capture something significant that happened in the domain, enabling loose coupling between bounded contexts.

**Implementation:**

```typescript
class OrderSubmitted implements DomainEvent {
  readonly orderId: OrderId;
  readonly customerId: CustomerId;
  readonly totalAmount: Money;
  readonly submittedAt: DateTime;
  readonly eventId: EventId;
  readonly occurredAt: DateTime;

  constructor(
    orderId: OrderId,
    customerId: CustomerId,
    totalAmount: Money
  ) {
    this.orderId = orderId;
    this.customerId = customerId;
    this.totalAmount = totalAmount;
    this.submittedAt = DateTime.now();
    this.eventId = EventId.generate();
    this.occurredAt = DateTime.now();
  }
}

class Order extends AggregateRoot<OrderId> {
  submit(): void {
    this._status = OrderStatus.SUBMITTED;
    this.recordEvent(new OrderSubmitted(
      this._id,
      this._customerId,
      this.calculateTotal()
    ));
  }
}
```

---

### Repository Pattern

The Repository pattern abstracts data access behind a collection-like interface, hiding persistence details from the domain.

**Implementation:**

```typescript
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  findByCustomerId(customerId: CustomerId): Promise<Order[]>;
  findByStatus(status: OrderStatus): Promise<Order[]>;
  save(order: Order): Promise<void>;
  delete(order: Order): Promise<void>;
}

class PostgresOrderRepository implements OrderRepository {
  async findById(id: OrderId): Promise<Order | null> {
    const result = await this.db.query(
      'SELECT * FROM orders WHERE id = $1',
      [id.value]
    );
    
    if (result.rows.length === 0) return null;
    
    return OrderMapper.toDomain(result.rows[0]);
  }

  async save(order: Order): Promise<void> {
    await this.db.transaction(async (tx) => {
      await tx.query(
        `INSERT INTO orders (id, status, created_at, version, data)
         VALUES ($1, $2, $3, $4, $5)
         ON CONFLICT (id) DO UPDATE SET status = $2, version = $4, data = $5`,
        [order.id.value, order.status, order.createdAt, order.version + 1, OrderMapper.toPersistence(order)]
      );
      
      for (const event of order.getUncommittedEvents()) {
        await this.eventStore.publish(event);
      }
    });
  }
}
```

---

### Factory Pattern

Factories encapsulate the creation logic for complex objects, especially when creation involves validation or requires assembling multiple components.

**Implementation:**

```typescript
class OrderFactory implements DomainService {
  constructor(
    private readonly productRepository: ProductRepository,
    private readonly pricingService: PricingService
  ) {}

  createOrder(customerId: CustomerId, items: OrderItemInput[]): Order {
    const lineItems = items.map(input => {
      const product = this.productRepository.findById(input.productId)
        .orElseThrow(() => new ProductNotFoundError(input.productId));
      
      const price = this.pricingService.calculatePrice(
        product.id,
        input.quantity,
        customerId
      );
      
      return OrderLineItem.create(product, input.quantity, price);
    });
    
    const orderId = OrderId.generate();
    return new Order(orderId, customerId, lineItems);
  }
}

class ProjectBuilder {
  private name?: ProjectName;
  private tasks: Task[] = [];
  private createdAt: Date = new Date();

  withName(name: string): this {
    this.name = ProjectName.of(name);
    return this;
  }

  withTask(task: Task): this {
    this.tasks.push(task);
    return this;
  }

  build(): Project {
    if (!this.name) {
      throw new DomainError('Project name is required');
    }
    return new Project(
      this.name,
      new TaskCollection(this.tasks),
      this.createdAt
    );
  }
}
```

---

### Domain Service Pattern

Domain Services encapsulate business logic that doesn't naturally belong to a single Entity or Value Object, especially operations that involve multiple domain objects.

**Implementation:**

```typescript
interface ProjectTransferService {
  transferTask(
    taskId: TaskId,
    fromProjectId: ProjectId,
    toCustomerId: CustomerId
  ): Promise<CqrsOutput>;
}

class OrderTransferServiceImpl implements OrderTransferService {
  constructor(
    private orderRepository: OrderRepository
  ) {}

  async execute(input: TransferItemInput): Promise<CqrsOutput> {
    const order = await this.orderRepository.findById(
      OrderId.of(input.orderId)
    );

    if (!order) {
      return CqrsOutput.create().fail().setMessage('Order not found');
    }

    const fromCustomer = order.findCustomer(CustomerName.of(input.fromCustomerName));
    const toCustomer = order.findCustomer(CustomerName.of(input.toCustomerName));

    if (!fromCustomer || !toCustomer) {
      return CqrsOutput.create().fail().setMessage('Source or target customer not found');
    }

    const item = fromCustomer.removeItem(ItemId.of(input.itemId));
    toCustomer.addItem(item.getDescription());

    await this.orderRepository.save(order);

    return CqrsOutput.create().succeed();
  }
}
```

---

### Read-Only Interface Pattern

Read-Only interfaces expose entities or value objects without modification capabilities, preventing unintended state changes.

**Implementation:**

```typescript
interface ReadOnlyProject {
  getId(): ProjectId;
  getName(): ProjectName;
  getTasks(): ReadonlyArray<ReadOnlyTask>;
  containsTask(taskId: TaskId): boolean;
  getTaskCount(): number;
}

class ReadOnlyProjectImpl implements ReadOnlyProject {
  constructor(private project: Project) {}

  getId(): ProjectId {
    return this.project.getId();
  }

  getTasks(): ReadonlyArray<ReadOnlyTask> {
    return this.project.getTasks()
      .map(task => new ReadOnlyTaskImpl(task));
  }

  containsTask(taskId: TaskId): boolean {
    return this.project.containsTask(taskId);
  }

  getTaskCount(): number {
    return this.project.getTaskCount();
  }
}

class ReadOnlyTaskImpl implements ReadOnlyTask {
  constructor(private task: Task) {}

  getId(): TaskId {
    return this.task.getId();
  }

  getDescription(): TaskDescription {
    return this.task.getDescription();
  }

  isDone(): boolean {
    return this.task.isDone();
  }

  setDone(done: boolean): never {
    throw new UnsupportedOperationException('Read-only view');
  }
}
```

**Use Cases for Read-Only Views:**
- Returning entities from query operations
- Preventing modification in presentation layer
- Sharing entities across boundaries safely
- Implementing observer patterns

---

## Context Mapping

Context Map showing relationships between bounded contexts:

```typescript
const contextMap = {
  upstream: {
    name: 'OrderManagement',
    responsibilities: [
      'Order creation',
      'Order lifecycle management',
      'Order history'
    ]
  },
  
  downstream: {
    name: 'Inventory',
    relationship: 'Customer-Supplier',
    protocol: 'Domain Events'
  },
  
  inventoryACL: {
    name: 'InventoryACL',
    role: 'Translate between contexts',
    implementations: [
      'Event consumer from Inventory context',
      'Adapter for inventory data format',
      'Translator for Inventory-specific concepts'
    ]
  },
  
  shared: {
    name: 'CoreTypes',
    sharedBy: ['OrderContext', 'CustomerContext', 'InventoryContext'],
    artifacts: [
      'Money (value object)',
      'Address (value object)',
      'Event types (domain events)'
    ]
  }
};
```

---

## DDD in AI-Assisted Development

```typescript
const dddPrompt = `Implement the following domain model following DDD principles:

Domain: E-commerce Order Processing

Bounded Context: Order Management

Entities:
- Order (aggregate root)
- OrderLineItem
- Customer

Value Objects:
- Money (with currency)
- Address
- OrderId
- Quantity

Aggregate Rules:
1. Order is the only entry point for modifications
2. Order status transitions: DRAFT → SUBMITTED → CONFIRMED → SHIPPED → DELIVERED
3. Total is calculated from line items
4. Inventory must be checked before submission

Domain Events:
- OrderCreated
- OrderSubmitted
- OrderConfirmed
- OrderShipped
- OrderCancelled

Please generate:
1. TypeScript interfaces for entities and value objects
2. Aggregate root with business logic
3. Domain events
4. Repository interface
5. Unit tests for aggregate invariants
`;
```

---

## Best Practices

1. **Start with Strategic Design**: Map bounded contexts before modeling entities

2. **Use the Ubiquitous Language**: Ensure all code reflects domain terminology

3. **Protect Aggregate Invariants**: Enforce business rules within aggregates

4. **Design Around Behavior**: Focus on what the domain does, not just data structures

5. **Separate Contexts**: Don't let bounded contexts leak into each other

6. **Leverage Domain Events**: Decouple contexts through event-driven communication

7. **Iterate with Domain Experts**: Continue refining the model through collaboration

8. **Keep Aggregates Small**: Design focused aggregates for better performance and consistency

9. **Make Value Objects Immutable**: Prevent side effects by making value objects immutable

10. **Use Factories for Complex Creation**: Encapsulate creation logic where it involves validation or assembly

---

## Anti-Patterns to Avoid

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| Anemic Domain Model | Entities without behavior | Move logic from services to domain objects |
| God Objects | One entity doing too much | Split into multiple aggregates |
| Anemic Aggregates | Violating aggregate invariants | Enforce rules in aggregate root |
| Context Bleed | Leaking concepts between contexts | Define clear boundaries |
| Anemic Services | Services with no domain logic | Move behavior to domain |
| Large Aggregates | Aggregate with too many objects | Split into smaller aggregates |
| Public Setters on Entities | Exposing state modification | Use private fields with controlled methods |
| Value Object Without Validation | Allowing invalid values | Validate in constructor |

```typescript
// WRONG - Data without behavior (Anemic Domain Model)
class Project {
  id: string;
  name: string;
  tasks: Task[];
}

// Behavior in separate service
class ProjectService {
  createProject(name: string): Project { ... }
  renameProject(project: Project, name: string): void { ... }
  addTask(project: Project, task: Task): void { ... }
}

// WRONG - Value object without validation
record Email(string value) {
  // No validation - allows invalid emails
}

// WRONG - Public setters on entities
class Project {
  public id: string;
  public name: string;  // Can be modified directly
  public tasks: Task[]; // Can be mutated externally
}
```

---

## References

1. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.

2. Vernon, Vaughn. "Implementing Domain-Driven Design." Addison-Wesley, 2013.

3. Vernon, Vaughn. "Domain-Driven Design Distilled." Addison-Wesley, 2016.

4. "Domain-Driven Design Reference" by Eric Evans

5. "Patterns, Principles, and Practices of Domain-Driven Design" by Scott Millett

6. "Effective Aggregate Design." Vaughn Vernon.

7. "Domain-Driven Design Community." https://domaindrivendesign.org/
