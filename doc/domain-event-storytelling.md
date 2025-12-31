# Domain Event Storytelling

## Overview

Domain Event Storytelling is a technique that uses narrative structures to communicate and explore domain events. It extends Event Storming by adding temporal context, character motivations, and story arcs to make domain events more meaningful and memorable. This approach leverages the human brain's natural affinity for stories to improve domain understanding.

## Core Concept

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DOMAIN EVENT STORYTELLING                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  STORY STRUCTURE → EVENT MAPPING                                       │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Once upon a time...  (Initial State)                        │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  [Customer browses catalog]                                    │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  Every day...  (Recurring Pattern)                           │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  [Customer adds items to cart]                               │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  One day...  (Trigger Event)                                 │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  [Customer decides to checkout]                               │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  Because of that...  (Causal Chain)                          │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  [System validates inventory]                                │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  [Customer enters payment]                                  │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  [Payment is processed]                                     │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  Until finally...  (Resolution)                              │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  [Order is confirmed]                                       │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  And ever since...  (New Normal)                           │   │
│  │      │                                                        │   │
│  │      ▼                                                        │   │
│  │  [Customer receives confirmation]                            │   │
│  │                                                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Story Template

```markdown
## Domain Event Story Template

### Title: [Descriptive name of the scenario]

### Characters (Actors)
- **Customer**: Primary actor who initiates actions
- **System**: The software system being designed
- **External Services**: Third-party systems involved

### Setting (Context)
- Where does this story take place?
- What is the current state of the system?
- What constraints exist?

### Plot (Event Sequence)

#### Act 1: Normal Operations
```
[Event]: Initial action in the story
Trigger: What caused this event?
Actor: Who/what triggered it?
Outcome: What happened as a result?
```

#### Act 2: Complication
```
[Event]: Something changes the flow
Trigger: What disrupted the normal flow?
Decision Point: What choices emerge?
```

#### Act 3: Resolution
```
[Event]: The story resolves
Outcome: What is the final state?
Consequence: What effects ripple outward?
```

### Key Events (Domain Events)
1. [Event name] - [Brief description]
2. [Event name] - [Brief description]
...

### Turning Points (Hotspots)
- What questions arose?
- What risks were identified?
- What opportunities were discovered?

### Moral (Lessons Learned)
- What did we learn about the domain?
- What should we remember for implementation?
```

## Example Story

```markdown
## Story: The Abandoned Cart Recovery

### Characters
- **E-commerce Customer**: Browse and shop
- **Cart Service**: Manages shopping cart state
- **Inventory Service**: Tracks product availability
- **Email Service**: Sends notifications
- **Marketing Team**: Wants to recover abandoned carts

### Setting
An e-commerce platform where customers can browse products, add them to a cart, and complete purchases. The cart is persisted across sessions.

### Plot

#### Act 1: Normal Operations
**Customer browses products**
- Trigger: Customer visits the website
- Action: Views product catalog
- Outcome: Customer finds items of interest

**Customer adds items to cart**
- Trigger: Customer clicks "Add to Cart"
- Action: System records items in cart
- Outcome: Cart contains items, persists to database

#### Act 2: Complication
**Customer leaves without purchasing**
- Trigger: Customer closes browser, abandons cart
- Action: Cart remains in "PENDING" state
- Decision: What should trigger a recovery attempt?

**Inventory reservation expires**
- Trigger: Time passes (24 hours)
- Action: Reserved items are released
- Decision: How do we handle pre-reserved items?

#### Act 3: Resolution
**Cart recovery notification sent**
- Trigger: 1 hour after abandonment
- Action: Email sent to customer
- Outcome: Customer receives reminder

**Customer returns and completes purchase**
- Trigger: Customer clicks email link
- Action: Cart restored with current prices
- Outcome: Order placed, payment processed

### Key Events
1. `ItemAddedToCart` - Customer added item to cart
2. `CartAbandoned` - Cart inactive for 2 hours
3. `InventoryReservationExpired` - Reserved items released
4. `RecoveryEmailSent` - Notification sent to customer
5. `CartRestored` - Customer returned via link
6. `OrderPlaced` - Customer completed purchase

### Turning Points
- **Question**: At what point should we consider a cart "abandoned"?
- **Risk**: Customer might feel spammed with recovery emails
- **Opportunity**: Personalize recovery based on cart contents

### Moral
- Event timing matters as much as event sequence
- Customer communication must balance persistence with respect
- External systems (inventory) affect customer experience
```

## Translating Stories to Code

```typescript
// From the story, we identify these domain events:

namespace CartEvents {
  export class ItemAddedToCart implements DomainEvent {
    readonly cartId: CartId;
    readonly productId: ProductId;
    readonly quantity: Quantity;
    readonly addedAt: DateTime;

    constructor(cartId: CartId, productId: ProductId, quantity: Quantity) {
      this.cartId = cartId;
      this.productId = productId;
      this.quantity = quantity;
      this.addedAt = DateTime.now();
    }
  }

  export class CartAbandoned implements DomainEvent {
    readonly cartId: CartId;
    readonly items: CartItemSnapshot[];
    readonly abandonedAt: DateTime;

    constructor(cartId: CartId, items: CartItemSnapshot[]) {
      this.cartId = cartId;
      this.items = items;
      this.abandonedAt = DateTime.now();
    }
  }

  export class RecoveryEmailSent implements DomainEvent {
    readonly cartId: CartId;
    readonly customerEmail: Email;
    readonly templateUsed: string;
    readonly sentAt: DateTime;

    constructor(cartId: CartId, customerEmail: Email, templateUsed: string) {
      this.cartId = cartId;
      this.customerEmail = customerEmail;
      this.templateUsed = templateUsed;
      this.sentAt = DateTime.now();
    }
  }

  export class CartRestored implements DomainEvent {
    readonly cartId: CartId;
    readonly originalCartId: CartId;
    readonly restoredAt: DateTime;

    constructor(cartId: CartId, originalCartId: CartId) {
      this.cartId = cartId;
      this.originalCartId = originalCartId;
      this.restoredAt = DateTime.now();
    }
  }
}

// Aggregate behavior from story

class Cart extends AggregateRoot<CartId> {
  private status: CartStatus;
  private items: CartItem[];
  private lastActivityAt: DateTime;

  constructor(id: CartId) {
    super(id);
    this.status = CartStatus.EMPTY;
    this.items = [];
    this.lastActivityAt = DateTime.now();
  }

  addItem(product: Product, quantity: Quantity): void {
    if (this.status === CartStatus.ABANDONED) {
      throw new CannotModifyAbandonedCartError();
    }

    const existing = this.items.find(i => i.productId.equals(product.id));
    if (existing) {
      existing.increaseQuantity(quantity);
    } else {
      this.items.push(CartItem.create(this.id, product, quantity));
    }

    this.status = this.items.length > 0 ? CartStatus.ACTIVE : CartStatus.EMPTY;
    this.lastActivityAt = DateTime.now();
    
    this.recordEvent(new ItemAddedToCart(this.id, product.id, quantity));
  }

  markAsAbandoned(): void {
    if (this.status !== CartStatus.ACTIVE) return;
    
    this.status = CartStatus.ABANDONED;
    this.recordEvent(new CartAbandoned(this.id, this.snapshot()));
  }

  isEligibleForRecovery(afterHours: number = 2): boolean {
    const hoursSinceActivity = DateTime.now().diff(this.lastActivityAt).hours;
    return this.status === CartStatus.ABANDONED && 
           hoursSinceActivity >= afterHours &&
           this.items.length > 0;
  }
}
```

## Story-Driven Testing

```typescript
// Feature: Cart Abandonment Recovery
import { describe, it, beforeEach } from 'vitest';
import { Cart } from './cart.aggregate';
import { CartStatus } from './cart-status';
import { Money } from '../../shared/money';

describe('Cart Abandonment Recovery Story', () => {
  describe('Act 1: Normal Operations', () => {
    it('customer can add items to their cart', () => {
      const cart = Cart.create(CartId.generate());
      const product = ProductFixture.create({ price: Money.fromAmount(29.99) });
      
      cart.addItem(product, Quantity.of(2));
      
      expect(cart.items).toHaveLength(2);
      expect(cart.total).toEqual(Money.fromAmount(59.98));
    });
  });

  describe('Act 2: Complication', () => {
    it('cart becomes abandoned after inactivity', () => {
      const cart = Cart.create(CartId.generate());
      cart.addItem(ProductFixture.create(), Quantity.of(1));
      
      // Simulate time passing
      cart.markAsAbandoned();
      
      expect(cart.status).toBe(CartStatus.ABANDONED);
    });

    it('cannot modify abandoned cart', () => {
      const cart = Cart.create(CartId.generate());
      cart.addItem(ProductFixture.create(), Quantity.of(1));
      cart.markAsAbandoned();
      
      expect(() => {
        cart.addItem(ProductFixture.create(), Quantity.of(1));
      }).toThrow(CannotModifyAbandonedCartError);
    });
  });

  describe('Act 3: Resolution', () => {
    it('cart can be recovered via email link', () => {
      const abandonedCart = Cart.create(CartId.generate());
      abandonedCart.addItem(ProductFixture.create(), Quantity.of(1));
      abandonedCart.markAsAbandoned();
      
      const recoveredCart = Cart.recover(abandonedCart.id);
      
      expect(recoveredCart.status).toBe(CartStatus.ACTIVE);
      expect(recoveredCart.items).toHaveLength(1);
    });

    it('restored cart reflects current prices', () => {
      const abandonedCart = Cart.create(CartId.generate());
      abandonedCart.addItem(ProductFixture.create({ 
        price: Money.fromAmount(29.99) 
      }), Quantity.of(1));
      abandonedCart.markAsAbandoned();
      
      // Price changed while abandoned
      const recoveredCart = Cart.recover(abandonedCart.id);
      
      expect(recoveredCart.items[0].unitPrice).toEqual(
        Money.fromAmount(34.99) // Current price
      );
    });
  });
});
```

## Benefits of Storytelling

| Aspect | Benefit |
|---------|---------|
| **Recall** | Stories are easier to remember than lists |
| **Context** | Events have clear "why" not just "what" |
| **Engagement** | Stories capture attention and imagination |
| **Alignment** | Common narrative unites team understanding |
| **Exploration** | Stories naturally surface edge cases |

## AI Integration

```markdown
## AI Prompt for Story Generation

Generate a domain event story from the following requirements:

```
Feature: User subscription management

Requirements:
- Users can subscribe to monthly plans
- Subscriptions auto-renew unless cancelled
- Payment is processed monthly
- Users receive renewal reminders
- Failed payments trigger retry logic
- Users can cancel at any time
```

Output format:
1. Character descriptions
2. Story with three acts
3. Key domain events
4. Edge cases discovered
5. Questions for domain expert
```

## Best Practices

1. **Use Real Language**: Avoid technical jargon in stories
2. **Include Failures**: What happens when things go wrong?
3. **Show Causality**: Events should connect logically
4. **Keep it Concise**: Stories should be digestible
5. **Iterate**: Refine stories with domain experts

## References

- "Domain Storytelling" by Stefan Hofer
- EventStorming.com
- "User Story Mapping" by Jeff Patton
