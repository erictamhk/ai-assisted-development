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

---

## ezSpec: A Domain-Driven Testing Framework

ezSpec is a Java-based testing framework designed for DDD projects. It provides a structured approach to creating executable specifications with rule-based scenario organization.

### Core Concept

Each use case should have at least one corresponding ezSpec test case following these rules:

### Rule Usage Guide

ezSpec's Rule feature organizes test scenarios by business logic:

```java
// Define Rule constants
static final String SUCCESS_RULE = "Successful Creation";
static final String VALIDATION_RULE = "Input Validation";

// Create Rules (after feature.initialize())
@BeforeAll
static void beforeAll() {
    feature = Feature.New(FEATURE_NAME);
    feature.initialize();
    
    feature.NewRule(SUCCESS_RULE);
    feature.NewRule(VALIDATION_RULE);
}

// Assign scenarios to rules
@EzScenario(rule = SUCCESS_RULE)
public void successful_scenario() { }

// Or use withRule()
feature.newScenario().withRule(VALIDATION_RULE)
    .Given("condition", env -> { })
    .When("action", env -> { })
    .ThenFailure(env -> { })
    .Execute();
```

### Basic Example (without Rules)

```java
@EzFeature
@EzFeatureReport
public class CreateBoardUseCaseTest {

    static String FEATURE_NAME = "Create Board";
    static Feature feature = Feature.New(FEATURE_NAME);

    @BeforeAll
    static void beforeAll() {
        feature.initialize();
    }

    @EzScenario
    public void create_a_board() {
        feature.newScenario()
                .Given("a user who wants to create a board", env -> {
                    String boardId = UUID.randomUUID().toString();
                    String ownerId = UUID.randomUUID().toString();
                    String boardName = "My Kanban Board";

                    env.put("boardId", boardId)
                            .put("ownerId", ownerId)
                            .put("boardName", boardName);
                })
                .When("I create a board", env -> {
                    CreateBoardUseCase createBoardUseCase = getContext().newCreateBoardUseCase();
                    CreateBoardInput input = CreateBoardInput.create();
                    input.id = env.gets("boardId");
                    input.name = env.gets("boardName");
                    input.ownerId = env.gets("ownerId");

                    var output = createBoardUseCase.execute(input);
                    env.put("output", output)
                            .put("input", input);
                })
                .ThenSuccess(env -> {
                    var output = env.get("output", CqrsOutput.class);
                    assertNotNull(output.getId());
                    assertEquals(ExitCode.SUCCESS, output.getExitCode());
                })
                .And("the board should be persisted", env -> {
                    var output = env.get("output", CqrsOutput.class);
                    var input = env.get("input", CreateBoardInput.class);
                    Board board = getContext().boardRepository()
                        .findById(BoardId.valueOf(output.getId())).get();
                    assertEquals(output.getId(), board.getId());
                    assertEquals(input.name, board.getName());
                })
                .And("a board created event should be published", env -> {
                    List<DomainEvent> events = getContext().getPublishedEvents();
                    assertEquals(1, events.size());
                    assertTrue(events.get(0) instanceof BoardEvents.BoardCreated);
                })
                .Execute();
    }

    @AfterAll
    static void afterAll() {
        PlainTextReport report = new PlainTextReport();
        feature.accept(report);
        System.out.println(report.toString());
    }

    private TestContext getContext() {
        return TestContext.getInstance();
    }

    static class TestContext {
        private static TestContext instance;
        private GenericInMemoryRepository<Board, BoardId> boardRepository;
        private MessageBus<DomainEvent> messageBus;
        private List<DomainEvent> publishedEvents;

        private TestContext() {
            publishedEvents = new ArrayList<>();
            messageBus = new BlockingMessageBus();
            
            messageBus.register(event -> {
                if (event instanceof DomainEvent) {
                    publishedEvents.add((DomainEvent) event);
                }
            });

            boardRepository = new GenericInMemoryRepository<>(messageBus);
        }

        public static TestContext getInstance() {
            if (instance == null) {
                instance = new TestContext();
            }
            return instance;
        }

        public CreateBoardUseCase newCreateBoardUseCase() {
            return new CreateBoardService(boardRepository);
        }

        public GenericInMemoryRepository<Board, BoardId> boardRepository() {
            return boardRepository;
        }

        public List<DomainEvent> getPublishedEvents() {
            return new ArrayList<>(publishedEvents);
        }
    }
}
```

### Using Rules for Scenario Organization

```java
@EzFeature
@EzFeatureReport
public class CreatePlanUseCaseTest {

    static String FEATURE_NAME = "Create Plan";
    static Feature feature;
    
    // Define Rules
    static final String SUCCESSFUL_CREATION_RULE = "Successful Plan Creation";
    static final String INPUT_VALIDATION_RULE = "Input Validation";
    static final String DUPLICATE_VALIDATION_RULE = "Duplicate Validation";

    @BeforeAll
    static void beforeAll() {
        feature = Feature.New(FEATURE_NAME);
        feature.initialize();
        
        feature.NewRule(SUCCESSFUL_CREATION_RULE);
        feature.NewRule(INPUT_VALIDATION_RULE);
        feature.NewRule(DUPLICATE_VALIDATION_RULE);
    }

    @EzScenario(rule = SUCCESSFUL_CREATION_RULE)
    public void create_plan_successfully() {
        feature.newScenario()
                .Given("valid plan data", env -> {
                    env.put("planId", UUID.randomUUID().toString());
                    env.put("planName", "My Plan");
                    env.put("userId", "user123");
                })
                .When("I create the plan", env -> {
                    CreatePlanInput input = CreatePlanInput.create();
                    input.id = env.gets("planId");
                    input.name = env.gets("planName");
                    input.userId = env.gets("userId");
                    
                    var output = createPlanUseCase.execute(input);
                    env.put("output", output);
                })
                .ThenSuccess(env -> {
                    var output = env.get("output", CqrsOutput.class);
                    assertEquals(ExitCode.SUCCESS, output.getExitCode());
                })
                .Execute();
    }

    @EzScenario(rule = INPUT_VALIDATION_RULE)
    public void reject_empty_plan_name() {
        feature.newScenario()
                .Given("a plan with empty name", env -> {
                    env.put("planName", "");
                })
                .When("I try to create the plan", env -> {
                    // Implementation
                })
                .ThenFailure(env -> {
                    // Assertions for failure case
                })
                .Execute();
    }

    @EzScenario(rule = DUPLICATE_VALIDATION_RULE)
    public void reject_duplicate_plan_id() {
        feature.newScenario().withRule(DUPLICATE_VALIDATION_RULE)
                .Given("an existing plan", env -> {
                    // Setup existing plan
                })
                .When("I create a plan with same ID", env -> {
                    // Try to create duplicate
                })
                .ThenFailure(env -> {
                    // Verify rejection
                })
                .Execute();
    }
}
```

### Key Points for ezSpec

- `static Feature feature` is a static data member
- **Rules MUST be created after `feature.initialize()`**
- Use `@EzScenario(rule = ruleName)` or `withRule(ruleName)` to assign scenarios
- If no rule specified, scenarios归类到 default rule
- `@EzFeatureReport` generates readable test reports

---

## Spec Organization Guide

Organize specification files to reflect domain model Aggregate boundaries.

### Directory Structure

```
.dev/specs/
├── product/              # Product Aggregate
│   ├── entity/          # Product domain model specs
│   │   └── product-spec.md
│   └── usecase/         # Product-related Use Cases
│       ├── create-product.json
│       └── set-product-goal.json
│
├── pbi/                  # ProductBacklogItem Aggregate
│   ├── entity/
│   │   └── pbi-spec.md
│   └── usecase/
│       ├── create-pbi.json
│       └── estimate-pbi.json
│
├── sprint/               # Sprint Aggregate
│   ├── entity/
│   └── usecase/
│       ├── create-sprint.json
│       └── start-sprint.json
│
├── team/                 # Team Aggregate
│   └── ...
└── tag/                  # Tag Aggregate
    └── ...
```

### Naming Conventions

| Type | Format | Example |
|------|--------|---------|
| Use Case Spec | `[action]-[aggregate].json` | `create-product.json` |
| Entity Spec | `[aggregate]-spec.md` | `product-spec.md` |

### Determining Spec Location

**Step 1**: Ask "Which Aggregate does this Use Case primarily operate on?"

**Step 2**: Verify using Aggregate identification checklist

**Step 3**: Place spec in the correct aggregate directory

### Common Mistakes

| Mistake | Why It's Wrong | Correct Location |
|---------|---------------|------------------|
| `product/usecase/create-pbi.json` | PBI is independent Aggregate | `pbi/usecase/create-pbi.json` |
| `product/usecase/create-sprint.json` | Sprint is independent Aggregate | `sprint/usecase/create-sprint.json` |
| `specs/usecases/create-product.json` | Loses Aggregate boundary visibility | `product/usecase/create-product.json` |

### Migration Guide

If specs are in wrong location:

```bash
# Step 1: Create correct directory structure
mkdir -p .dev/specs/[aggregate]/usecase

# Step 2: Move file
mv .dev/specs/product/usecase/create-sprint.json \
   .dev/specs/sprint/usecase/

# Step 3: Update references
# Update task-*.json spec.useCase paths
# Update test file references
# Update documentation links

# Step 4: Validate
grep -r "old/path" .  # Confirm no missing references
```

### Aggregate Reference Table

| Aggregate | Spec Directory | Primary Use Cases |
|-----------|---------------|-------------------|
| Product | `product/` | CreateProduct, SetProductGoal |
| ProductBacklogItem | `pbi/` | CreatePBI, EstimatePBI, AssignPBI |
| Sprint | `sprint/` | CreateSprint, StartSprint, CloseSprint |
| Team | `team/` | CreateTeam, AddMember, RemoveMember |
| Tag | `tag/` | CreateTag, DeleteTag, AssignTag |

---

## Use Case Spec Template

### Query Use Case Spec

```json
{
  "query": "[QueryName]",
  "behavior": "[描述查詢行為]",
  "input": [
    { "name": "[inputField]", "type": "[Type]", "note": "[說明]" }
  ],
  "method": "[projection].query",
  "output": "[OutputType]",
  "domainModelNotes": [
    "[任何領域模型相關說明]"
  ],
  "dependencies": [
    { "name": "[dependencyName]", "type": "[DependencyType]", "note": "[用途說明]" }
  ],
  "projections": [
    {
      "name": "[ProjectionName] extends Projection<[Input], [Output]>",
      "description": "[Projection 用途說明]",
      "location": "[aggregate].usecase.port.out.projection",
      "implementation": "[aggregate].adapter.out.projection.Jpa[ProjectionName]",
      "critical": true,
      "input": [
        { "name": "[inputField]", "type": "[Type]" }
      ]
    }
  ],
  "dataTransferObjects": [
    {
      "name": "[DtoName]",
      "description": "[DTO 用途說明]",
      "location": "[aggregate].usecase.port",
      "fields": [
        { "name": "[fieldName]", "type": "[Type]" }
      ]
    }
  ],
  "mappers": [
    {
      "name": "[MapperName]",
      "description": "🚨 CRITICAL: Must be generated, handles Entity to DTO conversion",
      "location": "[aggregate].usecase.port",
      "critical": true,
      "methods": [
        "toDto([Entity] entity) -> [Dto]",
        "toDomain([Dto] dto) -> [Entity] (if needed)"
      ],
      "note": "⚠️ Important: Mapper must be in usecase.port package, NOT in adapter!"
    }
  ],
  "_implementation_checklist": {
    "note": "Must complete all items during implementation",
    "items": [
      "✅ Query UseCase Interface (with Input/Output inner classes)",
      "✅ Query Service Implementation (in usecase.service package)",
      "✅ All DTOs listed in dataTransferObjects",
      "✅ All Mappers listed in mappers section (CRITICAL!)",
      "✅ Projection Interface (in usecase.port.out.projection)",
      "✅ Projection JPA Implementation (in adapter.out.projection)",
      "✅ BDD Tests"
    ]
  }
}
```

---

## Ubiquitous Language in Requirements Modeling

### The Challenge of Domain Concepts

**Scrum Core Artifacts困境**:
- Product Backlog is one of three core Scrum artifacts
- Product Owner manages Product Backlog as primary responsibility
- DDD modeling dilemma:
  1. As Entity/Value Object → Over-design, doesn't need ID
  2. As Query → Domain model missing core concept

### Solution: Domain Service as First-Class Citizen

```java
// Product Backlog as Domain Service (first-class citizen)
@DomainService
public class ProductBacklog {
    private final BacklogItemRepository repository;
    
    // Product Backlog's core responsibility: maintain priority ordering
    public void reorder(ProductId productId, List<BacklogItemId> newOrder) {
        // This is Product Backlog's core business logic!
        List<BacklogItem> items = repository.findByProductId(productId);
        
        // Business rule: Only READY items can be at top
        validateOrderingRules(items, newOrder);
        
        for (int i = 0; i < newOrder.size(); i++) {
            BacklogItem item = findById(items, newOrder.get(i));
            item.updateOrder(i);
            repository.save(item);
        }
        
        publishEvent(new ProductBacklogReordered(productId, newOrder));
    }
    
    // Product Backlog's business rules
    public void prepareForSprintPlanning(ProductId productId) {
        List<BacklogItem> items = getTopItems(productId, 10);
        
        // Business rule: Before Sprint Planning, top 10 items must be READY
        for (BacklogItem item : items) {
            if (!item.isReady()) {
                throw new NotReadyForSprintException(
                    "Top items must be READY before Sprint Planning"
                );
            }
        }
    }
    
    // Product Backlog queries (still domain concept)
    public List<BacklogItem> getRefinementCandidates(ProductId productId) {
        return repository.findByProductId(productId).stream()
            .filter(item -> item.needsRefinement())
            .limit(5)
            .toList();
    }
}
```

### Benefits of This Design

1. **Complete Ubiquitous Language**: Product Backlog is first-class domain citizen
2. **Clear Responsibilities**: Encapsulates Product Backlog-specific business rules
3. **No Over-Design**: No forced ID or Entity pattern

### Complete Domain Model Design

**1. Product Aggregate (the product itself)**
```java
@AggregateRoot
public class Product {
    private ProductId productId;
    private ProductName name;
    private ProductVision vision;
    private ProductGoal currentGoal;
    private ProductOwnerId productOwnerId;
    
    // Product's responsibility: manage product-level concepts
    public void setProductGoal(ProductGoal goal) {
        this.currentGoal = goal;
        publishEvent(new ProductGoalSet(productId, goal));
    }
    
    // Note: Doesn't directly manage Backlog Items
}
```

**2. BacklogItem Aggregate (individual items)**

The key insight is that Product Backlog should be modeled as a Domain Service rather than an Entity, because:
- It represents a collection/ordering concept, not an object with identity
- It encapsulates specific business rules (ordering, ready state validation)
- It avoids over-designing with unnecessary IDs

This approach maintains the integrity of the Ubiquitous Language while keeping the domain model clean and purposeful.
