# Domain-Driven Design

## Overview

Domain-Driven Design (DDD) is an approach to software development that emphasizes collaboration between domain experts and developers to create a shared model of the business domain. First articulated by Eric Evans in his 2003 book "Domain-Driven Design: Tackling Complexity in the Heart of Software," DDD provides a framework for tackling complex domain problems through strategic and tactical design patterns.

## Core Concepts

### Strategic Design Patterns

Strategic design addresses the "big picture" of the domain architecture:

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

### The Ubiquitous Language

The Ubiquitous Language is the shared vocabulary used by all team members:

```typescript
// Anti-pattern: Mixing technical and domain language
class OrderService {
  async processOrd() {
    const ord = await this.repo.findById(id);
    if (ord.status !== 'PENDING') throw new Error('Invalid');
    // ... technical jargon confuses domain experts
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
    
    // Domain experts understand this:
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

## Bounded Contexts

A Bounded Context is a boundary within which a particular domain model applies:

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

  export class Order {
    submit(): void {
      if (this.status !== OrderStatus.DRAFT) {
        throw new InvalidOrderStateTransitionError();
      }
      this.status = OrderStatus.SUBMITTED;
      this.addDomainEvent(new OrderSubmittedEvent(this.id));
    }
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

## Building Blocks

### Entities

```typescript
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

### Value Objects

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
```

### Aggregates

```typescript
// Order Aggregate Root
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

### Domain Events

```typescript
// Domain Event
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

// Publishing and handling
class Order extends AggregateRoot<OrderId> {
  submit(): void {
    // Business logic
    this._status = OrderStatus.SUBMITTED;
    
    // Raise domain event
    this.recordEvent(new OrderSubmitted(
      this._id,
      this._customerId,
      this.calculateTotal()
    ));
  }
}
```

### Repositories

```typescript
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  findByCustomerId(customerId: CustomerId): Promise<Order[]>;
  findByStatus(status: OrderStatus): Promise<Order[]>;
  save(order: Order): Promise<void>;
  delete(order: Order): Promise<void>;
}

// Implementation
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
      
      // Publish domain events
      for (const event of order.getUncommittedEvents()) {
        await this.eventStore.publish(event);
      }
    });
  }
}
```

### Domain Services

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
```

## Context Mapping

```typescript
// Context Map showing relationships between bounded contexts
const contextMap = {
  // Upstream-Downstream relationship
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
  
  // Anticorruption Layer
  inventoryACL: {
    name: 'InventoryACL',
    role: 'Translate between contexts',
    implementations: [
      'Event consumer from Inventory context',
      'Adapter for inventory data format',
      'Translator for Inventory-specific concepts'
    ]
  },
  
  // Shared Kernel
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

## DDD in AI-Assisted Development

```typescript
// AI Prompts for DDD
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

## Best Practices

1. **Start with Strategic Design**: Map bounded contexts before modeling entities

2. **Use the Ubiquitous Language**: Ensure all code reflects domain terminology

3. **Protect Aggregate Invariants**: Enforce business rules within aggregates

4. **Design Around Behavior**: Focus on what the domain does, not just data structures

5. **Separate Contexts**: Don't let bounded contexts leak into each other

6. **Leverage Domain Events**: Decouple contexts through event-driven communication

7. **Iterate with Domain Experts**: Continue refining the model through collaboration

## Anti-Patterns to Avoid

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| Anemic Domain Model | Entities without behavior | Move logic from services to domain objects |
| God Objects | One entity doing too much | Split into multiple aggregates |
| Anemic Aggregates | Violating aggregate invariants | Enforce rules in aggregate root |
| Context Bleed | Leaking concepts between contexts | Define clear boundaries |
| Anemic Services | Services with no domain logic | Move behavior to domain |

## References

- "Domain-Driven Design" by Eric Evans
- "Implementing Domain-Driven Design" by Vaughn Vernon
- "Domain-Driven Design Reference" by Eric Evans
- "Patterns, Principles, and Practices of Domain-Driven Design" by Scott Millett
