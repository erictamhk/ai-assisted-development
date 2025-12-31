# DDD Tactical Patterns

Tactical design patterns implement the domain model within a bounded context. These patterns translate strategic decisions (bounded contexts, ubiquitous language) into concrete code structures.

## Entity Pattern

Entities are domain objects with a distinct identity that persists through time and state changes. Unlike value objects, entities are compared by identity, not by their attributes.

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

**AI Coding Guidelines:**
- AI should generate entities with private fields and public getters
- Avoid public setters; use behavior methods instead
- Include `equals()` and `hashCode()` based on identity
- Domain events should be recorded within entity methods

---

## Value Object Pattern

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
    this.amount = Math.round(amount * 100) / 100;
    this.currency = currency;
  }

  static zero(currency: Currency): Money {
    return new Money(0, currency);
  }

  add(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount + other.amount, this.currency);
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

  getValue(): string { return this.value; }

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

**AI Coding Guidelines:**
- AI should always create immutable value objects
- Include validation in constructor, throw specific domain errors
- Implement `equals()` based on all attributes
- Provide factory methods like `of()`, `fromString()`

---

## Aggregate Root Pattern

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

**AI Coding Guidelines:**
- AI should always return copies/internal collections from getters
- AI should validate business rules within the aggregate
- AI should use defensive copying for collections
- Cross-aggregate references must use ID only

---

## Domain Events Pattern

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

**Benefits:**
- Decouples producer from consumers
- Enables eventual consistency
- Creates audit trail
- Supports event sourcing

**AI Coding Guidelines:**
- AI should generate domain events for state changes
- AI should use past tense for event names (OrderSubmitted, not SubmitOrder)
- AI should include all relevant data in events
- AI should consider idempotency in event handlers

---

## Repository Pattern

The Repository pattern abstracts data access behind a collection-like interface, hiding persistence details from the domain.

**Key Characteristics:**
- Acts like an in-memory collection
- Abstracts persistence technology
- Returns domain objects, not DTOs
- Throws specific exceptions or returns Optional/Result

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

**Best Practices:**
- Use framework GenericRepository when available
- Only include needed methods, don't create full CRUD
- Custom query methods should return domain objects, not DTOs
- Use appropriate base interface (IAppendRepository, IRepository)

**AI Coding Guidelines:**
- AI should use existing GenericInMemoryRepository framework
- AI should NOT generate custom repository interfaces with findBy methods
- AI should return Optional or null for single results
- AI should return empty collections for no results, never null

---

## Factory Pattern

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

**When to Use Factories:**
- When creation involves complex validation
- When multiple objects need to be assembled
- When creation logic should be separated from business logic
- When using the Builder pattern for complex construction

**AI Coding Guidelines:**
- AI should use factories for complex object creation
- AI should validate in factory before creating domain objects
- AI should use builder pattern when construction has optional parameters

---

## Domain Service Pattern

Domain Services encapsulate business logic that doesn't naturally belong to a single Entity or Value Object, especially operations that involve multiple domain objects.

**Implementation:**

```typescript
interface ProjectTransferService {
  transferTask(
    taskId: TaskId,
    fromProjectId: ProjectId,
    toProjectId: ProjectId
  ): Promise<CqrsOutput>;
}

class ProjectTransferServiceImpl implements ProjectTransferService {
  constructor(
    private toDoListRepository: ToDoListRepository
  ) {}

  async execute(input: TransferTaskInput): Promise<CqrsOutput> {
    const toDoList = await this.toDoListRepository.findById(
      ToDoListId.of(input.toDoListId)
    );

    if (!toDoList) {
      return CqrsOutput.create().fail().setMessage('ToDoList not found');
    }

    const fromProject = toDoList.findProject(ProjectName.of(input.fromProjectName));
    const toProject = toDoList.findProject(ProjectName.of(input.toProjectName));

    if (!fromProject || !toProject) {
      return CqrsOutput.create().fail().setMessage('Source or target project not found');
    }

    const task = fromProject.removeTask(TaskId.of(input.taskId));
    toProject.addTask(task.getDescription());

    await this.toDoListRepository.save(toDoList);

    return CqrsOutput.create().succeed();
  }
}
```

**When to Use Domain Services:**
- Operations involving multiple aggregates
- Cross-aggregate transactions
- Logic that doesn't belong to a single entity
- Coordination logic between bounded contexts

**AI Coding Guidelines:**
- AI should prefer putting behavior in entities/value objects first
- AI should use domain services only when logic spans multiple aggregates
- AI should coordinate through aggregate roots, not directly modify entities

---

## Read-Only Interface Pattern

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

**AI Coding Guidelines:**
- AI should create read-only views for query results
- AI should throw UnsupportedOperationException for mutation attempts
- AI should use ReadonlyArray for collections

---

## Domain Event Pattern Language

The following patterns form a cohesive language for domain modeling:

| Pattern | Problem | Solution |
|---------|---------|----------|
| **AGGREGATE** | Maintaining consistency of related objects | Group objects with a single entry point |
| **ENTITY** | Objects with identity that persist through changes | Use ID-based equality, encapsulate behavior |
| **VALUE OBJECT** | Concepts without identity | Create immutable, attribute-equal objects |
| **DOMAIN EVENT** | Capturing and communicating state changes | Model events as immutable past-tense objects |
| **BOUNDED CONTEXT** | Managing complexity in large domains | Divide into explicit boundaries |
| **REPOSITORY** | Accessing aggregates from persistence | Provide collection-like interface |
| **SERVICE LAYER** | Coordinating multiple domain operations | Define orchestration layer |

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
  public name: string;
  public tasks: Task[];
}
```

---

## AI Coding Specific Guidelines

When AI generates DDD tactical patterns:

1. **Always use Value Objects for primitives**: Replace String, int with meaningful types
2. **Aggregate roots control access**: Never allow direct entity modification
3. **Repositories use generic framework**: Don't create custom repository interfaces
4. **Domain events for state changes**: Every significant state change publishes an event
5. **Validation in constructors**: Value objects validate themselves
6. **Business rules in aggregates**: Invariants are enforced within the aggregate
7. **Read-only views for queries**: Prevent accidental modification
8. **Factory pattern for complex creation**: Encapsulate construction logic

---

## References

1. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
2. Vernon, Vaughn. "Implementing Domain-Driven Design." Addison-Wesley, 2013.
3. Vernon, Vaughn. "Domain-Driven Design Distilled." Addison-Wesley, 2016.
4. "Effective Aggregate Design." Vaughn Vernon.
