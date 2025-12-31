# Clean Architecture

## Overview

Clean Architecture is a software design philosophy introduced by Robert C. Martin (Uncle Bob) that emphasizes the separation of concerns through architectural layers. The core principle is that business rules and application logic should not depend on external frameworks, databases, or delivery mechanisms.

## Core Principles

### The Dependency Rule

```
                    ┌───────────────────────────────────────┐
                    │          Frameworks & Tools          │  ← External
                    │  (Web, DB, UI, External Services)   │
                    └───────────────────┬───────────────────┘
                                        │
                    ┌───────────────────▼───────────────────┐
                    │         Interface Adapters          │  ← Entry/Exit
                    │  (Controllers, Gateways, Presenters)│
                    └───────────────────┬───────────────────┘
                                        │
                    ┌───────────────────▼───────────────────┐
                    │          Application Services        │  ← Use Cases
                    │  (Interactors, Use Case Coordinators)│
                    └───────────────────┬───────────────────┘
                                        │
                    ┌───────────────────▼───────────────────┐
                    │           Domain Layer              │  ← Business Rules
                    │  (Entities, Domain Services, Value Objects)│
                    └───────────────────────────────────────┘

     Source code dependencies point INWARD toward the domain.
     Inner layers do NOT know about outer layers.
```

### Key Principles

1. **Independence of Frameworks**: Business logic independent of frameworks
2. **Testability**: Business rules testable without UI, DB, or external systems
3. **Independence of UI**: UI can change without changing business logic
4. **Independence of Database**: Business logic independent of database
5. **Independence of External Agencies**: Business rules don't know about outside world

## Layer Structure

```typescript
src/
├── enterprise/
│   └── business-rules/        # Domain Layer ( innermost)
│       ├── entities/
│       ├── value-objects/
│       ├── domain-services/
│       └── events/
│
├── application/
│   ├── use-cases/            # Application Services
│   │   ├── create-order/
│   │   ├── process-payment/
│   │   └── ship-order/
│   ├── ports/
│   │   ├── inbound/          # Input ports (use case interfaces)
│   │   └── outbound/         # Output ports (repository interfaces)
│   └── coordinators/
│
├── adapters/
│   ├── inbound/              # Interface Adapters
│   │   ├── http/             # HTTP controllers
│   │   ├── grpc/
│   │   └── messaging/
│   ├── outbound/
│   │   ├── persistence/      # Database adapters
│   │   ├── external-api/     # External service clients
│   │   └── events/           # Event publishers
│   └── presenters/
│
└── frameworks/
    ├── web/                  # Frameworks & Drivers
    │   ├── express/
    │   ├── fastify/
    │   └── swagger/
    ├── database/
    │   ├── typeorm/
    │   ├── prisma/
    │   └── mongoose/
    └── external/
        ├── stripe/
        ├── sendgrid/
        └── aws/
```

## Implementation Examples

### Domain Layer (Enterprise Business Rules)

```typescript
// entities/order.ts
import { Entity } from '../../shared/entity';
import { Money } from '../../value-objects/money';
import { OrderId } from '../../value-objects/order-id';
import { OrderStatus } from './order-status';
import { OrderLineItem } from './order-line-item';

export interface OrderProps {
  id: OrderId;
  customerId: string;
  lineItems: OrderLineItem[];
  status: OrderStatus;
  createdAt: Date;
}

export class Order extends Entity<OrderId, OrderProps> {
  private constructor(props: OrderProps) {
    super(props);
  }

  static create(props: OrderProps): Order {
    const order = new Order(props);
    order.validate();
    return order;
  }

  private validate(): void {
    if (this._props.lineItems.length === 0) {
      throw new Error('Order must have at least one line item');
    }
    if (!this._props.status) {
      throw new Error('Order must have a status');
    }
  }

  get total(): Money {
    return this._props.lineItems.reduce(
      (sum, item) => sum.add(item.subtotal),
      Money.zero()
    );
  }

  get canBeSubmitted(): boolean {
    return this._props.status === OrderStatus.DRAFT;
  }

  submit(): void {
    if (!this.canBeSubmitted) {
      throw new Error('Order cannot be submitted');
    }
    this._props.status = OrderStatus.SUBMITTED;
    this.recordDomainEvent(new OrderSubmitted(this._props.id));
  }
}

// value-objects/money.ts
export class Money {
  private constructor(
    private readonly amount: number,
    private readonly currency: string
  ) {}

  static zero(currency: string = 'USD'): Money {
    return new Money(0, currency);
  }

  static fromAmount(amount: number, currency: string = 'USD'): Money {
    return new Money(Math.round(amount * 100) / 100, currency);
  }

  add(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  private ensureSameCurrency(other: Money): void {
    if (this.currency !== other.currency) {
      throw new Error('Cannot operate on different currencies');
    }
  }
}
```

### Application Layer (Use Cases)

```typescript
// use-cases/create-order/create-order.handler.ts
import { OrderRepository } from '../../ports/outbound/order.repository';
import { InventoryService } from '../../ports/outbound/inventory.service';
import { OrderFactory } from '../../domain/factories/order.factory';

export interface CreateOrderInput {
  customerId: string;
  items: Array<{
    productId: string;
    quantity: number;
  }>;
}

export interface CreateOrderResult {
  orderId: string;
  total: number;
  status: string;
}

export class CreateOrderHandler {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly inventoryService: InventoryService,
    private readonly orderFactory: OrderFactory
  ) {}

  async execute(input: CreateOrderInput): Promise<CreateOrderResult> {
    // Validate availability
    await this.inventoryService.checkAvailability(input.items);
    
    // Create order using factory
    const order = this.orderFactory.createFromItems(
      input.customerId,
      input.items
    );
    
    // Persist
    await this.orderRepository.save(order);
    
    // Return result
    return {
      orderId: order.id.value,
      total: order.total.amount,
      status: order.status.value
    };
  }
}

// use-cases/create-order/create-order.controller.ts
import { Request, Response } from 'express';
import { CreateOrderHandler } from './create-order.handler';
import { CreateOrderInput } from './create-order.handler';

export class CreateOrderController {
  constructor(private readonly handler: CreateOrderHandler) {}

  async handle(req: Request, res: Response): Promise<void> {
    try {
      const input: CreateOrderInput = {
        customerId: req.body.customerId,
        items: req.body.items.map((item: any) => ({
          productId: item.productId,
          quantity: item.quantity
        }))
      };

      const result = await this.handler.execute(input);

      res.status(201).json({
        success: true,
        data: result
      });
    } catch (error) {
      res.status(400).json({
        success: false,
        error: error instanceof Error ? error.message : 'Unknown error'
      });
    }
  }
}
```

### Ports (Interfaces)

```typescript
// ports/inbound/order-input.port.ts
export interface CreateOrderInputPort {
  createOrder(input: CreateOrderInput): Promise<CreateOrderOutput>;
}

// ports/outbound/order.repository.ts
export interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  findByCustomerId(customerId: string): Promise<Order[]>;
  save(order: Order): Promise<void>;
  delete(id: string): Promise<void>;
}

// ports/outbound/inventory.service.ts
export interface InventoryService {
  checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;
  reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
  releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
}
```

### Adapters (Interface Adapters)

```typescript
// adapters/outbound/typeorm-order.repository.ts
import { OrderRepository } from '../../application/ports/outbound/order.repository';
import { Order } from '../../domain/entities/order';
import { OrderMapper } from './order.mapper';

export class TypeORMOrderRepository implements OrderRepository {
  constructor(
    private readonly orderRepository: any,
    private readonly eventPublisher: EventPublisher
  ) {}

  async findById(id: string): Promise<Order | null> {
    const entity = await this.orderRepository.findOne({ where: { id } });
    if (!entity) return null;
    return OrderMapper.toDomain(entity);
  }

  async save(order: Order): Promise<void> {
    const entity = OrderMapper.toPersistence(order);
    await this.orderRepository.save(entity);
    
    // Publish domain events
    for (const event of order.getUncommittedEvents()) {
      await this.eventPublisher.publish(event);
    }
  }
}

// adapters/inbound/http-order.controller.ts
import { Request, Response } from 'express';
import { CreateOrderHandler } from '../../application/use-cases/create-order/create-order.handler';

export class HTTPOrderController {
  constructor(private readonly handler: CreateOrderHandler) {}

  async createOrder(req: Request, res: Response): Promise<void> {
    const result = await this.handler.execute(req.body);
    res.status(201).json(result);
  }

  async getOrder(req: Request, res: Response): Promise<void> {
    const order = await this.handler.getOrder(req.params.id);
    res.status(200).json(order);
  }
}
```

## Dependency Injection Structure

```typescript
// container.ts - Composition Root
class Container {
  private readonly services = new Map<string, any>();

  register<T>(token: string, factory: (container: Container) => T): void {
    this.services.set(token, factory(this));
  }

  resolve<T>(token: string): T {
    const factory = this.services.get(token);
    if (!factory) {
      throw new Error(`Service ${token} not registered`);
    }
    return factory(this);
  }
}

// Setup
const container = new Container();

// Ports
container.register('OrderRepository', () => new TypeORMOrderRepository(...));
container.register('InventoryService', () => new StripeInventoryService(...));
container.register('EventPublisher', () => new KafkaEventPublisher(...));

// Use Cases
container.register('CreateOrderHandler', (c) => 
  new CreateOrderHandler(
    c.resolve('OrderRepository'),
    c.resolve('InventoryService'),
    c.resolve('OrderFactory')
  )
);

// Controllers
container.register('CreateOrderController', (c) =>
  new HTTPOrderController(c.resolve('CreateOrderHandler'))
);
```

## Clean Architecture in AI-Assisted Development

```markdown
## AI Prompt for Clean Architecture

Generate a feature following Clean Architecture:

Context: E-commerce Payment Processing

Requirements:
1. Process payment for an order
2. Handle payment failures with retries
3. Publish events on success/failure

Generate:
1. Domain entities (Payment, PaymentResult)
2. Value objects (PaymentAmount, PaymentMethod)
3. Domain events (PaymentProcessed, PaymentFailed)
4. Use case handler (ProcessPaymentHandler)
5. Input/output ports (interfaces)
6. Repository interface for payments
7. HTTP controller adapter

Constraints:
- All dependencies point inward to domain
- Domain has no imports from outer layers
- Use TypeScript
- Include JSDoc documentation
- Add unit tests for domain rules
```

## Benefits

| Benefit | Description |
|---------|-------------|
| Testability | Test business rules without external dependencies |
| Maintainability | Changes to outer layers don't affect inner layers |
| Flexibility | Swap implementations without code changes |
| Scalability | Clear separation enables team scaling |
| Framework Independence | Business logic survives framework changes |

## Anti-Patterns to Avoid

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| Anemic Domain | Entities without behavior | Move logic from services to domain |
| Dependency Inversion Violation | Domain depending on frameworks | Use dependency injection |
| Feature Scattered Across Layers | Related code in multiple layers | Organize by feature |
| God Interface | One port for everything | Separate inbound/outbound ports |
| Skipping Layers | Controllers directly calling repositories | Always go through use cases |
| Leaking Infrastructure | Domain entities with framework annotations | Keep domain pure |
| Concrete Dependencies | Use cases depending on concrete implementations | Depend on abstractions |
| Missing Ports | Direct imports of infrastructure in core | Define ports, implement in adapters |

---

## Clean Architecture for AI-Assisted Development

### AI Coding Guidelines

When AI generates code following Clean Architecture:

```typescript
// 1. Always define ports (interfaces) in inner layers
// Domain/Application layer defines the abstraction
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  save(order: Order): Promise<void>;
}

// 2. Outer layers implement the ports
// Adapter layer implements the abstraction
class PostgresOrderRepository implements OrderRepository {
  constructor(private readonly db: Database) {}
  
  async findById(id: OrderId): Promise<Order | null> {
    const row = await this.db.query('SELECT * FROM orders WHERE id = ?', [id.value]);
    return row ? OrderMapper.toDomain(row) : null;
  }
  
  async save(order: Order): Promise<void> {
    await this.db.query('INSERT INTO orders...', [OrderMapper.toPersistence(order)]);
  }
}

// 3. Domain layer has ZERO dependencies on outer layers
// Domain entities are pure TypeScript/JavaScript
class Order {
  private constructor(
    private readonly id: OrderId,
    private status: OrderStatus,
    private readonly lineItems: OrderLineItem[]
  ) {}

  static create(id: OrderId, lineItems: OrderLineItem[]): Order {
    if (lineItems.length === 0) {
      throw new DomainError('Order must have at least one line item');
    }
    return new Order(id, OrderStatus.DRAFT, lineItems);
  }

  submit(): void {
    if (this.status !== OrderStatus.DRAFT) {
      throw new DomainError('Order cannot be submitted');
    }
    this.status = OrderStatus.SUBMITTED;
  }
}

// 4. Use case layer orchestrates domain objects through ports
class CreateOrderHandler {
  constructor(
    private readonly orderRepository: OrderRepository,  // Interface
    private readonly inventoryService: InventoryService // Interface
  ) {}

  async execute(input: CreateOrderInput): Promise<CreateOrderOutput> {
    // Validate input
    this.validateInput(input);

    // Check inventory
    await this.inventoryService.checkAvailability(input.items);

    // Create domain object
    const order = Order.create(
      OrderId.generate(),
      input.items.map(item => OrderLineItem.create(item.productId, item.quantity))
    );

    // Persist through port
    await this.orderRepository.save(order);

    return { orderId: order.id.value, status: order.status.value };
  }

  private validateInput(input: CreateOrderInput): void {
    if (!input.customerId) throw new DomainError('Customer ID required');
    if (!input.items?.length) throw new DomainError('Items required');
  }
}
```

### Decision Flowchart for AI

```
┌─────────────────────────────────────────────────────────────────┐
│                    ARCHITECTURE DECISION TREE                   │
└─────────────────────────────────────────────────────────────────┘

Is this a new feature?
│
├─► NO → Modify existing code in same layer
│
└─► YES → Is this a small, simple feature?
         │
         ├─► YES → Consider layer-based structure
         │
         └─► NO → Use vertical slice structure

What type of code is this?
│
├─► Domain logic (business rules)
│   └─► Put in feature/domain/ or domain/ layer
│
├─► Business operation orchestration
│   └─► Put in feature/usecase/ or application/ layer
│
├─► External system integration
│   └─► Put in feature/adapter/ or adapters/ layer
│
└─► Framework/setup code
    └─► Put in shared/io/ or frameworks/ layer

Does this depend on external systems?
│
├─► YES → Define port (interface) in inner layer
│        Implement in outer layer (adapter)
│
└─► NO → Can be in domain layer (pure business logic)

Is this state-modifying?
│
├─► YES → Use Command pattern (CreateXxxHandler)
│
└─► NO → Use Query pattern (GetXxxHandler or XxxQuery)

Does this span multiple features?
│
├─► YES → Coordinate through aggregate root
│        Define shared port in shared/ if needed
│
└─► NO → Keep within feature slice
```

### Common AI Mistakes to Avoid

```typescript
// ❌ WRONG - Domain depends on framework
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  status: string;
}

// ✅ CORRECT - Pure domain entity
export class Order {
  private constructor(
    private readonly id: OrderId,
    private status: OrderStatus
  ) {}

  static create(id: OrderId, status: OrderStatus): Order {
    return new Order(id, status);
  }
}

// ❌ WRONG - Controller skips use case layer
class OrderController {
  constructor(private readonly orderRepository: OrderRepository) {}

  async createOrder(req: Request): Promise<Response> {
    // Bypasses use case - direct repository call
    const order = new Order(OrderId.generate(), OrderStatus.DRAFT);
    await this.orderRepository.save(order);
    return Response.created(order.id.value);
  }
}

// ✅ CORRECT - Controller delegates to use case
class OrderController {
  constructor(private readonly createOrderHandler: CreateOrderHandler) {}

  async createOrder(req: Request): Promise<Response> {
    const result = await this.createOrderHandler.execute(req.body);
    return Response.created(result.orderId);
  }
}

// ❌ WRONG - Use case depends on concrete implementation
class CreateOrderHandler {
  constructor(private readonly repo: TypeORMOrderRepository) {}
}

// ✅ CORRECT - Use case depends on abstraction
class CreateOrderHandler {
  constructor(private readonly repo: OrderRepository) {}
}
```

---

## References

- "Clean Architecture" by Robert C. Martin
- "Architecture: The Hard Parts" by Neal Ford
- "Hands-On Domain-Driven Design with .NET Core" by Alexey Zimarev
