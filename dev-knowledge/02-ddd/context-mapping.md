# Context Mapping

Context Mapping defines the relationships between bounded contexts, establishing how different parts of a system interact and share information.

## Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CONTEXT MAP                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌──────────────┐          ┌──────────────┐          ┌──────────────┐   │
│   │   SALES      │          │  INVENTORY   │          │   SHIPPING   │   │
│   │  BOUNDED     │◄────────►│  BOUNDED     │◄────────►│  BOUNDED     │   │
│   │   CONTEXT    │ Customer │   CONTEXT    │ Supplier │   CONTEXT    │   │
│   │              │  -Supplier│              │          │              │   │
│   └──────────────┘          └──────────────┘          └──────────────┘   │
│          │                        │                        │               │
│          │        ┌───────────────┴───────────────┐        │               │
│          │        │                               │        │               │
│          ▼        ▼                               ▼        ▼               │
│   ┌──────────────────────────────────────────────────────────────┐        │
│   │                    SHARED KERNEL                              │        │
│   │         (Common types: Money, Address, DateRange)            │        │
│   └──────────────────────────────────────────────────────────────┘        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Context Mapping Patterns

### Shared Kernel

Sharing common domain code between contexts to reduce duplication and maintain consistency.

```typescript
// Shared kernel types used across multiple bounded contexts
namespace SharedKernel {
  export class Money {
    constructor(
      private readonly amount: number,
      private readonly currency: Currency
    ) {}

    add(other: Money): Money {
      if (this.currency !== other.currency) {
        throw new CurrencyMismatchError();
      }
      return new Money(this.amount + other.amount, this.currency);
    }

    static zero(currency: Currency): Money {
      return new Money(0, currency);
    }
  }

  export class Address {
    constructor(
      readonly street: string,
      readonly city: string,
      readonly zipCode: string,
      readonly country: string
    ) {
      if (!street || !city || !zipCode || !country) {
        throw new InvalidAddressError('All address fields are required');
      }
    }
  }

  export class DateRange {
    constructor(
      readonly start: DateTime,
      readonly end: DateTime
    ) {
      if (end.isBefore(start)) {
        throw new InvalidDateRangeError('End date must be after start date');
      }
    }

    contains(date: DateTime): boolean {
      return date.isAfterOrEqual(this.start) && date.isBeforeOrEqual(this.end);
    }
  }
}

// Used by Sales, Inventory, and Shipping contexts
const orderTotal = new SharedKernel.Money(100, SharedKernel.Currency.USD);
const shippingAddress = new SharedKernel.Address('123 Main St', 'City', '12345', 'USA');
```

**When to Use:**
- Multiple contexts need the same domain concepts
- Duplication would lead to inconsistencies
- The shared code is stable and well-understood

**Guidelines:**
- Keep the shared kernel small
- Require team agreement to change
- Automate tests to prevent regressions

### Customer-Supplier

One context (supplier) provides services to another (customer), with the customer depending on the supplier's API.

```typescript
// Supplier Context (Inventory)
namespace InventoryContext {
  export interface InventoryService {
    reserveItems(items: OrderLineItem[]): Promise<ReservationResult>;
    releaseReservation(reservationId: ReservationId): Promise<void>;
    getStockLevel(productId: ProductId): Promise<StockLevel>;
  }

  export class InventoryServiceImpl implements InventoryService {
    async reserveItems(items: OrderLineItem[]): Promise<ReservationResult> {
      // Implementation for inventory reservation
    }

    async releaseReservation(reservationId: ReservationId): Promise<void> {
      // Implementation for releasing reservation
    }

    async getStockLevel(productId: ProductId): Promise<StockLevel> {
      // Implementation for stock level query
    }
  }
}

// Customer Context (Sales)
namespace SalesContext {
  export class OrderService {
    constructor(
      private readonly inventoryService: InventoryContext.InventoryService
    ) {}

    async createOrder(items: OrderItemInput[]): Promise<Order> {
      // Customer context uses supplier's API
      const reservation = await this.inventoryService.reserveItems(items);
      
      if (!reservation.success) {
        throw new InsufficientInventoryError(reservation.unavailableItems);
      }

      return this.completeOrderCreation(reservation.reservationId);
    }
  }
}
```

**When to Use:**
- Clear upstream/downstream relationship
- Supplier provides services to multiple customers
- Customer needs reliable supplier API

### Conformist

The downstream context adapts to the upstream model without translation, accepting the supplier's terminology.

```typescript
// Upstream defines the model (e.g., legacy system)
namespace LegacyERP {
  export interface CustomerRecord {
    CUST_ID: string;
    CUST_NAME: string;
    CUST_TYPE: 'STANDARD' | 'PREMIUM' | 'ENTERPRISE';
    CUST_SINCE: Date;
  }
}

// Downstream conforms to upstream model
namespace ModernCRM {
  // Direct mapping - no translation layer
  export class Customer implements Entity<CustomerId> {
    constructor(
      private readonly id: CustomerId,
      private name: CustomerName,
      private type: CustomerType,
      private since: DateTime
    ) {}

    static fromLegacy(record: LegacyERP.CustomerRecord): Customer {
      return new Customer(
        CustomerId.of(record.CUST_ID),
        CustomerName.of(record.CUST_NAME),
        CustomerType.fromLegacy(record.CUST_TYPE),
        DateTime.of(record.CUST_SINCE)
      );
    }
  }
}
```

**When to Use:**
- Upstream is unwilling or unable to accommodate downstream needs
- Downstream has limited influence
- Quick integration is more important than optimal model

### Anticorruption Layer

A translation layer that protects the downstream context from being contaminated by upstream model concepts.

```typescript
namespace DownstreamSystem {
  export class AnticorruptionLayer {
    private readonly upstreamAdapter: UpstreamAdapter;
    private readonly downstreamMapper: DomainMapper;

    async getProduct(productId: ProductId): Promise<DownstreamProduct> {
      // Convert downstream ID to upstream format
      const upstreamId = this.downstreamMapper.toUpstreamId(productId);
      
      // Call upstream API
      const upstreamProduct = await this.upstreamAdapter.getProduct(upstreamId);
      
      // Translate back to downstream model
      return this.downstreamMapper.toDownstreamProduct(upstreamProduct);
    }

    async submitOrder(order: DownstreamOrder): Promise<OrderConfirmation> {
      const upstreamOrder = this.downstreamMapper.toUpstreamOrder(order);
      const confirmation = await this.upstreamAdapter.submitOrder(upstreamOrder);
      return this.downstreamMapper.toConfirmation(confirmation);
    }
  }
}

// Translation logic
class DomainMapper {
  toUpstreamId(id: ProductId): string {
    return `UP-${id.value}`;
  }

  toDownstreamProduct(upstream: UpstreamProduct): DownstreamProduct {
    return new DownstreamProduct(
      ProductId.of(this.fromUpstreamId(upstream.id)),
      ProductName.of(upstream.name),
      Money.of(upstream.price, Currency.USD),
      ProductCategory.of(upstream.category)
    );
  }
}
```

**When to Use:**
- Integrating with legacy or external systems
- Upstream model is poor or frequently changing
- Need to isolate downstream from upstream changes

### Open Host Service

A context publishes a documented protocol for others to integrate with.

```typescript
namespace ShippingContext {
  // Published language (API contract)
  export interface ShippingService {
    calculateShipping(request: ShippingRequest): Promise<ShippingQuote>;
    createShipment(request: ShipmentRequest): Promise<ShipmentConfirmation>;
    trackShipment(trackingNumber: string): Promise<TrackingInfo>;
    cancelShipment(shipmentId: ShipmentId): Promise<void>;
  }

  // Request/Response DTOs
  export class ShippingRequest {
    constructor(
      readonly origin: Address,
      readonly destination: Address,
      readonly packages: Package[],
      readonly serviceLevel: ServiceLevel
    ) {}
  }

  export class ShippingQuote {
    constructor(
      readonly carrier: Carrier,
      readonly service: string,
      readonly cost: Money,
      readonly estimatedDays: number
    ) {}
  }
}

// Consumers implement against the published interface
class MyShippingAdapter implements ShippingContext.ShippingService {
  async calculateShipping(request: ShippingContext.ShippingRequest): Promise<ShippingContext.ShippingQuote> {
    // Integration with carrier APIs
  }
}
```

**When to Use:**
- Multiple external systems need to integrate
- You want to formalize integration contracts
- Building a platform or API

## Integration Patterns

### Event-Driven Integration

Using domain events for communication between bounded contexts.

```typescript
// Event definitions
namespace SalesEvents {
  export class OrderPlaced implements DomainEvent {
    constructor(
      readonly orderId: OrderId,
      readonly customerId: CustomerId,
      readonly total: Money,
      readonly occurredAt: DateTime
    ) {}
  }

  export class OrderShipped implements DomainEvent {
    constructor(
      readonly orderId: OrderId,
      readonly trackingNumber: TrackingNumber,
      readonly shippedAt: DateTime
    ) {}
  }
}

// Event publisher in Sales context
class Order extends AggregateRoot<OrderId> {
  ship(trackingNumber: TrackingNumber): void {
    this.status = OrderStatus.SHIPPED;
    this.trackingNumber = trackingNumber;
    this.recordEvent(new SalesEvents.OrderShipped(this.id, trackingNumber, DateTime.now()));
  }
}

// Event consumer in Shipping context
class ShippingEventHandler {
  async handle(event: DomainEvent): Promise<void> {
    switch (event.constructor.name) {
      case 'OrderPlaced':
        await this.createShipment(event as SalesEvents.OrderPlaced);
        break;
      case 'OrderShipped':
        await this.updateTracking(event as SalesEvents.OrderShipped);
        break;
    }
  }
}
```

### API-Based Integration

Synchronous integration through well-defined APIs.

```typescript
// REST API between contexts
class OrderController {
  constructor(
    private readonly orderService: OrderService
  ) {}

  @Post('/orders')
  async createOrder(@Body request: CreateOrderRequest): Promise<OrderResponse> {
    const order = await this.orderService.createOrder(request);
    return {
      orderId: order.id.value,
      status: order.status,
      total: order.total
    };
  }

  @Get('/orders/:id/shipping')
  async getShippingOptions(@Param('id') orderId: string): Promise<ShippingOptionsResponse> {
    // Call Shipping context via internal API
    const options = await this.shippingClient.getOptions(orderId);
    return options;
  }
}
```

## Context Map Documentation

Documenting the context map helps teams understand system boundaries:

```typescript
const contextMap = {
  boundedContexts: [
    {
      name: 'Sales',
      responsibilities: ['Order management', 'Pricing', 'Promotions'],
      ubiquitousLanguage: ['Order', 'LineItem', 'Cart', 'Checkout'],
      technology: 'TypeScript/Node.js'
    },
    {
      name: 'Inventory',
      responsibilities: ['Stock levels', 'Reservations', 'Warehousing'],
      ubiquitousLanguage: ['SKU', 'Stock', 'Reservation', 'Warehouse'],
      technology: 'Java/Spring'
    },
    {
      name: 'Shipping',
      responsibilities: ['Label generation', 'Tracking', 'Carrier integration'],
      ubiquitousLanguage: ['Shipment', 'Tracking', 'Carrier', 'Label'],
      technology: 'Python/FastAPI'
    }
  ],
  relationships: [
    {
      upstream: 'Sales',
      downstream: 'Inventory',
      type: 'Customer-Supplier',
      integration: 'Domain Events',
      protocol: 'Async'
    },
    {
      upstream: 'Sales',
      downstream: 'Shipping',
      type: 'Customer-Supplier',
      integration: 'REST API',
      protocol: 'Sync'
    },
    {
      upstream: 'Inventory',
      downstream: 'Shipping',
      type: 'Shared Kernel',
      sharedArtifacts: ['Address', 'Package']
    }
  ],
  sharedKernel: ['Money', 'Address', 'DateRange', 'Quantity']
};
```

## Best Practices

1. **Make relationships explicit**: Document how contexts interact
2. **Choose integration wisely**: Events for loose coupling, APIs for transactions
3. **Protect your context**: Use anticorruption layers when needed
4. **Keep kernel small**: Minimize shared code to reduce coupling
5. **Evolve boundaries**: Contexts can be split or merged as understanding grows
6. **Coordinate language**: Ensure mapping between different ubiquitous languages

## References

- Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software."
- Vernon, Vaughn. "Implementing Domain-Driven Design."
- "Context Mapping" patterns from the DDD community
