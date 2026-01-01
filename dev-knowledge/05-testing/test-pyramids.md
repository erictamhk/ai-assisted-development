# Test Pyramids

The Test Pyramid is a fundamental concept in software quality engineering that provides a structured framework for balancing test effort, isolation, and execution speed across different levels of testing. This document covers the test pyramid model, its layers, and AI-specific guidelines for implementing an effective testing strategy.

---

## The Test Pyramid Overview

The Test Pyramid, introduced by Mike Cohn in his book "Succeeding with Agile" (2009), is a visual metaphor for creating an effective test automation strategy:

```
                    🔺 E2E / UI Tests
                   /   (Few, Slow, Expensive)
                  /
         🟡 Integration / Service Tests
        /           (Some, Medium speed)
       /
✅ Unit Tests
(Many, Fast, Cheap)
```

**Core Principle:** Tests become broader, fewer, and more expensive to execute as you move up the pyramid. The base should be wide and solid.

---

## The Three Layers

### Layer 1: Unit Tests

Unit tests form the foundation of the pyramid. They focus on the narrowest scope, typically a single function, method, or class.

**Characteristics:**
- Fast execution (milliseconds)
- Highly isolated (no external dependencies)
- Large quantity (70-80% of test suite)
- Easy to write and maintain
- Provide immediate feedback

**What to Test:**
- Pure functions
- Individual methods
- Value objects and entities
- Business logic in isolation
- Edge cases and boundary conditions

**Implementation:**

```typescript
// ✅ GOOD - Focused unit test
describe('Money', () => {
  describe('add', () => {
    it('should add two money values with same currency', () => {
      const money1 = new Money(100, 'USD');
      const money2 = new Money(50, 'USD');
      
      const result = money1.add(money2);
      
      expect(result.amount).toBe(150);
      expect(result.currency).toBe('USD');
    });

    it('should throw when currencies do not match', () => {
      const usd = new Money(100, 'USD');
      const eur = new Money(100, 'EUR');
      
      expect(() => usd.add(eur)).toThrow(CurrencyMismatchError);
    });
  });
});

// ❌ BAD - Testing too much, external dependencies
describe('OrderService', () => {
  it('should process order end-to-end', async () => {
    const order = await database.getOrder('order-123');
    const result = await orderService.process(order);
    await paymentGateway.charge(order.customer, result.total);
    expect(result.status).toBe('PROCESSED');
  });
});
```

**AI Guidelines:**
- AI should generate unit tests for all domain logic
- AI should mock external dependencies (database, APIs, services)
- AI should follow Arrange-Act-Assert pattern
- AI should test one behavior per test case
- AI should aim for 80%+ coverage on domain layer

---

### Layer 2: Integration / Service Tests

Integration tests verify the interaction between multiple components, modules, or services.

**Characteristics:**
- Moderate execution time (seconds to minutes)
- Test component interactions
- Verify data flow between layers
- May use test doubles for external services
- 15-20% of test suite

**What to Test:**
- Repository implementations
- API endpoints and controllers
- Service layer interactions
- Database operations
- External service integrations (with mocks)

**Implementation:**

```typescript
// ✅ GOOD - Integration test for repository
describe('UserRepository', () => {
  let repository: UserRepository;
  let testDatabase: TestDatabase;

  beforeAll(async () => {
    testDatabase = await TestDatabase.start();
    repository = new PostgresUserRepository(testDatabase.connection);
  });

  afterAll(async () => {
    await testDatabase.stop();
  });

  beforeEach(async () => {
    await testDatabase.clear();
  });

  it('should find user by email', async () => {
    const user = new User({ email: 'test@example.com', name: 'Test' });
    await repository.save(user);

    const found = await repository.findByEmail('test@example.com');

    expect(found).not.toBeNull();
    expect(found!.email).toBe('test@example.com');
  });
});

// ✅ GOOD - API integration test
describe('Orders API', () => {
  it('should create order via POST /orders', async () => {
    const orderData = { items: [{ productId: 'p1', quantity: 2 }] };

    const response = await request(app)
      .post('/api/orders')
      .send(orderData)
      .expect(201);

    expect(response.body.id).toBeDefined();
    expect(response.body.status).toBe('CREATED');
  });
});
```

**AI Guidelines:**
- AI should test repository implementations against real databases
- AI should verify API contracts and response formats
- AI should test service-to-service interactions
- AI should use test containers for integration tests
- AI should clean up test data after each test

---

### Layer 3: End-to-End (E2E) Tests

E2E tests verify the complete system from the user's perspective, simulating real user workflows.

**Characteristics:**
- Slow execution (minutes to hours)
- Test complete user journeys
- May include UI automation
- Few in number (5-10% of test suite)
- Highest confidence level

**What to Test:**
- Critical user journeys
- Payment flows
- Authentication flows
- Cross-feature workflows
- Third-party integrations

**Implementation:**

```typescript
// ✅ GOOD - E2E test for checkout flow
describe('Checkout Flow E2E', () => {
  it('should complete purchase from cart to confirmation', async () => {
    const page = await browser.newPage();

    // 1. Login
    await page.goto('/login');
    await page.fill('#email', 'user@test.com');
    await page.fill('#password', 'password123');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');

    // 2. Add items to cart
    await page.goto('/products');
    await page.click('[data-product-id="p1"] .add-to-cart');
    await page.waitForSelector('.cart-badge');

    // 3. Checkout
    await page.click('.cart-icon');
    await page.click('.checkout-button');
    await page.fill('#card_number', '4242424242424242');
    await page.fill('#expiry', '12/25');
    await page.fill('#cvc', '123');
    await page.click('#pay-now');

    // 4. Verify confirmation
    await page.waitForSelector('.order-confirmation');
    const orderId = await page.textContent('.order-id');

    // 5. Verify via API
    const orderResponse = await request(app)
      .get(`/api/orders/${orderId}`)
      .set('Authorization', `Bearer ${await getAuthToken()}`);

    expect(orderResponse.body.status).toBe('COMPLETED');
  });
});
```

**AI Guidelines:**
- AI should focus on critical business paths only
- AI should combine API for setup, UI for user flows
- AI should use selectors that are resilient to UI changes
- AI should implement proper test isolation
- AI should prioritize test execution order

---

## Testing Ratios

| Layer | Percentage | Characteristics |
|-------|------------|-----------------|
| Unit | 70-80% | Fast, isolated, many |
| Integration | 15-20% | Moderate speed, component tests |
| E2E | 5-10% | Slow, complete flows, few |

**Example Distribution for a Feature:**

```
Unit Tests:          50-60 tests
Integration Tests:   10-15 tests
E2E Tests:           3-5 tests
```

---

## Anti-Patterns to Avoid

### The Ice Cream Cone Anti-Pattern

```
🔺🔺🔺
🔺🔺🔺🔺🔺
🔺🔺🔺🔺🔺🔺🔺
🔺🔺🔺🔺🔺🔺🔺🔺🔺  <- Too many E2E tests!
🔀🔀🔀🔀🔀🔀🔀🔀🔀🔀
🔀🔀🔀🔀🔀🔀🔀🔀🔀🔀🔀🔀
```

**Problems with inverted pyramid:**
- Slow test suite execution
- Flaky tests that break frequently
- High maintenance overhead
- Long feedback loops
- Low confidence in test results

### Test Overlap

Testing the same behavior at multiple levels:

```typescript
// ❌ BAD - Same validation tested at all levels
describe('Order Validation - Unit', () => {
  it('should reject order with no items', () => {
    const order = new Order([]);
    expect(() => order.validate()).toThrow();
  });
});

describe('Order API - Integration', () => {
  it('should reject order with no items', async () => {
    const response = await request(app)
      .post('/api/orders')
      .send({ items: [] });
    expect(response.status).toBe(400);
  });
});

describe('Checkout E2E - UI', () => {
  it('should not allow checkout with empty cart', async () => {
    await page.goto('/checkout');
    await page.click('#place-order');
    expect(await page.textContent('.error')).toContain('Items required');
  });
});
```

**Solution:** Push tests down to the appropriate level

---

## AI-Assisted Testing

### AI for Test Generation

AI can automatically generate test cases:

```typescript
// AI-generated unit tests from code analysis
describe('calculateDiscount', () => {
  it('should return 0 for amount less than 100', () => {
    expect(calculateDiscount(50)).toBe(0);
  });

  it('should return 10% discount for amount 100-499', () => {
    expect(calculateDiscount(100)).toBe(10);
    expect(calculateDiscount(250)).toBe(25);
    expect(calculateDiscount(499)).toBe(49.9);
  });

  it('should return 20% discount for amount 500+', () => {
    expect(calculateDiscount(500)).toBe(100);
    expect(calculateDiscount(1000)).toBe(200);
  });

  it('should throw for negative amounts', () => {
    expect(() => calculateDiscount(-10)).toThrow();
  });
});
```

**AI Guidelines for Test Generation:**
- AI should generate tests for all code paths
- AI should include edge cases and boundary conditions
- AI should use property-based testing for pure functions
- AI should follow existing test patterns in the codebase

### AI for Test Maintenance

AI can help update tests when code changes:

```typescript
// When Order class changes, AI can:
// 1. Detect affected tests
// 2. Update test setup
// 3. Fix assertions
// 4. Suggest new test cases
```

**AI Guidelines for Test Maintenance:**
- AI should analyze impact of code changes on tests
- AI should auto-fix breaking tests when safe
- AI should flag tests that need human review
- AI should suggest test improvements

---

## Test Organization

### Directory Structure

```
src/
├── features/
│   └── orders/
│       ├── domain/
│       │   └── __tests__/
│       │       ├── order.aggregate.test.ts
│       │       └── money.vo.test.ts
│       ├── usecase/
│       │   └── __tests__/
│       │       ├── create-order.service.test.ts
│       │       └── order-mapper.test.ts
│       └── adapter/
│           └── __tests__/
│               ├── order.controller.test.ts
│               └── order.repository.test.ts
├── e2e/
│   ├── checkout.flow.test.ts
│   ├── order.management.test.ts
│   └── fixtures/
└── test/
    ├── fixtures/
    │   ├── test-data.ts
    │   └── mock-services.ts
    └── utils/
        ├── test-helpers.ts
        └── assertion-helpers.ts
```

### Naming Conventions

| Pattern | Example | Purpose |
|---------|---------|---------|
| `{feature}.test.ts` | `order.test.ts` | Unit tests |
| `{feature}.integration.test.ts` | `order.integration.test.ts` | Integration tests |
| `{feature}.e2e.test.ts` | `checkout.e2e.test.ts` | E2E tests |

---

## Best Practices for AI-Generated Tests

1. **Descriptive Test Names**
   ```typescript
   // ✅ GOOD
   it('should return the order total including tax and shipping', () => { });
   
   // ❌ BAD
   it('should calculate total', () => { });
   ```

2. **Single Assertion Per Test (when possible)**
   ```typescript
   // ✅ GOOD - One clear assertion
   it('should calculate order total from line items', () => {
     const order = new Order([item1, item2]);
     expect(order.total).toBe(150);
   });
   ```

3. **Proper Test Setup/Teardown**
   ```typescript
   beforeEach(async () => {
     await testDatabase.clear();
     await seedTestData();
   });
   ```

4. **Fast Tests**
   - Mock slow external dependencies
   - Use in-memory databases for tests
   - Parallelize test execution

5. **Independent Tests**
   - No test should depend on another test's state
   - Each test should setup its own data
   - Tests should be runnable in any order

---

## Test Coverage Guidelines

| Layer | Target Coverage | Metric |
|-------|-----------------|--------|
| Domain | 90%+ | Line/Branch coverage |
| Use Case | 80%+ | Line coverage |
| Adapter | 70%+ | Integration points |
| E2E | Critical paths | Feature coverage |

**AI Guidelines:**
- AI should flag uncovered critical paths
- AI should prioritize tests for uncovered code
- AI should avoid coverage for trivial code (getters/setters)

---

## Tools by Layer

| Layer | Tools |
|-------|-------|
| Unit | Jest, Vitest, Mocha, JUnit, pytest |
| Integration | Supertest, Playwright, REST-assured, Pact |
| E2E | Playwright, Cypress, Selenium, Puppeteer |
| Contract | Pact, Spring Cloud Contract |

---

## References and Further Reading

1. Cohn, Mike. "Succeeding with Agile: Software Development Using Scrum." 2009.
2. Fowler, Martin. "The Practical Test Pyramid." https://martinfowler.com/articles/practical-test-pyramid.html
3. "The Test Pyramid 2.0: AI-assisted testing across the pyramid." Frontiers in Artificial Intelligence, 2025.
4. AI Coding Exercise Repository - Testing Strategy Analysis (internal reference)

---

## Testing Conventions (from Project Constitution)

### Test Organization Principles

| Convention | Rule | Example |
|------------|------|---------|
| Package Structure | Test packages mirror main | `domain.entity` → `domain.entity.test` |
| Unit tests per class | One test class per domain class | `Calculator` → `CalculatorTest` |
| Integration tests | For use case layer | `CreateOrderUseCaseIntegrationTest` |
| Naming convention | `XxxTest` for unit tests | `CalculatorTest` |

### Test Structure Standards

```java
// Arrange-Act-Assert Pattern
@Test
void should_add_two_positive_numbers_correctly() {
    // Arrange
    Calculator calculator = new Calculator();

    // Act
    int result = calculator.add(2, 3);

    // Assert
    assertEquals(5, result);
}
```

### Test Naming Guidelines

| Pattern | Purpose | Example |
|---------|---------|---------|
| Method names | Describe behavior with underscores | `add_a_project_with_duplicated_name_has_no_effect()` |
| Unit test | Test single behavior | `CalculatorTest` |
| Integration test | Verify component interaction | `CreateProductUseCaseIntegrationTest` |

### Test Coverage Guidelines

| Layer | Target Coverage | Focus |
|-------|-----------------|-------|
| Domain (Entities) | 90%+ | All business logic |
| Use Cases | 80%+ | Application rules |
| Adapters | 70%+ | Integration points |

---

## AI-Assisted Testing Patterns

### BDD Testing Pattern

```typescript
describe("CreateProductUseCase", () => {
  let feature: Feature;
  let useCase: CreateProductUseCase;

  beforeAll(() => {
    feature = new Feature("Create Product Use Case");
    feature.initialize();
  });

  it("should create product with valid input", () => {
    feature.newScenario("Successfully create a product with valid input")
      .given("valid product creation input", env => {
        const input = CreateProductInput.create();
        input.productId = "prod-001";
        input.name = "Test Product";
        input.userId = "user-001";
        env.set("input", input);
      })
      .when("the use case is executed", env => {
        const input = env.get<CreateProductInput>("input");
        const output = useCase.execute(input);
        env.set("output", output);
      })
      .then("the product should be created successfully", env => {
        const output = env.get<CqrsOutput>("output");
        expect(output.isSuccessful()).toBe(true);
      })
      .execute();
  });
});
```

### Dual Profile Testing

Use profile-based testing to validate different execution contexts:

| Profile | Purpose | Use Case |
|---------|---------|----------|
| `test-inmemory` | Fast unit tests | Local development, CI |
| `test-outbox` | Integration tests | Event sourcing validation |

```xml
<!-- pom.xml - Test Profiles -->
<profiles>
    <profile>
        <id>test-inmemory</id>
        <includes>
            <include>**/InMemoryTestSuite.java</include>
        </includes>
    </profile>

    <profile>
        <id>test-outbox</id>
        <includes>
            <include>**/OutboxTestSuite.java</include>
        </includes>
    </profile>
</profiles>
```

### Mutation Testing with PIT

Validate test quality by introducing code mutations:

```xml
<!-- PIT Configuration -->
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <configuration>
        <targetClasses>
            <param>com.example.domain.*</param>
        </targetClasses>
        <excludedClasses>
            <param>*Events</param>
            <param>*Events$*</param>
        </excludedClasses>
    </configuration>
</plugin>
```

---

## AI Testing Anti-Patterns

### Common AI Testing Mistakes

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Testing implementation details | Brittle tests | Test behavior, not internals |
| Over-mocked tests | Low confidence | Test real interactions where possible |
| Large test setup | Hard to maintain | Extract helper methods, use builders |
| Inconsistent naming | Hard to understand | Use descriptive names with underscores |
| Duplicate test code | Wasted effort | Extract to parameterized tests |

### AI-Specific Testing Guidelines

1. **Generate tests for all code paths** - Ensure complete coverage
2. **Include edge cases** - Boundary conditions, null values, exceptions
3. **Use property-based testing** - For pure functions and algorithms
4. **Follow existing patterns** - Match codebase conventions
5. **Validate test isolation** - No test should depend on another
