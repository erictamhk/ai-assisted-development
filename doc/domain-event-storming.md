# Domain Event Storming

## Overview

Domain Event Storming is a collaborative workshop technique for exploring complex business domains by focusing on the events that occur within the system. Created by Alberto Brandolini, it brings together domain experts and developers to discover, model, and analyze domain events in real-time, using sticky notes on a large surface.

## Core Concepts

### Domain Events

A domain event represents something that happened in the system that domain experts care about:

```
┌─────────────────────────────────────────────────────────────────────┐
│                     DOMAIN EVENT STORMING                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  [Order Placed] ──► [Payment Received] ──► [Order Confirmed]    │
│       │                  │                    │                     │
│       ▼                  ▼                    ▼                     │
│  Orange Sticky        Orange Sticky        Orange Sticky           │
│  "Something that     "Something that      "Something that         │
│   happened"           happened"             happened"               │
│                                                                     │
│  CAUSED BY:           CAUSED BY:            CAUSED BY:            │
│  Customer clicks      Payment gateway       System verifies         │
│  "Place Order"       confirms payment      inventory              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Event Storming Symbols

| Symbol | Color | Meaning |
|--------|-------|----------|
| Domain Event | Orange | Something that happened |
| Command | Blue | User action or system trigger |
| Aggregate | Yellow | Entity that handles commands |
| Policy | Purple | Business rules/decision logic |
| External System | Pink | External services or systems |
| Hotspot | Red | Questions, problems, risks |
| Timeline | ───► | Event sequence |

## Event Storming Levels

### Level 1: Big Picture

For exploring the overall domain structure:

```
┌────────────────────────────────────────────────────────────────────────────┐
│                    BIG PICTURE EVENT STORMING                           │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  ════════════════════════════════════════════════════════════════════════  │
│  │                                                               │   │
│  │  [User]──►[Browse]──►[Add to]──►[Check]──►[Place]──►[Pay] │   │
│  │   │      Products    Cart       Out    Order    Order              │   │
│  │   │                                                         │   │
│  │  👤  Customer                                          💳  │   │
│  │  📧  Email System  📦  Inventory    💳  Payment        │   │
│  │               🟥  Hotspot: How do we handle backorders?         │   │
│  │  🟢  Opportunity: Real-time inventory sync                │   │
│  │                                                               │   │
│  ════════════════════════════════════════════════════════════════════════  │
│                                                                        │
│  GOALS:                                                              │
│  • Discover domain events                                              │
│  • Identify hotspots and opportunities                                 │
│  • Align understanding across teams                                    │
│                                                                        │
└────────────────────────────────────────────────────────────────────────────┘
```

### Level 2: Process Modeling

For detailed workflow analysis:

```
┌────────────────────────────────────────────────────────────────────────────┐
│                    PROCESS MODELING                                    │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  👤 Customer                                                       │
│     │                                                               │
│     ▼                                                               │
│  🟦 [Browse Products]                                              │
│     │                                                               │
│     ▼                                                               │
│  🟣 Policy: Display products from active inventory                     │
│     │                                                               │
│     ▼                                                               │
│  🟦 [Add to Cart]                                                  │
│     │                                                               │
│     ▼                                                               │
│  🟣 Policy: Validate quantity limits                                 │
│     │                                                               │
│     ▼                                                               │
│  🟦 [Review Cart]                                                  │
│     │                                                               │
│     ▼                                                               │
│  🟣 Policy: Calculate shipping options                              │
│     │                                                               │
│     ▼                                                               │
│  🟦 [Place Order]                                                 │
│     │                                                               │
│     ▼                                                               │
│  💳 Payment System                                                 │
│     │                                                               │
│     ▼                                                               │
│  🟦 [Order Confirmed]                                              │
│     │                                                               │
│     │  ───► 📧 Notification Sent                                   │
│     │                                                               │
│  ════════════════════════════════════════════════════════════════════════  │
│                                                                        │
│  KEY: 🟦 Event  🟣 Policy  👤 Actor  💳 External System            │
│                                                                        │
└────────────────────────────────────────────────────────────────────────────┘
```

### Level 3: Software Design

For translating events to code structure:

```typescript
// Event Storming → Code Translation

// From the event storming, we identified:
// Events: OrderPlaced, PaymentReceived, OrderConfirmed
// Commands: PlaceOrder, MakePayment, ConfirmOrder
// Aggregates: Order, Payment

// Domain Events
namespace OrderEvents {
  export class OrderPlaced implements DomainEvent {
    readonly orderId: OrderId;
    readonly customerId: CustomerId;
    readonly totalAmount: Money;
    readonly occurredAt: DateTime;

    constructor(orderId: OrderId, customerId: CustomerId, totalAmount: Money) {
      this.orderId = orderId;
      this.customerId = customerId;
      this.totalAmount = totalAmount;
      this.occurredAt = DateTime.now();
    }
  }

  export class PaymentReceived implements DomainEvent {
    readonly paymentId: PaymentId;
    readonly orderId: OrderId;
    readonly amount: Money;
    readonly method: PaymentMethod;

    constructor(paymentId: PaymentId, orderId: OrderId, amount: Money, method: PaymentMethod) {
      this.paymentId = paymentId;
      this.orderId = orderId;
      this.amount = amount;
      this.method = method;
    }
  }
}

// Aggregate (from Policy: Handle order lifecycle)
class Order extends AggregateRoot<OrderId> {
  private status: OrderStatus;
  private lineItems: OrderLineItem[];
  private payment: Payment | null;

  constructor(id: OrderId) {
    super(id);
    this.status = OrderStatus.DRAFT;
    this.lineItems = [];
    this.payment = null;
  }

  static fromCustomer(customerId: CustomerId): Order {
    const order = new Order(OrderId.generate());
    order.recordEvent(new OrderCreated(customerId));
    return order;
  }

  placeOrder(): void {
    if (this.lineItems.length === 0) {
      throw new CannotPlaceEmptyOrderError();
    }
    this.status = OrderStatus.PLACED;
    this.recordEvent(new OrderPlaced(this.id, this.customerId, this.calculateTotal()));
  }

  receivePayment(amount: Money, method: PaymentMethod): void {
    if (this.status !== OrderStatus.PLACED) {
      throw new InvalidOrderStateError('Cannot receive payment for this order');
    }
    this.payment = Payment.create(this.id, amount, method);
    this.status = OrderStatus.PAID;
    this.recordEvent(new PaymentReceived(this.payment.id, this.id, amount, method));
  }
}
```

## Facilitation Guide

### Preparation

```markdown
## Event Storming Session Checklist

### Before the Session
- [ ] Book a room with wall space (8+ meters ideally)
- [ ] Prepare materials:
  - [ ] Orange sticky notes (events)
  - [ ] Blue sticky notes (commands)
  - [ ] Yellow sticky notes (aggregates)
  - [ ] Purple sticky notes (policies)
  - [ ] Pink sticky notes (external systems)
  - [ ] Red sticky notes (hotspots)
  - [ ] Green sticky notes (opportunities)
  - [ ] Large markers for everyone
  - [ ] Timer for timeboxing
- [ ] Invite participants:
  - [ ] Domain experts (mandatory)
  - [ ] Developers (mandatory)
  - [ ] Product owner/stakeholder
  - [ ] Facilitator (if not you)

### Materials Setup
┌─────────────────────────────────────────┐
│  🟧 Events      |  "Something that      │
│                |   happened"           │
├─────────────────────────────────────────┤
│  🟦 Commands    |  "User action or     │
│                |   system trigger"     │
├─────────────────────────────────────────┤
│  🟨 Aggregates  |  "Entity that       │
│                |   handles commands"  │
├─────────────────────────────────────────┤
│  🟪 Policies    |  "Business rules"   │
├─────────────────────────────────────────┤
│  🟥 External    |  "Outside systems" │
├─────────────────────────────────────────┤
│  🟥 Hotspots    |  "Questions/risks" │
├─────────────────────────────────────────┤
│  🟩 Opportunity |  "Improvements"    │
└─────────────────────────────────────────┘
```

### Session Flow

```markdown
## Event Storming Agenda

### 1. Introduction (10 min)
- Welcome and purpose
- Explain notation symbols
- Set ground rules

### 2. Exploration Phase (30 min)
- Write events on orange stickies
- Don't organize yet - just capture
- Everyone writes simultaneously

### 3. Timeline Assembly (20 min)
- Arrange events chronologically
- Group related events
- Identify gaps

### 4. Actor Identification (15 min)
- Who triggers these events?
- Who is affected?

### 5. Policy Discovery (30 min)
- What decisions are made?
- Where do branch points occur?

### 6. Hotspot Identification (15 min)
- Mark questions with red
- Mark opportunities with green

### 7. Synthesis (20 min)
- Document key findings
- Prioritize areas to explore further
- Plan next steps
```

## Event Storming in AI-Assisted Development

```markdown
## AI Prompt for Event Storming Analysis

Based on the following event storming notes, generate TypeScript domain code:

```
Events identified:
- Customer browses products
- Customer adds item to cart
- Customer removes item from cart
- Customer applies promo code
- System calculates discounts
- Customer places order
- System validates inventory
- System reserves inventory
- Customer enters payment info
- Payment is processed
- Order is confirmed
- Inventory is updated
- Customer receives confirmation email

Hotspots discovered:
- What happens when inventory reservation fails?
- How to handle partial inventory availability?

Policies:
- Only 1 promo code per order
- Inventory reserved for 15 minutes
- Payment must be received within 30 minutes
```

Generate:
1. Domain events with TypeScript interfaces
2. Aggregate structure (Cart, Order, Inventory)
3. Business rules as methods
4. Sample use case implementation
5. Questions raised by the analysis

```typescript
// Output structure
namespace CustomerEvents {
  export class ProductBrowsed { /* ... */ }
  export class ItemAddedToCart { /* ... */ }
  // ...
}

class Cart implements AggregateRoot<CartId> {
  // Business logic
}
```

## Output
[Generated TypeScript domain code]
```

## Best Practices

### Do's

1. **Include Domain Experts**: Their knowledge is essential
2. **Use Simple Language**: Avoid technical jargon during exploration
3. **Timebox Each Phase**: Keep energy and focus
4. **Embrace Hotspots**: Questions are valuable findings
5. **Take Photos**: Document the results

### Don'ts

1. **Don't Start with Code**: Focus on understanding first
2. **Don't Force Consensus**: Document disagreements
3. **Don't Skip the Timeline**: Sequence matters
4. **Don't Ignore Biddable Domains**: People are part of the system
5. **Don't Stop at First Solution**: Explore alternatives

## Common Patterns

### Event Sourcing Ready

```
┌─────────────────────────────────────────────┐
│  Event Storming → Event Sourcing           │
├─────────────────────────────────────────────┤
│                                             │
│  [Order Placed]                            │
│         │                                  │
│         ▼                                  │
│  Event Store:                              │
│  - OrderPlacedEvent { orderId, items }    │
│  - ItemAddedEvent { orderId, item }       │
│  - ItemRemovedEvent { orderId, item }      │
│  - PaymentReceivedEvent { orderId, ... }   │
│  - OrderConfirmedEvent { orderId }          │
│                                             │
│  Rebuild State:                            │
│  Order = apply([all events])              │
│                                             │
└─────────────────────────────────────────────┘
```

### CQRS Ready

```
┌─────────────────────────────────────────────┐
│  Event Storming → CQRS                     │
├─────────────────────────────────────────────┤
│                                             │
│  Commands:                                  │
│  - PlaceOrder                               │
│  - AddItemToCart                            │
│  - RemoveItemFromCart                       │
│  - CompletePayment                          │
│                                             │
│  Queries:                                   │
│  - GetCartContents                          │
│  - GetOrderStatus                           │
│  - GetOrderHistory                          │
│                                             │
│  Events:                                    │
│  - OrderPlaced (write side)                 │
│  - CartUpdated (read side projection)       │
│                                             │
└─────────────────────────────────────────────┘
```

## References

- "Introducing EventStorming" by Alberto Brandolini
- EventStorming.com
- "Domain-Driven Design" by Eric Evans
- "Learning Domain-Driven Design" by Vladik Khononov
