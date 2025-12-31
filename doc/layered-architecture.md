# Layered Architecture (Ports and Adapters / Clean Architecture)

## Overview

Layered Architecture, also known as **Hexagonal Architecture** (Ports and Adapters) or **Clean Architecture**, is a software design philosophy that emphasizes the separation of concerns through architectural layers. The core principle is that business rules and application logic should remain independent of external frameworks, databases, delivery mechanisms, and user interfaces.

This architectural pattern was formalized by Alistair Cockburn as Hexagonal Architecture and further developed by Robert C. Martin (Uncle Bob) as Clean Architecture. Both approaches share the same fundamental insight: business logic should not depend on databases, web frameworks, or any external system. Instead, external systems depend on abstractions defined by the business logic. This inversion of dependencies ensures that the core application can be tested independently of its delivery mechanisms and that external systems can be replaced without changing business code.

The architecture creates clear boundaries between what the application does (core/domain) and how it integrates with the outside world (adapters). This separation enables teams to work on different parts independently, allows technology choices to be deferred, and ensures that core business logic outlives specific technology choices.

---

## Core Principles

### The Dependency Rule

The most critical rule in layered architectures is that **dependencies always point inward** toward the domain. This is the Dependency Inversion Principle applied at the architectural level. Inner layers do not know about outer layers, and all source code dependencies point toward the more abstract inner layers.

```
                    ┌───────────────────────────────────────┐
                    │          External Systems             │  ← Outermost Layer
                    │  (Web, DB, UI, External Services)    │
                    └───────────────────┬───────────────────┘
                                        │
                    ┌───────────────────▼───────────────────┐
                    │            Adapters Layer             │  ← Interface Adapters
                    │  (Controllers, Gateways, Presenters,  │
                    │   Database Adapters, External Clients)│
                    └───────────────────┬───────────────────┘
                                        │
                    ┌───────────────────▼───────────────────┐
                    │         Application Layer             │  ← Use Cases / Services
                    │  (Interactors, Use Case Coordinators, │
                    │   Application Services)               │
                    └───────────────────┬───────────────────┘
                                        │
                    ┌───────────────────▼───────────────────┐
                    │            Domain Layer               │  ← Innermost Layer
                    │  (Entities, Value Objects, Aggregates,│
                    │   Domain Services, Domain Events)     │
                    └───────────────────────────────────────┘

     Source code dependencies point INWARD toward the domain.
     Inner layers do NOT know about outer layers.
```

### Key Principles

1. **Independence of Frameworks**: Business logic remains independent of frameworks and can be tested without them
2. **Testability**: Business rules are testable without UI, database, or external systems
3. **Independence of UI**: User interfaces can change without affecting business logic
4. **Independence of Database**: Business logic is decoupled from database implementation details
5. **Independence of External Agencies**: Business rules have no knowledge of the outside world
6. **Framework Independence**: Business logic survives framework changes and upgrades

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LAYERED ARCHITECTURE                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│                         ┌─────────────────────┐                         │
│                         │                     │                         │
│                         │        DOMAIN       │                         │
│                         │        LAYER        │                         │
│                         │                     │                         │
│                         │  ┌───────────────┐  │                         │
│                         │  │   Entities    │  │                         │
│                         │  │  Value Objects│  │                         │
│                         │  │  Aggregates   │  │                         │
│                         │  │  Domain Events│  │                         │
│                         │  └───────────────┘  │                         │
│                         │                     │                         │
│                         └──────────┬──────────┘                         │
│                                    │                                    │
│                                    ▼                                    │
│                         ┌─────────────────────┐                         │
│                         │     APPLICATION     │                         │
│                         │        LAYER        │                         │
│                         │                     │                         │
│                         │  ┌───────────────┐  │                         │
│                         │  │   Use Cases   │  │                         │
│                         │  │   Services    │  │                         │
│                         │  │   Ports       │  │                         │
│                         │  └───────────────┘  │                         │
│                         │                     │                         │
│                         └──────────┬──────────┘                         │
│                                    │                                    │
│              ┌─────────────────────┼─────────────────────┐              │
│              │                     │                     │              │
│              ▼                     ▼                     ▼              │
│    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │
│    │     PORTS       │  │                 │  │     PORTS       │       │
│    │   (Interfaces)  │  │                 │  │   (Interfaces)  │       │
│    │                 │  │                 │  │                 │       │
│    │  • Input Ports  │  │                 │  │  • Output Ports │       │
│    │    (Use Cases)  │  │                 │  │  • Repositories │       │
│    │                 │  │                 │  │  • Presenters   │       │
│    └────────┬────────┘  │                 │  └────────┬────────┘       │
│             │           │                 │           │                │
│             │           │                 │           │                │
│             ▼           │                 │           ▼                │
│    ┌─────────────────┐  │                 │  ┌─────────────────┐       │
│    │   ADAPTERS      │  │                 │  │   ADAPTERS      │       │
│    │   (Inbound)     │  │                 │  │   (Outbound)    │       │
│    │                 │  │                 │  │                 │       │
│    │  • Web/HTTP     │  │                 │  │  • Database     │       │
│    │  • Console/CLI  │  │                 │  │  • File System  │       │
│    │  • API/Gateway  │  │                 │  │  • External APIs│       │
│    │  • Messaging    │  │                 │  │  • Messaging    │       │
│    │  • Mobile       │  │                 │  │  • Event Bus    │       │
│    └─────────────────┘  │                 │  └─────────────────┘       │
│                         │                 │                            │
└─────────────────────────┼─────────────────┼────────────────────────────┘
                          │                 │
                          ▼                 ▼
              ┌───────────────────┐ ┌───────────────────┐
              │    EXTERNAL       │ │    EXTERNAL       │
              │    SYSTEMS        │ │    SYSTEMS        │
              │                   │ │                   │
              │  • Web Browsers   │ │  • Databases      │
              │  • Mobile Apps    │ │  • File Systems   │
              │  • CLI Users      │ │  • Web Services   │
              │  • API Clients    │ │  • Message Queues │
              └───────────────────┘ └───────────────────┘
```

---

## The Three Core Layers

### Domain Layer (Core)

The domain layer contains the business logic of the application at its purest form. It consists of entities, value objects, aggregates, domain services, and domain events. This layer has **zero dependencies** on any other layer. It defines interfaces (ports) that describe what it needs from the outside world. The domain layer represents the heart of the application and should be completely isolated from infrastructure concerns.

The domain layer is where the most stable code resides because it contains the core business rules that rarely change, regardless of how the application is delivered or what technologies are used. This layer can be thoroughly tested without any external dependencies, making it the most reliable and valuable part of the codebase.

### Application Layer (Use Cases)

The application layer contains application-specific business logic that orchestrates operations. It coordinates between the domain layer and external systems by using domain objects and calling through ports to interact with external concerns. This layer depends only on domain abstractions and contains use cases, application services, and input/output port definitions.

The application layer defines what the system does without caring about how it communicates with the outside world. Each use case represents a single business operation and follows a consistent pattern of validating input, performing business operations, persisting changes, and returning results.

### Adapter Layer (External Integration)

The adapter layer contains implementations of ports that integrate with external systems. Adapters are divided into two categories: inbound adapters handle user input and external requests, while outbound adapters handle communication with external systems. Adapters depend on ports (interfaces), never the reverse.

This layer translates between external protocols and internal abstractions, allowing the core application to remain unchanged regardless of how it is accessed or what external systems it integrates with.

---

## Ports and Adapters Explained

### Ports (Interfaces)

Ports define the boundary between the application core and external systems. They are abstract contracts that specify what the application expects from and provides to the outside world. Ports create a stable interface that shields the core from changes in external systems.

**Input Ports (Use Case Interfaces)**

Input ports define operations that external systems can perform on the application. They represent the application's API for incoming operations. These ports are implemented by application services and called by inbound adapters.

```typescript
// Input Port - What the outside world can DO
interface CreateOrderUseCase {
  execute(input: CreateOrderInput): Promise<CreateOrderOutput>;
}

interface GetOrderUseCase {
  execute(input: GetOrderInput): Promise<GetOrderOutput>;
}

interface ProcessPaymentUseCase {
  execute(input: ProcessPaymentInput): Promise<ProcessPaymentOutput>;
}

interface ShipOrderUseCase {
  execute(input: ShipOrderInput): Promise<ShipOrderOutput>;
}
```

**Output Ports (Repository and Service Interfaces)**

Output ports define operations that the application needs to perform on external systems. These ports are implemented by outbound adapters and called by application services and domain logic. Output ports decouple the core from specific implementations of persistence, external services, and other infrastructure concerns.

```typescript
// Output Port - What the application NEEDS from external systems
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  findByCustomerId(customerId: string): Promise<Order[]>;
  save(order: Order): Promise<void>;
  delete(id: OrderId): Promise<void>;
}

interface InventoryService {
  checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;
  reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
  releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
}

interface PaymentGateway {
  processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;
  refundPayment(transactionId: string): Promise<RefundResult>;
}

interface EventPublisher {
  publish(event: DomainEvent): Promise<void>;
  publishAll(events: DomainEvent[]): Promise<void>;
}

interface NotificationService {
  sendNotification(customerId: string, message: string): Promise<void>;
}
```

### Adapters (Implementations)

Adapters implement ports and handle the specifics of interacting with external systems. They translate between external protocols and internal abstractions, keeping the core completely unaware of how external communication happens.

**Inbound Adapters (Delivery Mechanisms)**

Inbound adapters translate external input into calls to input ports. They handle the specifics of receiving requests from various delivery mechanisms and translating them into a format the application can understand.

```typescript
// HTTP Controller (Inbound Adapter)
class CreateOrderController {
  constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}

  async handleRequest(request: HttpRequest): Promise<HttpResponse> {
    try {
      const input: CreateOrderInput = {
        customerId: request.body.customerId,
        items: request.body.items.map((item: any) => ({
          productId: item.productId,
          quantity: item.quantity
        })),
        shippingAddress: request.body.shippingAddress,
        paymentMethod: request.body.paymentMethod
      };

      const result = await this.createOrderUseCase.execute(input);

      return new HttpResponse(
        result.isSuccess ? 201 : 400,
        {
          success: result.isSuccess,
          data: result.isSuccess ? result.orderId : null,
          error: result.isSuccess ? null : result.error
        }
      );
    } catch (error) {
      return new HttpResponse(
        500,
        {
          success: false,
          error: error instanceof Error ? error.message : 'Unknown error'
        }
      );
    }
  }
}

// gRPC Controller (Inbound Adapter)
class OrderGrpcController {
  constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}

  async createOrder(call: ServerUnaryCall<CreateOrderRequest>): Promise<CreateOrderResponse> {
    const input: CreateOrderInput = {
      customerId: call.request.customerId,
      items: call.request.items.map(item => ({
        productId: item.productId,
        quantity: item.quantity
      }))
    };

    const result = await this.createOrderUseCase.execute(input);

    return {
      success: result.isSuccess,
      orderId: result.isSuccess ? result.orderId : '',
      error: result.error || ''
    };
  }
}

// Message Queue Consumer (Inbound Adapter)
class OrderMessageHandler {
  constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}

  async handleMessage(message: OrderMessage): Promise<void> {
    const input: CreateOrderInput = {
      customerId: message.customerId,
      items: message.items
    };

    const result = await this.createOrderUseCase.execute(input);

    if (!result.isSuccess) {
      throw new Error(`Failed to process order: ${result.error}`);
    }
  }
}

// Console Controller (Inbound Adapter)
class OrderConsoleController {
  constructor(
    private readonly createOrderUseCase: CreateOrderUseCase,
    private readonly getOrderUseCase: GetOrderUseCase
  ) {}

  handleCommand(args: string[]): Response {
    const command = args[0];

    switch (command) {
      case 'create':
        return this.handleCreateOrder(args);
      case 'get':
        return this.handleGetOrder(args);
      default:
        return Response.error('Unknown command');
    }
  }

  private handleCreateOrder(args: string[]): Response {
    const input: CreateOrderInput = {
      customerId: args[1],
      items: JSON.parse(args[2])
    };

    const result = this.createOrderUseCase.execute(input);
    return Response.of(result);
  }

  private handleGetOrder(args: string[]): Response {
    const input: GetOrderInput = { orderId: args[1] };
    const result = this.getOrderUseCase.execute(input);
    return Response.of(result);
  }
}
```

**Outbound Adapters (External System Integrations)**

Outbound adapters implement output ports and handle communication with external systems. They translate between internal abstractions and external system specifics.

```typescript
// TypeOutbound Adapter)
class TypeORMOrderORM Repository Implementation (Repository implements OrderRepository {
  constructor(
    private readonly entityManager: EntityManager,
    private readonly eventPublisher: EventPublisher
  ) {}

  async findById(id: OrderId): Promise<Order | null> {
    const entity = await this.entityManager.findOne(OrderEntity, {
      where: { id: id.value }
    });

    if (!entity) return null;

    return OrderMapper.toDomain(entity);
  }

  async findByCustomerId(customerId: string): Promise<Order[]> {
    const entities = await this.entityManager.find(OrderEntity, {
      where: { customerId }
    });

    return entities.map(OrderMapper.toDomain);
  }

  async save(order: Order): Promise<void> {
    const entity = OrderMapper.toPersistence(order);

    if (order.isNew) {
      await this.entityManager.save(entity);
    } else {
      await this.entityManager.update(OrderEntity, entity.id, entity);
    }

    for (const event of order.getUncommittedEvents()) {
      await this.eventPublisher.publish(event);
    }
  }

  async delete(id: OrderId): Promise<void> {
    await this.entityManager.delete(OrderEntity, { id: id.value });
  }
}

// In-Memory Repository (Outbound Adapter for Testing)
class InMemoryOrderRepository implements OrderRepository {
  private orders: Map<string, Order> = new Map();

  async findById(id: OrderId): Promise<Order | null> {
    return this.orders.get(id.value) || null;
  }

  async findByCustomerId(customerId: string): Promise<Order[]> {
    return Array.from(this.orders.values()).filter(
      order => order.customerId === customerId
    );
  }

  async save(order: Order): Promise<void> {
    this.orders.set(order.id.value, order);
  }

  async delete(id: OrderId): Promise<void> {
    this.orders.delete(id.value);
  }
}

// Stripe Payment Gateway (Outbound Adapter)
class StripePaymentGateway implements PaymentGateway {
  constructor(
    private readonly stripeClient: StripeClient,
    private readonly webhookSecret: string
  ) {}

  async processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult> {
    try {
      const paymentIntent = await this.stripeClient.paymentIntents.create({
        amount: amount.inCents,
        currency: amount.currency.toLowerCase(),
        payment_method: paymentMethod.stripeToken,
        confirm: true,
        automatic_payment_methods: {
          enabled: true,
          allow_redirects: 'never'
        }
      });

      return {
        success: paymentIntent.status === 'succeeded',
        transactionId: paymentIntent.id,
        error: paymentIntent.last_payment_error?.message
      };
    } catch (error) {
      return {
        success: false,
        transactionId: '',
        error: error instanceof Error ? error.message : 'Payment failed'
      };
    }
  }

  async refundPayment(transactionId: string): Promise<RefundResult> {
    try {
      const refund = await this.stripeClient.refunds.create({
        payment_intent: transactionId
      });

      return {
        success: refund.status === 'succeeded',
        refundId: refund.id
      };
    } catch (error) {
      return {
        success: false,
        refundId: '',
        error: error instanceof Error ? error.message : 'Refund failed'
      };
    }
  }
}

// Kafka Event Publisher (Outbound Adapter)
class KafkaEventPublisher implements EventPublisher {
  constructor(
    private readonly kafkaProducer: KafkaProducer,
    private readonly topicPrefix: string
  ) {}

  async publish(event: DomainEvent): Promise<void> {
    const topic = `${this.topicPrefix}.${event.constructor.name}`;
    const message = {
      key: event.aggregateId,
      value: JSON.stringify(event),
      timestamp: Date.now()
    };

    await this.kafkaProducer.send({ topic, messages: [message] });
  }

  async publishAll(events: DomainEvent[]): Promise<void> {
    const messages = events.map(event => ({
      topic: `${this.topicPrefix}.${event.constructor.name}`,
      key: event.aggregateId,
      value: JSON.stringify(event),
      timestamp: Date.now()
    }));

    await this.kafkaProducer.send({ messages });
  }
}
```

---

## Layer Structure

```
src/
├── domain/                           # Domain Layer (innermost)
│   ├── entities/                     # Domain entities with behavior
│   │   ├── order.ts
│   │   ├── customer.ts
│   │   └── product.ts
│   ├── value-objects/                # Value objects with validation
│   │   ├── money.ts
│   │   ├── address.ts
│   │   └── email.ts
│   ├── aggregates/                   # Aggregate roots
│   │   ├── order-aggregate.ts
│   │   └── customer-aggregate.ts
│   ├── domain-services/              # Domain services for complex logic
│   │   └── pricing-service.ts
│   ├── events/                       # Domain events
│   │   ├── order-created.event.ts
│   │   └── payment-completed.event.ts
│   ├── repositories/                 # Repository interfaces (ports)
│   │   ├── order.repository.interface.ts
│   │   └── customer.repository.interface.ts
│   ├── services/                     # Service interfaces (ports)
│   │   ├── inventory.service.interface.ts
│   │   └── payment.service.interface.ts
│   └── factories/                    # Domain factories
│       └── order.factory.ts
│
├── application/                      # Application Layer
│   ├── use-cases/                    # Use case implementations
│   │   ├── create-order/
│   │   │   ├── create-order.handler.ts
│   │   │   ├── create-order.input.ts
│   │   │   └── create-order.output.ts
│   │   ├── process-payment/
│   │   │   ├── process-payment.handler.ts
│   │   │   ├── process-payment.input.ts
│   │   │   └── process-payment.output.ts
│   │   └── ship-order/
│   │       ├── ship-order.handler.ts
│   │       ├── ship-order.input.ts
│   │       └── ship-order.output.ts
│   ├── ports/
│   │   ├── inbound/                  # Input port interfaces
│   │   │   ├── create-order.port.ts
│   │   │   └── process-payment.port.ts
│   │   └── outbound/                 # Output port interfaces
│   │       ├── order.repository.port.ts
│   │       └── payment-gateway.port.ts
│   └── coordinators/                 # Application coordinators
│       └── order-coordinator.ts
│
├── adapters/                         # Adapter Layer
│   ├── inbound/                      # Inbound adapters (controllers)
│   │   ├── http/
│   │   │   ├── controllers/
│   │   │   │   ├── order.controller.ts
│   │   │   │   └── payment.controller.ts
│   │   │   ├── middlewares/
│   │   │   │   ├── auth.middleware.ts
│   │   │   │   └── validation.middleware.ts
│   │   │   └── routes/
│   │   │       ├── order.routes.ts
│   │   │       └── payment.routes.ts
│   │   ├── grpc/
│   │   │   └── order.grpc.controller.ts
│   │   ├── messaging/
│   │   │   ├── order.message-handler.ts
│   │   │   └── payment.message-handler.ts
│   │   └── cli/
│   │       └── order.cli.controller.ts
│   ├── outbound/                     # Outbound adapters (implementations)
│   │   ├── persistence/
│   │   │   ├── typeorm/
│   │   │   │   ├── order.repository.ts
│   │   │   │   ├── customer.repository.ts
│   │   │   │   └── entities/
│   │   │   │       ├── order.entity.ts
│   │   │   │       └── customer.entity.ts
│   │   │   ├── prisma/
│   │   │   │   ├── order.repository.ts
│   │   │   │   └── customer.repository.ts
│   │   │   └── mongodb/
│   │   │       ├── order.repository.ts
│   │   │       └── customer.repository.ts
│   │   ├── external-api/
│   │   │   ├── stripe/
│   │   │   │   ├── payment-gateway.adapter.ts
│   │   │   │   └── stripe-client.ts
│   │   │   ├── sendgrid/
│   │   │   │   └── notification.adapter.ts
│   │   │   └── aws/
│   │   │       └── email.service.ts
│   │   ├── events/
│   │   │   ├── kafka/
│   │   │   │   └── event-publisher.ts
│   │   │   └── rabbitmq/
│   │   │       └── event-publisher.ts
│   │   └── cache/
│   │       └── redis/
│   │           └── cache.adapter.ts
│   └── presenters/                   # Response formatters
│       ├── json/
│       │   └── order.presenter.ts
│       └── grpc/
│           └── order.presenter.ts
│
├── frameworks/                       # Frameworks and Drivers
│   ├── web/
│   │   ├── express/
│   │   │   ├── app.ts
│   │   │   └── server.ts
│   │   ├── fastify/
│   │   │   ├── app.ts
│   │   │   └── server.ts
│   │   └── nestjs/                   # NestJS modules
│   │       ├── order.module.ts
│   │       └── payment.module.ts
│   ├── database/
│   │   ├── typeorm/
│   │   │   ├── data-source.ts
│   │   │   └── migrations/
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   └── client.ts
│   │   └── mongoose/
│   │       ├── connection.ts
│   │       └── schemas/
│   ├── messaging/
│   │   ├── kafka/
│   │   │   ├── producer.ts
│   │   │   └── consumer.ts
│   │   └── rabbitmq/
│   │       ├── connection.ts
│   │       └── publisher.ts
│   └── external/
│       ├── stripe/
│       │   └── client.ts
│       ├── sendgrid/
│       │   └── client.ts
│       └── aws/
│           ├── config.ts
│           └── client.ts
│
└── shared/
    ├── utils/                        # Shared utilities
    ├── types/                        # Shared TypeScript types
    ├── errors/                       # Custom error classes
    └── mappers/                      # Cross-layer mappers
```

---

## Implementation Examples

### Domain Layer (Enterprise Business Rules)

The domain layer contains pure business logic with no dependencies on external frameworks or libraries. This makes it completely testable and portable.

```typescript
// domain/entities/order.ts
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
  shippingAddress: Address;
  paymentMethod: PaymentMethod;
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
      throw new DomainError('Order must have at least one line item');
    }
    if (!this._props.status) {
      throw new DomainError('Order must have a status');
    }
    if (!this._props.shippingAddress) {
      throw new DomainError('Order must have a shipping address');
    }
  }

  get total(): Money {
    return this._props.lineItems.reduce(
      (sum, item) => sum.add(item.subtotal),
      Money.zero(this._props.lineItems[0]?.productCurrency ?? 'USD')
    );
  }

  get canBeSubmitted(): boolean {
    return this._props.status === OrderStatus.DRAFT;
  }

  get canBeCancelled(): boolean {
    return this._props.status !== OrderStatus.SHIPPED &&
           this._props.status !== OrderStatus.DELIVERED;
  }

  submit(): void {
    if (!this.canBeSubmitted) {
      throw new DomainError('Order cannot be submitted');
    }
    this._props.status = OrderStatus.SUBMITTED;
    this.recordDomainEvent(new OrderSubmitted(this._props.id));
  }

  cancel(): void {
    if (!this.canBeCancelled) {
      throw new DomainError('Order cannot be cancelled');
    }
    this._props.status = OrderStatus.CANCELLED;
    this.recordDomainEvent(new OrderCancelled(this._props.id));
  }

  ship(): void {
    if (this._props.status !== OrderStatus.PAID) {
      throw new DomainError('Order must be paid before shipping');
    }
    this._props.status = OrderStatus.SHIPPED;
    this.recordDomainEvent(new OrderShipped(this._props.id));
  }
}

// domain/value-objects/money.ts
export class Money {
  private constructor(
    private readonly amount: number,
    private readonly currency: string
  ) {}

  static zero(currency: string = 'USD'): Money {
    return new Money(0, currency);
  }

  static fromAmount(amount: number, currency: string = 'USD'): Money {
    if (amount < 0) {
      throw new DomainError('Amount cannot be negative');
    }
    return new Money(Math.round(amount * 100) / 100, currency);
  }

  add(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount + other.amount, this.currency);
  }

  subtract(other: Money): Money {
    this.ensureSameCurrency(other);
    const result = this.amount - other.amount;
    if (result < 0) {
      throw new DomainError('Result cannot be negative');
    }
    return new Money(result, this.currency);
  }

  multiply(factor: number): Money {
    if (factor < 0) {
      throw new DomainError('Factor cannot be negative');
    }
    return new Money(this.amount * factor, this.currency);
  }

  isGreaterThan(other: Money): boolean {
    this.ensureSameCurrency(other);
    return this.amount > other.amount;
  }

  isLessThan(other: Money): boolean {
    this.ensureSameCurrency(other);
    return this.amount < other.amount;
  }

  get amountValue(): number {
    return this.amount;
  }

  get currencyCode(): string {
    return this.currency;
  }

  private ensureSameCurrency(other: Money): void {
    if (this.currency !== other.currency) {
      throw new DomainError('Cannot operate on different currencies');
    }
  }
}

// domain/aggregates/order-aggregate.ts
export class OrderAggregate {
  private constructor(
    public readonly order: Order,
    private readonly lineItems: OrderLineItem[],
    private readonly payments: Payment[]
  ) {}

  static createFromOrder(order: Order): OrderAggregate {
    return new OrderAggregate(order, [], []);
  }

  get totalPaid(): Money {
    return this.payments
      .filter(p => p.status === PaymentStatus.COMPLETED)
      .reduce((sum, payment) => sum.add(payment.amount), Money.zero());
  }

  isFullyPaid(): boolean {
    return this.totalPaid.isGreaterThanOrEqual(this.order.total);
  }

  addPayment(payment: Payment): void {
    if (this.isFullyPaid()) {
      throw new DomainError('Order is already fully paid');
    }
    this.payments.push(payment);
  }
}

// domain/events/order-submitted.event.ts
export class OrderSubmitted extends DomainEvent {
  constructor(
    public readonly orderId: OrderId,
    public readonly customerId: string,
    public readonly total: Money
  ) {
    super('OrderSubmitted', orderId.value);
  }
}
```

### Application Layer (Use Cases)

Application layer contains use case implementations that orchestrate domain objects and external adapters through ports.

```typescript
// application/use-cases/create-order/create-order.handler.ts
import { OrderRepository } from '../../ports/outbound/order.repository';
import { InventoryService } from '../../ports/outbound/inventory.service';
import { PaymentGateway } from '../../ports/outbound/payment-gateway';
import { EventPublisher } from '../../ports/outbound/event-publisher';
import { OrderFactory } from '../../domain/factories/order.factory';

export interface CreateOrderInput {
  customerId: string;
  items: Array<{
    productId: string;
    quantity: number;
  }>;
  shippingAddress: Address;
  paymentMethod: PaymentMethod;
}

export interface CreateOrderOutput {
  isSuccess: boolean;
  orderId?: string;
  total?: number;
  error?: string;
}

export class CreateOrderHandler {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly inventoryService: InventoryService,
    private readonly paymentGateway: PaymentGateway,
    private readonly eventPublisher: EventPublisher,
    private readonly orderFactory: OrderFactory
  ) {}

  async execute(input: CreateOrderInput): Promise<CreateOrderOutput> {
    try {
      // Step 1: Validate input
      this.validateInput(input);

      // Step 2: Check inventory availability
      await this.inventoryService.checkAvailability(input.items);

      // Step 3: Create order using factory
      const order = this.orderFactory.createFromItems(
        input.customerId,
        input.items,
        input.shippingAddress,
        input.paymentMethod
      );

      // Step 4: Persist order
      await this.orderRepository.save(order);

      // Step 5: Publish domain events
      await this.eventPublisher.publishAll(order.getUncommittedEvents());

      // Step 6: Return success result
      return {
        isSuccess: true,
        orderId: order.id.value,
        total: order.total.amountValue
      };
    } catch (error) {
      return {
        isSuccess: false,
        error: error instanceof Error ? error.message : 'Unknown error'
      };
    }
  }

  private validateInput(input: CreateOrderInput): void {
    if (!input.customerId) {
      throw new DomainError('Customer ID is required');
    }
    if (!input.items || input.items.length === 0) {
      throw new DomainError('Order must have at least one item');
    }
    if (!input.shippingAddress) {
      throw new DomainError('Shipping address is required');
    }
    if (!input.paymentMethod) {
      throw new DomainError('Payment method is required');
    }
  }
}

// application/use-cases/process-payment/process-payment.handler.ts
export interface ProcessPaymentInput {
  orderId: string;
  paymentToken: string;
  amount: number;
  currency: string;
}

export interface ProcessPaymentOutput {
  isSuccess: boolean;
  transactionId?: string;
  error?: string;
}

export class ProcessPaymentHandler {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly paymentGateway: PaymentGateway
  ) {}

  async execute(input: ProcessPaymentInput): Promise<ProcessPaymentOutput> {
    try {
      // Step 1: Load order
      const order = await this.orderRepository.findById(
        OrderId.of(input.orderId)
      );

      if (!order) {
        return {
          isSuccess: false,
          error: 'Order not found'
        };
      }

      // Step 2: Create payment
      const amount = Money.fromAmount(input.amount, input.currency);
      const paymentMethod = PaymentMethod.fromToken(input.paymentToken);

      // Step 3: Process payment through gateway
      const result = await this.paymentGateway.processPayment(
        amount,
        paymentMethod
      );

      if (!result.success) {
        order.recordDomainEvent(new PaymentFailed(order.id, result.error!));
        await this.orderRepository.save(order);

        return {
          isSuccess: false,
          error: result.error
        };
      }

      // Step 4: Update order status
      order.markAsPaid(result.transactionId!);
      await this.orderRepository.save(order);

      // Step 5: Return success
      return {
        isSuccess: true,
        transactionId: result.transactionId
      };
    } catch (error) {
      return {
        isSuccess: false,
        error: error instanceof Error ? error.message : 'Payment processing failed'
      };
    }
  }
}
```

### Ports (Interfaces)

Ports define the contracts between layers, enabling dependency inversion.

```typescript
// application/ports/inbound/create-order.port.ts
export interface CreateOrderInputPort {
  createOrder(input: CreateOrderInput): Promise<CreateOrderOutput>;
}

// application/ports/outbound/order.repository.ts
export interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  findByCustomerId(customerId: string): Promise<Order[]>;
  findByStatus(status: OrderStatus): Promise<Order[]>;
  save(order: Order): Promise<void>;
  delete(id: OrderId): Promise<void>;
}

// application/ports/outbound/inventory.service.ts
export interface InventoryService {
  checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;
  reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
  releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
  getProduct(productId: string): Promise<Product | null>;
}

// application/ports/outbound/payment-gateway.ts
export interface PaymentGateway {
  processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;
  refundPayment(transactionId: string, amount?: Money): Promise<RefundResult>;
  getPaymentStatus(transactionId: string): Promise<PaymentStatus>;
}

// application/ports/outbound/event-publisher.ts
export interface EventPublisher {
  publish(event: DomainEvent): Promise<void>;
  publishAll(events: DomainEvent[]): Promise<void>;
}

// application/ports/outbound/notification.service.ts
export interface NotificationService {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
  sendSms(to: string, message: string): Promise<void>;
  sendPushNotification(userId: string, notification: PushNotification): Promise<void>;
}
```

---

## Dependency Injection Structure

The composition root wires all dependencies together, following the dependency inversion principle.

```typescript
// infrastructure/container.ts - Composition Root
class Container {
  private readonly services = new Map<string, any>();
  private readonly factories = new Map<string, (container: Container) => any>();

  register<T>(token: string, factory: (container: Container) => T): void {
    this.factories.set(token, factory);
  }

  resolve<T>(token: string): T {
    if (this.services.has(token)) {
      return this.services.get(token);
    }

    const factory = this.factories.get(token);
    if (!factory) {
      throw new Error(`Service ${token} not registered`);
    }

    const instance = factory(this);
    this.services.set(token, instance);
    return instance;
  }

  registerSingleton<T>(token: string, instance: T): void {
    this.services.set(token, instance);
  }
}

// Setup dependency container
const container = new Container();

// Register ports (abstractions)
container.register('OrderRepository', () => new TypeORMOrderRepository(
  container.resolve('EntityManager'),
  container.resolve('EventPublisher')
));

container.register('InventoryService', () => new InventoryServiceImpl(
  container.resolve('InventoryRepository')
));

container.register('PaymentGateway', () => new StripePaymentGateway(
  container.resolve('StripeClient')
));

container.register('EventPublisher', () => new KafkaEventPublisher(
  container.resolve('KafkaProducer'),
  container.resolve('TopicPrefix')
));

container.register('NotificationService', () => new SendgridNotificationService(
  container.resolve('SendgridClient')
));

// Register use case handlers
container.register('CreateOrderHandler', (c) => new CreateOrderHandler(
  c.resolve('OrderRepository'),
  c.resolve('InventoryService'),
  c.resolve('PaymentGateway'),
  c.resolve('EventPublisher'),
  c.resolve('OrderFactory')
));

container.register('ProcessPaymentHandler', (c) => new ProcessPaymentHandler(
  c.resolve('OrderRepository'),
  c.resolve('PaymentGateway')
));

container.register('ShipOrderHandler', (c) => new ShipOrderHandler(
  c.resolve('OrderRepository'),
  c.resolve('InventoryService'),
  c.resolve('NotificationService')
));

// Register inbound adapters (controllers)
container.register('CreateOrderController', (c) =>
  new HTTPOrderController(c.resolve('CreateOrderHandler'))
);

container.register('ProcessPaymentController', (c) =>
  new HTTPPaymentController(c.resolve('ProcessPaymentHandler'))
);

// Register domain factories
container.register('OrderFactory', () => new OrderFactory(
  container.resolve('ProductRepository')
));

export { container };
```

---

## CQRS Pattern in Layered Architecture

Command Query Responsibility Segregation (CQRS) separates operations that change state (Commands) from operations that read data (Queries). This pattern fits naturally with layered architecture.

```typescript
// application/ports/inbound/command.port.ts
export interface Command<Input, Output> {
  execute(input: Input): Promise<Output>;
}

// application/ports/inbound/query.port.ts
export interface Query<Input, Output> {
  execute(input: Input): Promise<Output>;
}

// application/use-cases/commands/create-order.command.ts
export interface CreateOrderCommandInput {
  customerId: string;
  items: Array<{ productId: string; quantity: number }>;
  shippingAddress: Address;
}

export interface CreateOrderCommandOutput {
  orderId: string;
  total: number;
}

export interface CreateOrderCommand extends Command<CreateOrderCommandInput, CreateOrderCommandOutput> {
}

// application/use-cases/queries/get-order.query.ts
export interface GetOrderQueryInput {
  orderId: string;
  includeItems?: boolean;
}

export interface GetOrderQueryOutput {
  order: OrderDto | null;
  error?: string;
}

export interface GetOrderQuery extends Query<GetOrderQueryInput, GetOrderQueryOutput> {
}

// Different handlers for different operations
export class CreateOrderCommandHandler implements CreateOrderCommand {
  // Modifies state - returns new order ID and total
}

export class GetOrderQueryHandler implements GetOrderQuery {
  // Only reads - returns order data, never modifies state
}
```

---

## Data Transfer Objects (DTOs) and Mappers

DTOs transfer data between layers without behavior. Mappers translate between different representations in different layers.

```typescript
// application/dtos/order.dto.ts
export class OrderDto {
  id: string;
  customerId: string;
  status: string;
  total: number;
  currency: string;
  items: OrderLineItemDto[];
  shippingAddress: AddressDto;
  createdAt: Date;
  updatedAt: Date;
}

export class OrderLineItemDto {
  productId: string;
  productName: string;
  quantity: number;
  unitPrice: number;
  subtotal: number;
}

export class AddressDto {
  street: string;
  city: string;
  state: string;
  postalCode: string;
  country: string;
}

// infrastructure/mappers/order.mapper.ts
export class OrderMapper {
  // Domain → DTO
  static toDto(order: Order): OrderDto {
    return {
      id: order.id.value,
      customerId: order.customerId,
      status: order.status.value,
      total: order.total.amountValue,
      currency: order.total.currencyCode,
      items: order.lineItems.map(item => ({
        productId: item.productId,
        productName: item.productName,
        quantity: item.quantity,
        unitPrice: item.unitPrice.amountValue,
        subtotal: item.subtotal.amountValue
      })),
      shippingAddress: {
        street: order.shippingAddress.street,
        city: order.shippingAddress.city,
        state: order.shippingAddress.state,
        postalCode: order.shippingAddress.postalCode,
        country: order.shippingAddress.country
      },
      createdAt: order.createdAt,
      updatedAt: order.updatedAt
    };
  }

  // Domain → Persistence Object
  static toPersistence(order: Order): OrderPo {
    return {
      id: order.id.value,
      customer_id: order.customerId,
      status: order.status.value,
      total_amount: order.total.amountValue,
      currency: order.total.currencyCode,
      shipping_address: JSON.stringify(order.shippingAddress),
      created_at: order.createdAt,
      updated_at: new Date()
    };
  }

  // Persistence Object → Domain
  static toDomain(po: OrderPo): Order {
    return Order.reconstitute({
      id: OrderId.of(po.id),
      customerId: po.customer_id,
      lineItems: [], // Would need to load from separate table
      status: OrderStatus.of(po.status),
      shippingAddress: JSON.parse(po.shipping_address),
      createdAt: po.created_at
    });
  }

  // DTO → Input (for use cases)
  static toCreateOrderInput(dto: OrderDto): CreateOrderInput {
    return {
      customerId: dto.customerId,
      items: dto.items.map(item => ({
        productId: item.productId,
        quantity: item.quantity
      })),
      shippingAddress: Address.of(dto.shippingAddress)
    };
  }
}
```

---

## Read-Only Interface Pattern

Prevent unintended modifications by exposing read-only views of domain objects.

```typescript
// domain/interfaces/read-only-order.interface.ts
export interface ReadOnlyOrder {
  getId(): OrderId;
  getCustomerId(): string;
  getStatus(): OrderStatus;
  getTotal(): Money;
  getLineItems(): ReadonlyArray<ReadOnlyOrderLineItem>;
  getCreatedAt(): Date;
}

// infrastructure/adapters/presenters/read-only-order.adapter.ts
export class ReadOnlyOrderAdapter implements ReadOnlyOrder {
  constructor(private order: Order) {}

  getId(): OrderId {
    return this.order.id;
  }

  getCustomerId(): string {
    return this.order.customerId;
  }

  getStatus(): OrderStatus {
    return this.order.status;
  }

  getTotal(): Money {
    return this.order.total;
  }

  getLineItems(): ReadonlyArray<ReadOnlyOrderLineItem> {
    return this.order.lineItems.map(item => new ReadOnlyOrderLineItemAdapter(item));
  }

  getCreatedAt(): Date {
    return this.order.createdAt;
  }
}

export class ReadOnlyOrderLineItemAdapter implements ReadOnlyOrderLineItem {
  constructor(private item: OrderLineItem) {}

  getProductId(): string {
    return this.item.productId;
  }

  getQuantity(): number {
    return this.item.quantity;
  }

  getUnitPrice(): Money {
    return this.item.unitPrice;
  }

  getSubtotal(): Money {
    return this.item.subtotal;
  }
}
```

---

## Testing Strategy

Because of the clear separation, testing can be done at multiple levels, each with different characteristics and purposes.

```typescript
// 1. Domain Layer - Pure unit tests, no dependencies
describe('Order Entity', () => {
  it('should create order with valid data', () => {
    const orderId = OrderId.of('order-123');
    const customerId = 'customer-456';
    const lineItems = [
      OrderLineItem.create({
        productId: 'product-1',
        quantity: 2,
        unitPrice: Money.fromAmount(25.00)
      })
    ];
    const status = OrderStatus.DRAFT;

    const order = Order.create({
      id: orderId,
      customerId,
      lineItems,
      status,
      shippingAddress: TestAddresses.VALID_ADDRESS,
      paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
    });

    expect(order.id).toEqual(orderId);
    expect(order.customerId).toBe(customerId);
    expect(order.total.amountValue).toBe(50.00);
  });

  it('should reject empty line items', () => {
    expect(() => {
      Order.create({
        id: OrderId.of('order-123'),
        customerId: 'customer-456',
        lineItems: [],
        status: OrderStatus.DRAFT,
        shippingAddress: TestAddresses.VALID_ADDRESS,
        paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
      });
    }).toThrow(DomainError);
  });

  it('should calculate total correctly', () => {
    const order = Order.create({
      id: OrderId.of('order-123'),
      customerId: 'customer-456',
      lineItems: [
        OrderLineItem.create({
          productId: 'product-1',
          quantity: 2,
          unitPrice: Money.fromAmount(25.00)
        }),
        OrderLineItem.create({
          productId: 'product-2',
          quantity: 1,
          unitPrice: Money.fromAmount(50.00)
        })
      ],
      status: OrderStatus.DRAFT,
      shippingAddress: TestAddresses.VALID_ADDRESS,
      paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
    });

    expect(order.total.amountValue).toBe(100.00);
  });

  it('should transition to SUBMITTED when submitted', () => {
    const order = Order.create(TestOrderData.VALID_DRAFT_ORDER);

    order.submit();

    expect(order.status).toBe(OrderStatus.SUBMITTED);
  });
});

// 2. Use Case Layer - Tests with in-memory repositories
describe('CreateOrderHandler', () => {
  let handler: CreateOrderHandler;
  let orderRepository: InMemoryOrderRepository;
  let inventoryService: MockInventoryService;
  let paymentGateway: MockPaymentGateway;
  let eventPublisher: MockEventPublisher;
  let orderFactory: OrderFactory;

  beforeEach(() => {
    orderRepository = new InMemoryOrderRepository();
    inventoryService = new MockInventoryService();
    paymentGateway = new MockPaymentGateway();
    eventPublisher = new MockEventPublisher();
    orderFactory = new OrderFactory();

    handler = new CreateOrderHandler(
      orderRepository,
      inventoryService,
      paymentGateway,
      eventPublisher,
      orderFactory
    );
  });

  it('should create order successfully', async () => {
    const input: CreateOrderInput = {
      customerId: 'customer-123',
      items: [
        { productId: 'product-1', quantity: 2 }
      ],
      shippingAddress: TestAddresses.VALID_ADDRESS,
      paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
    };

    const result = await handler.execute(input);

    expect(result.isSuccess).toBe(true);
    expect(result.orderId).toBeDefined();
    expect(result.total).toBe(50.00);
    expect(orderRepository.orders.size).toBe(1);
  });

  it('should fail when inventory is unavailable', async () => {
    inventoryService.setAvailabilityCheckError(
      new DomainError('Product is out of stock')
    );

    const input: CreateOrderInput = {
      customerId: 'customer-123',
      items: [
        { productId: 'out-of-stock-product', quantity: 1 }
      ],
      shippingAddress: TestAddresses.VALID_ADDRESS,
      paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
    };

    const result = await handler.execute(input);

    expect(result.isSuccess).toBe(false);
    expect(result.error).toBe('Product is out of stock');
    expect(orderRepository.orders.size).toBe(0);
  });
});

// 3. Integration Tests - Full stack with test adapters
describe('Order API Integration', () => {
  it('should create order via HTTP API', async () => {
    const app = createTestApp();
    const response = await request(app)
      .post('/api/orders')
      .send({
        customerId: 'customer-123',
        items: [{ productId: 'product-1', quantity: 2 }],
        shippingAddress: TestAddresses.VALID_ADDRESS,
        paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
      })
      .expect(201);

    expect(response.body.success).toBe(true);
    expect(response.body.data.orderId).toBeDefined();
  });
});

// 4. Contract Tests - Adapter contract verification
describe('OrderRepository Contract', () => {
  let repository: OrderRepository;

  beforeEach(() => {
    repository = new InMemoryOrderRepository();
  });

  it('should save and find order by id', async () => {
    const order = Order.create(TestOrderData.VALID_DRAFT_ORDER);

    await repository.save(order);
    const found = await repository.findById(order.id);

    expect(found).toEqual(order);
  });

  it('should return null for non-existent order', async () => {
    const found = await repository.findById(OrderId.of('non-existent'));

    expect(found).toBeNull();
  });
});
```

---

## The Three Fundamental Principles

Layered Architecture is governed by three fundamental principles that guide the organization of code and dependencies. These principles, when properly applied, create a clean separation between business logic and external concerns.

### 1. Layering Principle (分層原則)

The architecture is divided into concentric layers, each with a specific responsibility and level of abstraction. The most important principle of layering is that each layer has a clear purpose and that objects within each layer know only about their own layer and the layers inside it.

**The Four Layers:**

- **Entities Layer (Innermost)**: Contains domain model objects including Aggregates, Entities, Value Objects, and Domain Services following Domain-Driven Design (DDD) patterns. This layer represents the core business logic and should be completely independent of other layers.

- **Use Cases Layer**: Contains application-specific business logic, corresponding to DDD's Application Services. This layer orchestrates domain objects to perform specific business operations and defines input/output port interfaces.

- **Interface Adapters Layer**: Contains adapters that translate between external systems and the application core. This includes controllers, gateways, presenters, and database adapters that implement the ports defined in the inner layers.

- **Frameworks & Drivers Layer (Outermost)**: Contains external frameworks, tools, and delivery mechanisms such as Web frameworks, database systems, external services, and UI frameworks.

**Placement Considerations:**

While the four layers are clearly separated in principle, some objects fall into gray areas during implementation. For example, the placement of DDD Repositories and Mappers can be debated. Some practitioners place them in the Use Cases layer, while others place them closer to the Entities layer. The key is to maintain consistency within the codebase and ensure that the dependency rules are respected.

```
Layer Placement Decision Guide:

┌─────────────────────────────────────────────────────────────┐
│                  FRAMEWORKS & DRIVERS                       │
│    Web frameworks, Database drivers, External libraries     │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                  INTERFACE ADAPTERS                          │
│    Controllers, Presenters, Gateway implementations,        │
│    Mappers, Serializers                                     │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                      USE CASES                              │
│    Application services, Use case handlers,                 │
│    Port interfaces (Input/Output), DTOs                     │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                       ENTITIES                               │
│    Aggregates, Entities, Value Objects, Domain Services,    │
│    Domain Events, Domain Factories                          │
└─────────────────────────────────────────────────────────────┘
```

### 2. Dependency Principle (相依性原則)

Source code dependencies must always point **inward** from outer layers to inner layers. This means that code in outer layers may depend on abstractions defined in inner layers, but inner layers must never depend on outer layers. This principle is the cornerstone of clean architecture and is achieved through the **Dependency Inversion Principle (DIP)**.

**Dependency Rules:**

- Objects in the Entities layer can only reference other objects within the Entities layer
- Objects in the Use Cases layer can only reference the Entities layer and other objects within the Use Cases layer
- Objects in the Interface Adapters layer can reference both Use Cases and Entities layers (through defined ports)
- The Frameworks & Drivers layer can reference all inner layers through their public interfaces

**Achieving Inward Dependencies with Dependency Inversion:**

When the Use Cases layer needs to persist Aggregates to a database, it must reference database objects. However, this would violate the dependency rule since the database is in an outer layer. The solution is to define a port (interface) in the Use Cases layer and have the outer database layer implement it. This way, the dependency direction is reversed: the outer layer depends on the inner layer's abstraction.

```typescript
// Use Cases layer defines the abstraction (port)
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: OrderId): Promise<Order | null>;
}

// Interface Adapters layer implements the port
class SqlOrderRepository implements OrderRepository {
  constructor(private db: DatabaseConnection) {}

  async save(order: Order): Promise<void> {
    // Implementation using SQL database
    await this.db.query('INSERT INTO orders ...');
  }

  async findById(id: OrderId): Promise<Order | null> {
    // Implementation using SQL database
    const result = await this.db.query('SELECT * FROM orders WHERE id = ?', [id.value]);
    return result ? OrderMapper.toDomain(result) : null;
  }
}

// Dependency direction:
// UseCases ──depends on──> OrderRepository (interface) ──implemented by──> SqlOrderRepository
```

**Practical Challenges:**

Completely adhering to the dependency principle can be challenging in practice. For example, when domain entities need to publish Domain Events using an event bus tool, using an external library directly would violate the dependency rule. The solution is to define an interface for the event bus and have the external library implement it through an adapter. While this adds complexity, it maintains the architectural integrity of the system.

### 3. Cross-Layer Principle (跨層原則)

When domain objects need to cross layer boundaries to be passed outside the Use Cases layer (to UI, database, or other external systems), they must not be passed directly. Instead, interfaces and data structures should be defined in the Use Cases layer, and domain objects should be converted to DTOs (Data Transfer Objects) before crossing layers.

**Rationale:**

Directly passing domain objects outside the Use Cases layer can lead to the domain layer being affected by presentation logic or persistence requirements. For example, if a UI component needs specific data from an Order entity, the entity might be modified to accommodate UI needs, thereby contaminating the domain logic with presentation concerns. To maintain separation of concerns and single responsibility, objects crossing layers must be transformed.

```typescript
// WRONG - Domain object passed directly to UI
class OrderController {
  async getOrder(req: Request, res: Response): Promise<void> {
    const order = await this.getOrderUseCase.execute(req.params.orderId);
    res.json(order); // Direct domain object exposure
  }
}

// RIGHT - Domain object converted to DTO
class OrderController {
  async getOrder(req: Request, res: Response): Promise<void> {
    const result = await this.getOrderUseCase.execute(req.params.orderId);
    const dto = OrderMapper.toDto(result.order); // Convert to DTO
    res.json(dto);
  }
}

// DTO defined in Use Cases layer
interface OrderDto {
  id: string;
  status: string;
  total: number;
  items: OrderItemDto[];
}
```

**Benefits of the Cross-Layer Principle:**

- **Separation of Concerns**: Domain objects remain focused on business logic, not presentation or persistence
- **Flexibility**: UI and persistence can change without affecting domain objects
- **Testability**: Domain objects can be tested in isolation without UI or database dependencies
- **Encapsulation**: Internal domain structure is hidden from external layers

### Practical Considerations: The "Cleanliness" Trade-off

While strict adherence to all three principles results in a clean and maintainable architecture, practical implementation often requires balancing architectural purity with development efficiency. Consider the following guidelines:

**Strict Adherence Recommended:**

- The Dependency Principle should generally be followed, especially between the Entities and Use Cases layers. When external tools are needed, define abstractions and use adapters rather than direct dependencies.

**Acceptable Relaxation:**

- The Interface Adapters layer often couples tightly with specific frameworks (e.g., Web Controllers with Web framework classes). Attempting complete dependency inversion in this layer can be overly complex and may not provide sufficient benefit.

**Practical Simplification:**

- Separating Use Cases into Commands (state-modifying operations) and Queries (read operations) using the CQRS pattern can significantly simplify the design of Interface Adapters. Commands can share common presenters and view models, while only Queries need customized output handling. This approach keeps the adapter layer lean and focused.

---

## Benefits of Layered Architecture

| Benefit | Description |
|---------|-------------|
| **Testability** | Business rules can be tested without UI, database, or external systems using pure unit tests |
| **Maintainability** | Changes to outer layers (frameworks, databases) do not affect inner layers (domain, use cases) |
| **Flexibility** | External systems and delivery mechanisms can be swapped without changing business code |
| **Scalability** | Clear separation enables parallel development by multiple teams |
| **Framework Independence** | Business logic survives framework changes and technology migrations |
| **Adaptability** | Application can be delivered through multiple channels (HTTP, gRPC, CLI, messaging) |
| **Longevity** | Core business logic outlives specific technology choices |
| **Test doubles** | Easy to create mock implementations of ports for testing |
| **Technology Deferral** | Technology choices can be deferred until the outer layers |
| **Domain Focus** | Business logic remains pure and focused on domain rules |

---

## Anti-Patterns to Avoid

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| **Anemic Domain** | Entities with no behavior, logic in services | Move business logic from services to domain entities |
| **Dependency Inversion Violation** | Domain depending on frameworks or infrastructure | Use dependency injection, define ports in domain/application |
| **Feature Scattered Across Layers** | Related code in multiple layers | Organize by feature module, not by layer |
| **God Interface** | One port/interface for everything | Separate inbound/outbound ports, single responsibility |
| **Skipping Layers** | Controllers directly calling repositories | Always go through use cases for business operations |
| **Leaking Infrastructure** | Domain entities with framework annotations | Keep domain pure, use adapters for persistence mapping |
| **Concrete Dependencies** | Use cases depending on concrete implementations | Depend on abstractions (interfaces), inject implementations |
| **Missing Ports** | Direct imports of infrastructure in core | Define ports, implement in adapters |

### Common Violations and Fixes

```typescript
// WRONG - Domain depends on JPA/ORM annotations
@Entity
class Order {
  @Id
  private id: string;
  
  @Column
  private status: string;
}

// RIGHT - Pure domain entity
export class Order {
  private constructor(
    private readonly id: OrderId,
    private readonly status: OrderStatus
  ) {}
}

// WRONG - Controller directly calls repository
class OrderController {
  async handle() {
    // Bypasses use case layer
    await this.entityManager.save(order);
  }
}

// RIGHT - Controller calls use case
class OrderController {
  async handle(request: Request): Promise<Response> {
    const result = await this.createOrderHandler.execute(request.body);
    return Response.of(result);
  }
}

// WRONG - Use case depends on concrete implementation
class CreateOrderHandler {
  constructor(
    private repository: TypeORMOrderRepository // Should be interface
  ) {}
}

// RIGHT - Use case depends on abstraction
class CreateOrderHandler {
  constructor(
    private repository: OrderRepository // Interface
  ) {}
}
```

---

## Layered Architecture in AI-Assisted Development

```markdown
## AI Prompt for Layered Architecture Implementation

Generate a feature following Layered Architecture (Ports and Adapters):

Context: E-commerce Order Management System

Requirements:
1. Create order from shopping cart items
2. Validate inventory availability
3. Process payment
4. Send order confirmation notification
5. Publish order created event

Generate:
1. Domain entities (Order, OrderLineItem)
2. Value objects (Money, Address, OrderId)
3. Domain events (OrderCreated, PaymentProcessed)
4. Use case handler (CreateOrderHandler)
5. Input/output ports (interfaces)
6. Repository interface for orders
7. HTTP controller adapter
8. TypeORM repository adapter
9. Unit tests for domain rules
10. Integration tests for use cases

Constraints:
- All dependencies point inward to domain
- Domain has no imports from outer layers
- Use TypeScript
- Include JSDoc documentation
- Follow dependency inversion principle
- Support multiple payment gateways (strategy pattern)
```

---

## Detailed Architecture Principles (From Clean Architecture Literature)

### Software Architecture: Definition and Purpose

Understanding the foundational concepts of software architecture is essential before diving into implementation details. Software architecture is fundamentally the **shape** given to a system by those who build it. This shape is determined by three fundamental elements:

1. **Components**: The building blocks that constitute the system
2. **Arrangement**: How components are organized and structured
3. **Communication**: How components interact with each other and with external systems

The purpose of architecture is to serve four key objectives:

- **Development**: Enabling efficient and maintainable software development
- **Deployment**: Supporting smooth and reliable system deployment
- **Operation**: Facilitating system operation and monitoring
- **Maintenance**: Enabling easy modification and enhancement over time

**The ultimate goals of good architecture are:**

- **Minimize the total cost of the software lifecycle**
- **Maximize programmer productivity**

A well-designed architecture reflects the system's functionality clearly. When developers examine a properly structured codebase, they should be able to understand what the system does simply by looking at its organization. For example, a web shopping system should clearly show components for displaying product lists, shopping cart management, checkout processing, payment handling, shipping selection, and order tracking.

---

### Port and Adapter Architecture: Different Perspectives

Port and Adapter Architecture, also known as Hexagonal Architecture, can be visualized from different perspectives, leading to different but equivalent diagrams.

**From the Client Perspective:**

```
Client → Port → Adapter → Use Case
```

The client first interacts with a Port (interface), which is then handled by an Adapter that converts the request and calls the Use Case. This perspective emphasizes how external systems access the application.

**From the Use Case Perspective:**

```
Use Case → Output Port → Adapter → External System
```

The Use Case communicates through Output Ports to external systems. The Adapter implements these ports and handles the translation between internal and external formats.

**Port vs. Adapter: What is What?**

A common point of confusion is distinguishing between Ports and Adapters:

- **Ports** are the interfaces or contracts that define what the application can do or what it needs. Examples include:
  - JDBC drivers (enable connection to databases)
  - REST API entry points (enable connection to services)
  - Repository interfaces (define data access contracts)

- **Adapters** are implementations that convert between external formats and internal interfaces. Examples include:
  - Controllers that convert HTTP requests to use case inputs
  - Repositories that convert database results to domain objects
  - Presenters that format use case outputs for display

**Key Insight:** Software architecture can be continuously chained with multiple Port and Adapter sets. Just as a laptop's Thunderbolt port can connect through multiple adapters (HDMI to VGA, USB to Ethernet), an application's ports can connect to various external systems through appropriate adapters.

---

### Reference Examples: Learning Points

When studying Clean Architecture implementations, consider the following evaluation criteria:

**1. Project Structure:**
- Is it organized as a single project or multiple projects?
- If multiple projects, what are they and why were they separated?
- What are the dependencies between projects?

**2. Entity Design:**
- What entities exist in the domain layer?
- Are they rich with behavior or anemic (pure data)?
- Do entities contain any framework dependencies?

**3. Use Case Implementation:**
- How are bidirectional interfaces implemented (Input/Output ports)?
- How are input and output data structures defined?
- Does the use case follow the Command pattern?

**4. Cross-Layer Object Handling:**
- How are objects transformed when crossing layer boundaries?
- Are DTOs or View Models used?
- How are mappers implemented?

**5. Dependency Compliance:**
- Does the architecture follow the dependency rule?
- Are there any cross-layer references from outer to inner layers?
- How are framework dependencies isolated?

**6. Repository Pattern:**
- Where are repositories defined (which layer)?
- Are repository interfaces in the domain/application layer?
- How is persistence technology isolated?

**7. Presenter and View Model:**
- How are results presented to the user interface?
- Are View Models used to separate presentation concerns?
- How does the use case communicate results?

---

### Layering Principle: Distance from I/O

The layering principle is defined by a key concept: **the further a component layer is from I/O, the higher its level; the closer to I/O, the lower its level**.

This concept aligns with organizational hierarchies. In a company, receptionists handling phone calls and visitor reception are close to I/O and have lower organizational levels. Executives who are far from daily operational I/O have higher levels. Similarly, in software:

- **Entities (Innermost, Highest Level)**: Core business logic, farthest from I/O, most abstract
- **Use Cases (High Level)**: Application-specific business logic, defines what the system does
- **Interface Adapters (Low Level)**: Translates between external protocols and internal abstractions
- **Frameworks & Drivers (Outermost, Lowest Level)**: Directly handles I/O, uses external tools and frameworks

**The Four Layers in Detail:**

**Entities Layer:**
- Contains domain model objects (Aggregates, Entities, Value Objects)
- Domain Services for complex business logic
- Domain Events for modeling state changes
- Domain Factories for creating complex objects
- **Zero dependencies** on any other layer
- Represents the core business rules that should be reusable across applications

**Use Cases Layer:**
- Contains application services (Use Case handlers)
- Defines Input Port interfaces (what external systems can do)
- Defines Output Port interfaces (what the application needs)
- Contains Input/Output DTOs for data transfer
- Orchestrates domain objects to perform business operations
- Does not contain business rules, only application logic

**Interface Adapters Layer:**
- Controllers (HTTP, gRPC, CLI) that receive external requests
- Presenters that format output for presentation
- Gateway implementations (Repository implementations)
- Mappers that transform between layers
- Translates external protocols to internal abstractions

**Frameworks & Drivers Layer:**
- Web frameworks (Express, Spring, Fastify)
- Database drivers (JDBC, TypeORM, Prisma)
- UI frameworks (React, Angular)
- External service SDKs (Stripe, AWS)
- Message queue clients (Kafka, RabbitMQ)
- Contains "glue code" connecting the application to external tools

---

### Dependency Principle: Implementation Challenges

The dependency principle states: **"Source code dependencies must point only inward, toward higher-level policies."** While conceptually simple, practical implementation presents challenges.

**Challenge 1: Framework Annotations in Domain Entities**

A common violation is placing JPA/Hibernate annotations directly onjava
@Entity
 domain entities:

```@Table(name = "hosts")
public class Host {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Embedded
    private Address address;
}
```

This creates a dependency between the Entities layer and the persistence framework. While convenient, this couples the domain model to a specific technology.

**Solutions:**

1. **Accept Practical Relaxation**: Accept the coupling for simpler code, understanding the trade-off
2. **Create Separate Persistence Objects**: Define POJOs for persistence and use mappers to convert between domain entities and persistence objects
3. **Use XML Mappings**: Move metadata to configuration files (though this just changes implicit to explicit dependency)

**Challenge 2: External Library Dependencies**

When domain logic needs external functionality (e.g., an event bus for domain events), using external libraries directly violates the dependency principle.

**Solution:** Define an abstraction (interface) in the domain/application layer. Have the external library implement this interface through an adapter. This maintains architectural integrity at the cost of additional code.

**Challenge 3: Database Access in Use Cases**

Use Cases need to persist aggregates but cannot depend on the database directly. The solution is dependency inversion:

```typescript
// Domain/Application layer defines the abstraction
interface HostRepository {
    save(host: Host): Promise<void>;
    findById(id: HostId): Promise<Host | null>;
    findAll(): Promise<Host[]>;
    delete(id: HostId): Promise<void>;
}

// Infrastructure layer implements the abstraction
class JPAHostRepository implements HostRepository {
    constructor(private entityManager: EntityManager) {}
    
    async save(host: Host): Promise<void> {
        const entity = HostMapper.toPersistence(host);
        await this.entityManager.save(entity);
    }
    
    async findById(id: HostId): Promise<Host | null> {
        const entity = await this.entityManager.findOne(HostEntity, {
            where: { id: id.value }
        });
        return entity ? HostMapper.toDomain(entity) : null;
    }
    
    async findAll(): Promise<Host[]> {
        const entities = await this.entityManager.find(HostEntity, {});
        return entities.map(HostMapper.toDomain);
    }
    
    async delete(id: HostId): Promise<void> {
        await this.entityManager.delete(HostEntity, { where: { id: id.value } });
    }
}
```

**Benefits of Strictly Following the Dependency Principle:**

1. **Enhanced Testability**: Core business logic can be tested without UI, database, or external systems
2. **Framework Independence**: Business logic survives framework changes
3. **Technology Flexibility**: Easy to swap databases, web frameworks, or external services
4. **Maintainability**: Changes to outer layers do not affect inner layers
5. **Parallel Development**: Teams can work on different layers independently

---

### Cross-Layer Principle: Bidirectional Interfaces

When objects need to cross layer boundaries, the cross-layer principle governs how this should be done to maintain separation of concerns.

**The Problem with Direct Passing:**

If a Use Case directly passes domain entities to the presentation layer, several issues arise:

1. Presentation requirements might influence domain entity design
2. Domain entities might be modified to accommodate UI needs
3. Serialization issues (circular references, lazy loading proxies)
4. Coupling between domain logic and presentation concerns

**The Solution: Bidirectional Interfaces**

Clean Architecture recommends defining bidirectional interfaces for Use Cases:

1. **Input Interface**: Defines the data the Use Case accepts (Input Port)
2. **Output Interface**: Defines how the Use Case returns results (Output Port, implemented by Presenter)

**Implementation Pattern:**

```typescript
// Use Case Input Interface
interface FetchHostsInput {
    getSearchCriteria(): HostSearchCriteria;
}

// Use Case Output Interface (Presenter)
interface FetchHostsOutput {
    presentHosts(hosts: Host[]): void;
    presentError(error: string): void;
}

// Concrete Use Case Input Data
class FetchHostsInputData implements FetchHostsInput {
    private criteria: HostSearchCriteria;
    
    constructor(criteria: HostSearchCriteria) {
        this.criteria = criteria;
    }
    
    getSearchCriteria(): HostSearchCriteria {
        return this.criteria;
    }
}

// Concrete Presenter
class HostListPresenter implements FetchHostsOutput {
    private viewModel: HostListViewModel;
    
    presentHosts(hosts: Host[]): void {
        this.viewModel = {
            hosts: hosts.map(host => ({
                id: host.getId().value,
                name: host.getName().value,
                status: host.getStatus().value
            })),
            isEmpty: hosts.length === 0
        };
    }
    
    presentError(error: string): void {
        this.viewModel = {
            error,
            hosts: [],
            isEmpty: true
        };
    }
    
    getViewModel(): HostListViewModel {
        return this.viewModel;
    }
}

// Use Case Implementation
class FetchHostsUseCase implements FetchHostsInput {
    constructor(
        private hostRepository: HostRepository,
        private outputPort: FetchHostsOutput
    ) {}
    
    execute(input: FetchHostsInput): void {
        try {
            const criteria = input.getSearchCriteria();
            const hosts = this.hostRepository.findByCriteria(criteria);
            this.outputPort.presentHosts(hosts);
        } catch (error) {
            this.outputPort.presentError(error.message);
        }
    }
}

// Controller (Inbound Adapter)
class HostController {
    constructor(private fetchHostsUseCase: FetchHostsUseCase) {}
    
    getHosts(request: HttpRequest): HttpResponse {
        const criteria = HostSearchCriteria.fromRequest(request);
        const input = new FetchHostsInputData(criteria);
        const presenter = new HostListPresenter();
        
        this.fetchHostsUseCase.execute(input);
        
        return HttpResponse.ok(presenter.getViewModel());
    }
}
```

**Benefits of the Cross-Layer Principle:**

1. **Separation of Concerns**: Domain objects remain focused on business logic
2. **Flexibility**: UI and persistence can change without affecting domain objects
3. **Testability**: Each layer can be tested independently
4. **Encapsulation**: Internal domain structure is hidden from external layers
5. **Adaptability**: Easy to add new presentation formats or data sources

---

### The Cleanliness Trade-off

While strict adherence to all three principles results in a clean and maintainable architecture, practical implementation often requires balancing architectural purity with development efficiency.

**When to Strictly Follow:**

- The Dependency Principle should generally be followed, especially between Entities and Use Cases layers
- When external tools are needed, define abstractions and use adapters
- Critical business logic that needs to be stable and testable

**When to Accept Relaxation:**

- The Interface Adapters layer often couples tightly with specific frameworks
- Web Controllers with Web framework classes
- Attempting complete dependency inversion in this layer can be overly complex

**Practical Simplifications:**

- Separating Use Cases into Commands (state-modifying operations) and Queries (read operations) using the CQRS pattern simplifies adapter design
- Commands can share common presenters and view models
- Only Queries need customized output handling
- This approach keeps the adapter layer lean and focused

**Key Insight:** There is no absolute cleanliness—every clean room still has dust. The goal is to achieve the right balance for your specific context and requirements.

---

## Comparison with Related Patterns

### Layered Architecture vs. Clean Architecture

Clean Architecture, introduced by Robert C. Martin, is essentially the same concept as Hexagonal Architecture with a slightly different layer naming convention. Clean Architecture typically includes four layers: Frameworks & Drivers, Interface Adapters, Application Business Rules, and Enterprise Business Rules. The core principles and dependency rules are identical.

### Layered Architecture vs. Onion Architecture

Onion Architecture, proposed by Jeffrey Palermo, uses similar concentric layer concepts but emphasizes the "onion" metaphor with dependencies pointing toward the center. The layers are often named: Application Core (inner), Domain Services, Application Services (outer), and Infrastructure. The fundamental principles are the same as Layered Architecture.

### Layered Architecture vs. Microservices

While both promote separation of concerns, Layered Architecture is an application-level pattern for organizing code within a single service, whereas Microservices is an architectural style for decomposing an entire system into independently deployable services. Layered Architecture principles can be applied within each microservice.

---

## References and Further Reading

1. Cockburn, Alistair. "Hexagonal Architecture." 2005.
2. Martin, Robert C. "Clean Architecture: A Craftsman's Guide to Software Structure and Design." Prentice Hall, 2017.
3. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
4. Vernon, Vaughn. "Implementing Domain-Driven Design." Addison-Wesley, 2013.
5. Martin, Robert C. "The Dependency Inversion Principle." 1996.
6. Percival, Harry, and Bob Gregory. "Architecture Patterns with Python." O'Reilly, 2021.
7. Ford, Neal, et al. "Architecture: The Hard Parts." O'Reilly, 2021.
8. Zimarev, Alexey. "Hands-On Domain-Driven Design with .NET Core." Packt, 2019.
9. Chen, Teddy. "Clean Architecture（1）：軟體架構的定義與目的" (Clean Architecture (1): Software Architecture Definition and Purpose). teddy-chen-tw.blogspot.com, 2018.
10. Chen, Teddy. "Clean Architecture（2）：Port and Adapter Architecture" (Clean Architecture (2): Port and Adapter Architecture). teddy-chen-tw.blogspot.com, 2018.
11. Chen, Teddy. "Clean Architecture（3）：精選參考範例" (Clean Architecture (3): Selected Reference Examples). teddy-chen-tw.blogspot.com, 2018.
12. Chen, Teddy. "Clean Architecture（4）：架構三原則首部曲—分層原則" (Clean Architecture (4): Architecture Three Principles Part 1 - Layering Principle). teddy-chen-tw.blogspot.com, 2018.
13. Chen, Teddy. "Clean Architecture（5）：架構三原則二部曲—相依性原則" (Clean Architecture (5): Architecture Three Principles Part 2 - Dependency Principle). teddy-chen-tw.blogspot.com, 2018.
14. Chen, Teddy. "Clean Architecture（6）：架構三原則三部曲—跨層原則" (Clean Architecture (6): Architecture Three Principles Part 3 - Cross-Layer Principle). teddy-chen-tw.blogspot.com, 2018.
15. Chen, Teddy. "再談Clean Architecture三原則" (Revisiting Clean Architecture Three Principles). teddy-chen-tw.blogspot.com, 2020.
16. Cockburn, Alistair. "Hexagonal Architecture." http://alistair.cockburn.us/Hexagonal+architecture
17. 8th Light. "The Clean Architecture." https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
