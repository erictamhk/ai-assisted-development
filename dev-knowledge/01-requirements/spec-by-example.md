# Specification by Example

## Overview

Specification by Example (SbE) is a requirements gathering and testing approach that uses concrete examples to illustrate and validate business requirements. Popularized by Gojko Adzic, this technique replaces abstract specifications with real-world scenarios that all stakeholders can understand and verify.

## Core Concept

```
┌─────────────────────────────────────────────────────────────────────────┐
│              SPECIFICATION BY EXAMPLE WORKFLOW                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. GATHER EXAMPLES              2. DERIVE SPECIFICATIONS          │
│  ┌───────────────────┐           ┌───────────────────┐             │
│  │ Business rules   │           │ Executable specs  │             │
│  │ expressed as    │──────────►│ written in       │             │
│  │ concrete        │   Derive   │ Gherkin/other    │             │
│  │ examples        │            │ DSL              │             │
│  └───────────────────┘           └───────────────────┘             │
│           │                              │                         │
│           │                              │                         │
│           ▼                              ▼                         │
│  3. AUTOMATE VALIDATION        4. LIVING DOCUMENTATION           │
│  ┌───────────────────┐           ┌───────────────────┐             │
│  │ Examples become   │           │ Documentation    │             │
│  │ automated tests  │◄─────────►│ always in sync   │             │
│  └───────────────────┘   Sync    with code             │
│                                                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

## Key Concepts

### Business Examples

```markdown
## Example: Shopping Cart Discount Rules

### Scenario 1: First-time customer gets welcome discount
**Given** a customer with no previous orders
**When** they apply the code "WELCOME10"
**Then** they receive 10% off their first order
**And** the discount is capped at $20

### Scenario 2: Loyalty discount for returning customers
**Given** a customer with more than 5 previous orders
**When** their cart total exceeds $100
**Then** they automatically receive 15% discount
**And** no promo code is required

### Scenario 3: Promotional events
**Given** the store is running a "BOGO" sale
**When** a customer adds 2 identical items to cart
**Then** they are charged for only 1 item
**And** the discount applies only to same-product pairs

### Scenario 4: Discount stacking rules
**Given** a cart with applicable discounts
**When** multiple discount conditions are met
**Then** the system applies the best single discount
**And** discounts do not combine
```

### From Examples to Specifications

```typescript
// Example-driven specification
interface DiscountSpecification {
  canApply(cart: ShoppingCart, customer: Customer): boolean;
  calculateDiscount(cart: ShoppingCart, customer: Customer): Money;
  priority: number;
}

class WelcomeDiscountSpecification implements DiscountSpecification {
  priority = 1;
  
  canApply(cart: ShoppingCart, customer: Customer): boolean {
    return customer.orderCount === 0 && 
           cart.hasPromoCode('WELCOME10');
  }
  
  calculateDiscount(cart: ShoppingCart, customer: Customer): Money {
    const discount = cart.subtotal.multiply(0.10);
    return discount.min(Money.fromAmount(20));
  }
}

class LoyaltyDiscountSpecification implements DiscountSpecification {
  priority = 2;
  
  canApply(cart: ShoppingCart, customer: Customer): boolean {
    return customer.orderCount > 5 && 
           cart.subtotal.greaterThan(Money.fromAmount(100));
  }
  
  calculateDiscount(cart: ShoppingCart, customer: Customer): Money {
    return cart.subtotal.multiply(0.15);
  }
}

class BogoSaleSpecification implements DiscountSpecification {
  priority = 3;
  
  canApply(cart: ShoppingCart, customer: Customer): boolean {
    return this.countEligiblePairs(cart) > 0;
  }
  
  calculateDiscount(cart: ShoppingCart, customer: Customer): Money {
    return this.calculatePairDiscounts(cart);
  }
}
```

## Gherkin Syntax

```gherkin
Feature: Shopping Cart Discounts

  Background:
    Given the discount engine is active
    And promotional periods are correctly configured

  Rule: Welcome discount for new customers

    Example: First-time customer using welcome code
      Given the customer has no previous orders
      When they enter "WELCOME10" at checkout
      Then the discount of 10% should be applied
      And the maximum discount should be $20

    Example: Welcome code not valid for returning customers
      Given the customer has 3 previous orders
      When they enter "WELCOME10" at checkout
      Then the discount should not be applied
      And an error message "Welcome code only valid for new customers" should be shown

  Rule: Loyalty discount applies automatically

    Example: Customer qualifies for loyalty discount
      Given the customer has 10 previous orders
      And their cart total is $150
      When they proceed to checkout
      Then a 15% discount should be automatically applied

    Example: Cart below threshold doesn't qualify
      Given the customer has 10 previous orders
      And their cart total is $50
      When they proceed to checkout
      Then no automatic discount should be applied

  Rule: Buy-one-get-one (BOGO) promotion

    Example: Customer adds 2 identical items
      Given a BOGO sale is active for "Widget"
      And the customer adds 2 "Widget" items to cart
      When they view the cart
      Then they should be charged for only 1 item
      And the discount should be $9.99 (price of one widget)

    Example: Customer adds 3 identical items
      Given a BOGO sale is active for "Widget"
      And the customer adds 3 "Widget" items to cart
      When they view the cart
      Then they should be charged for 2 items
      And the discount should be $9.99

  Rule: Discount stacking is not allowed

    Example: Multiple applicable discounts
      Given the customer has 10 previous orders
      And they enter "WELCOME10" at checkout
      And their cart total is $200
      When the discount is calculated
      Then only the loyalty discount (15%) should be applied
      And the welcome code should not be applied
```

## Example Mapping

```markdown
## Example Mapping Session

### Question: How do discounts work?

├── Business Rule: Welcome discount
│   ├── Example: First-time customer with code → 10% off, max $20
│   ├── Example: Returning customer with code → Invalid
│   └── Example: No code entered → No welcome discount
│
├── Business Rule: Loyalty discount
│   ├── Example: 10+ orders, cart $150+ → 15% off
│   ├── Example: 10+ orders, cart $50 → No discount
│   └── Example: 5 orders exactly → No discount
│
├── Business Rule: BOGO promotion
│   ├── Example: 2 identical items → Charge for 1
│   ├── Example: 3 identical items → Charge for 2
│   ├── Example: Mixed items → No BOGO
│   └── Example: BOGO with welcome code → BOGO applies
│
└── Business Rule: Single discount
    ├── Example: Both welcome and loyalty apply → Best discount
    ├── Example: BOGO with welcome code → BOGO applies
    └── Example: All three conditions → Best of all
```

## Living Documentation

```typescript
// Living documentation that stays in sync
import { describe, it, beforeAll, afterAll } from 'vitest';
import { DiscountEngine } from './discount-engine';
import { ShoppingCart } from './shopping-cart';
import { Customer } from './customer';

// This test suite IS the specification
// Running it validates the specification
describe('Discount Engine Specifications', () => {
  let engine: DiscountEngine;
  let database: TestDatabase;

  beforeAll(async () => {
    database = new TestDatabase();
    await database.seed([
      { type: 'product', sku: 'WIDGET', price: 9.99 },
      { type: 'customer', id: 'cust_1', orderCount: 0 },
      { type: 'customer', id: 'cust_2', orderCount: 10 }
    ]);
    
    engine = new DiscountEngine(database);
  });

  afterAll(async () => {
    await database.cleanup();
  });

  it('first-time customer gets 10% welcome discount', async () => {
    // From example: "First-time customer using welcome code"
    const cart = ShoppingCart.create()
      .addItem('WIDGET', 1)
      .addPromoCode('WELCOME10');
    
    const customer = database.customers.find('cust_1');
    
    const result = engine.calculateDiscount(cart, customer);
    
    expect(result.discountAmount).toBe(0.99); // 10% of $9.99
    expect(result.maxDiscountReached).toBe(false);
  });

  it('welcome discount capped at $20', async () => {
    // From example: "maximum discount should be $20"
    const cart = ShoppingCart.create()
      .addItem('EXPENSIVE_ITEM', 1, 250)
      .addPromoCode('WELCOME10');
    
    const customer = database.customers.find('cust_1');
    
    const result = engine.calculateDiscount(cart, customer);
    
    expect(result.discountAmount).toBe(20);
    expect(result.maxDiscountReached).toBe(true);
  });
});
```

## Benefits

| Aspect | Benefit |
|---------|---------|
| **Clarity** | Examples eliminate ambiguity |
| **Communication** | Bridges technical/business gap |
| **Test Coverage** | Examples become automated tests |
| **Living Docs** | Tests document expected behavior |
| **Confidence** | Verify requirements with examples |

## Anti-Patterns

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| **Over-Specification** | Too many edge cases | Focus on key scenarios |
| **Under-Specification** | Missing important examples | Collaborate with domain experts |
| **Technical Examples** | Examples focus on UI/technical details | Focus on business value |
| **One-Way Communication** | Examples only from BA | Collaborative workshops |

## AI Integration

```markdown
## AI Prompt for Specification by Example

Generate Gherkin specifications from these requirements:

```
Feature: User subscription billing

Requirements:
- Users choose monthly or annual plans
- Annual plans get 20% discount
- Payment attempted on subscription date
- Failed payments retry 3 times over 7 days
- User notified before each retry
- Subscription cancels after final failure
- Prorated refunds for annual plans cancelled mid-term
```

Output:
1. Gherkin Feature file with scenarios
2. Example mapping table
3. Questions for domain expert
4. Suggested test cases
```

## Best Practices

1. **Collaborate with Domain Experts**: Examples should come from business knowledge

2. **Use Realistic Data**: Examples should reflect actual business scenarios

3. **Cover Key Paths**: Focus on happy path and important edge cases

4. **Keep Examples Independent**: Each example should be self-contained

5. **Automate Validation**: Turn examples into tests

6. **Maintain Living Docs**: Keep examples in sync with code

## References

- "Specification by Example" by Gojko Adzic
- "Bridging the Communication Gap" by Gojko Adzic
- Cucumber.io
- Gherkin Reference
