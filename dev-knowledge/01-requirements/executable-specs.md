# Executable Specifications

## Core Concept

Executable specifications are requirements or design documents that can be run as automated tests. Rather than static text that can become outdated or ambiguous, executable specifications serve as both documentation and validation, ensuring that the system behaves exactly as specified and that the documentation always reflects current behavior.

The concept, championed by Gojko Adzic and others in the specification by example community, transforms documentation from a secondary artifact into a primary deliverable that drives development. When specifications are executable, they become a single source of truth that cannot drift from actual behavior without detection.

Executable specifications solve several persistent problems in software development: requirements documents become outdated as soon as they're written, test cases and specifications diverge over time, and it's difficult to verify that documentation accurately describes the system. By making specifications executable, we create living documents that are always current, always accurate, and always verified.

The key insight is that the same examples used to illustrate requirements can be run as automated tests. This creates a tight feedback loop where specification and implementation evolve together, with the specifications always accurately describing what the system does.

---

## The Specification by Example Process

```
┌─────────────────────────────────────────────────────────────────────────┐
│              SPECIFICATION BY EXAMPLE WORKFLOW                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│  │   DERIVE    │───►│  EXAMPLES   │───►│  AUTOMATE   │                 │
│  │  EXAMPLES   │    │             │    │             │                 │
│  │             │    │  Concrete   │    │  Write      │                 │
│  │  Identify   │    │  scenarios  │    │  executable │                 │
│  │  key        │    │  that       │    │  tests from │                 │
│  │  examples   │    │  illustrate │    │  examples   │                 │
│  │             │    │  behavior   │    │             │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│         │                  │                  │                         │
│         ▼                  ▼                  ▼                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│   │  BUSINESS   │    │  DISCUSS    │    │   EXECUTE   │                 │
│   │  OUTCOMES   │    │  & REFINE   │    │   & VALIDATE│                 │
│   │             │    │             │    │             │                 │
│   │  Define     │    │  Collaborate│    │  Run tests  │                 │
│   │  measurable │    │  to clarify │    │  to verify  │                 │
│   │  outcomes   │    │  and        │    │  behavior   │                 │
│   │             │    │  improve    │    │             │                 │
│   │             │    │  examples   │    │             │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│                                           │                             │
│                                           ▼                             │
│                                    ┌─────────────┐                      │
│                                    │  MAINTAIN   │                      │
│                                    │             │                      │
│                                    │  Specifications│                   │
│                                    │  stay current│                   │
│                                    │  with code   │                   │
│                                    └─────────────┘                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Activities

1. **Derive Examples from Business Outcomes**: Work backwards from what the business wants to achieve to identify concrete examples that demonstrate success.

2. **Collaborate on Examples**: Involve business stakeholders, developers, and testers in creating examples to ensure completeness and accuracy.

3. **Automate Examples**: Transform examples into executable specifications that can be run as tests.

4. **Validate Continuously**: Run specifications against the system to verify behavior and detect regressions.

5. **Maintain as Living Documentation**: Keep specifications updated as the system evolves, ensuring they remain accurate and useful.

---

## Executable Specifications with Gherkin

```gherkin
# features/checkout/discount-calculation.feature
Feature: Discount Calculation
  As a store owner
  I want to apply discounts to orders
  So that I can run promotions and increase sales

  Background:
    Given the store has the following discount rules:
      | rule_name     | discount_percent | minimum_order | code    |
      | Summer Sale   | 15               | 50.00         | SUMMER  |
      | VIP Customer  | 20               | 0             | <none>  |
      | First Order   | 10               | 25.00         | FIRST10 |

  Rule: Percentage discounts are applied to order subtotal
    Example: 15% discount on $100 order
      Given the cart contains:
        | product  | quantity | price |
        | Widget A | 2        | 25.00 |
        | Widget B | 2        | 25.00 |
      When the customer applies the "SUMMER" discount code
      Then the discount amount should be $15.00
      And the final total should be $85.00

    Example: Discount code is not case sensitive
      Given the cart contains:
        | product  | quantity | price |
        | Widget A | 1        | 50.00 |
      When the customer applies the "summer" discount code
      Then the discount amount should be $7.50

  Rule: Minimum order requirements must be met
    Example: Order below minimum does not qualify for discount
      Given the cart contains:
        | product  | quantity | price |
        | Widget A | 1        | 30.00 |
      When the customer applies the "SUMMER" discount code
      Then an error message "Order minimum of $50.00 not met" should be shown
      And the discount should not be applied

    Example: Order at minimum qualifies for discount
      Given the cart contains:
        | product  | quantity | price |
        | Widget A | 2        | 25.00 |
      When the customer applies the "SUMMER" discount code
      Then the discount should be applied

  Rule: VIP customers get automatic discounts without code
    Example: VIP customer receives automatic 20% discount
      Given the customer is a VIP member
      And the cart contains:
        | product  | quantity | price |
        | Widget A | 1        | 100.00 |
      When the customer completes checkout
      Then the discount amount should be $20.00
      And the final total should be $80.00

  Rule: Only one discount can be applied per order
    Example: Attempting to apply multiple discounts
      Given the cart contains:
        | product  | quantity | price |
        | Widget A | 1        | 100.00 |
      And the customer is a VIP member
      When the customer applies the "FIRST10" discount code
      Then an error message "Only one discount per order is allowed" should be shown
```

---

## Executable Specifications in TypeScript

```typescript
// features/checkout/step-definitions/discount.steps.ts
import { Given, When, Then, And } from '@cucumber/cucumber';
import { expect } from 'chai';
import { ShoppingCart } from '../../../src/checkout/shopping-cart';
import { Product } from '../../../src/checkout/product';
import { Customer } from '../../../src/checkout/customer';

let cart: ShoppingCart;
let discountError: Error | null = null;

Given('the store has the following discount rules:', function (dataTable: any) {
  const rules = dataTable.hashes();
  // Setup discount rules in the system
  rules.forEach((rule: any) => {
    if (rule.code !== '<none>') {
      DiscountRuleRepository.addPromoCode(rule.code, {
        percent: parseFloat(rule.discount_percent),
        minimumOrder: parseFloat(rule.minimum_order),
      });
    }
  });
});

Given('the cart contains:', function (dataTable: any) {
  cart = new ShoppingCart();
  const items = dataTable.hashes();
  items.forEach((item: any) => {
    const product = new Product(item.product, parseFloat(item.price));
    cart.addItem(product, parseInt(item.quantity));
  });
});

When('the customer applies the {string} discount code', function (code: string) {
  try {
    cart.applyPromoCode(code);
  } catch (error) {
    discountError = error as Error;
  }
});

Then('the discount amount should be ${float}', function (expectedAmount: number) {
  expect(cart.getDiscountAmount()).to.be.closeTo(expectedAmount, 0.01);
});

Then('the final total should be ${float}', function (expectedTotal: number) {
  expect(cart.getFinalTotal()).to.be.closeTo(expectedTotal, 0.01);
});

Then('an error message {string} should be shown', function (expectedMessage: string) {
  expect(discountError).to.not.be.null;
  expect(discountError?.message).to.equal(expectedMessage);
});

And('the discount should not be applied', function () {
  expect(cart.getDiscountAmount()).to.equal(0);
});

Given('the customer is a VIP member', function () {
  const customer = new Customer('VIP123', 'vip');
  cart.setCustomer(customer);
});

When('the customer completes checkout', function () {
  // Checkout logic
});

Then('the discount should be applied', function () {
  expect(cart.getDiscountAmount()).to.be.greaterThan(0);
});

// src/checkout/discount-rule-repository.ts
class DiscountRuleRepository {
  private static promoCodes: Map<string, DiscountRule> = new Map();

  static addPromoCode(code: string, rule: DiscountRule): void {
    this.promoCodes.set(code.toUpperCase(), rule);
  }

  static getPromoCode(code: string): DiscountRule | undefined {
    return this.promoCodes.get(code.toUpperCase());
  }

  static clear(): void {
    this.promoCodes.clear();
  }
}

interface DiscountRule {
  percent: number;
  minimumOrder: number;
}

// src/checkout/shopping-cart.ts
class ShoppingCart {
  private items: { product: Product; quantity: number }[] = [];
  private discountAmount: number = 0;
  private customer: Customer | null = null;

  addItem(product: Product, quantity: number): void {
    this.items.push({ product, quantity });
  }

  applyPromoCode(code: string): void {
    const rule = DiscountRuleRepository.getPromoCode(code);
    if (!rule) {
      throw new Error('Invalid discount code');
    }

    const subtotal = this.getSubtotal();
    if (subtotal < rule.minimumOrder) {
      throw new Error(`Order minimum of $${rule.minimumOrder.toFixed(2)} not met`);
    }

    this.discountAmount = subtotal * (rule.percent / 100);
  }

  getSubtotal(): number {
    return this.items.reduce(
      (sum, item) => sum + item.product.price * item.quantity,
      0
    );
  }

  getDiscountAmount(): number {
    return this.discountAmount;
  }

  getFinalTotal(): number {
    return this.getSubtotal() - this.discountAmount;
  }

  setCustomer(customer: Customer): void {
    this.customer = customer;
  }
}
```

---

## Living Documentation Reports

```typescript
// living-documentation/generate-report.ts
import * as fs from 'fs';
import * as path from 'path';

interface SpecificationReport {
  feature: string;
  totalScenarios: number;
  passingScenarios: number;
  failingScenarios: number;
  lastExecuted: Date;
  owner: string;
  businessValue: string;
}

function generateLivingDocumentationReport(): string {
  const report: SpecificationReport[] = [];

  // Load feature files
  const featuresPath = './features';
  const featureFiles = findAllFeatureFiles(featuresPath);

  for (const file of featureFiles) {
    const feature = parseFeatureFile(file);
    const testResults = getTestResultsForFeature(feature.name);

    report.push({
      feature: feature.name,
      totalScenarios: feature.scenarios.length,
      passingScenarios: testResults.passing,
      failingScenarios: testResults.failing,
      lastExecuted: new Date(),
      owner: feature.tags.find(t => t.startsWith('@owner:'))?.replace('@owner:', '') || 'Unknown',
      businessValue: feature.description.split('\n')[0],
    });
  }

  return renderReport(report);
}

function renderReport(report: SpecificationReport[]): string {
  let markdown = `# Living Documentation Report\n\n`;
  markdown += `Generated: ${new Date().toISOString()}\n\n`;
  markdown += `## Executive Summary\n\n`;
  markdown += `| Metric | Value |\n`;
  markdown += `|--------|-------|\n`;
  markdown += `| Total Features | ${report.length} |\n`;
  markdown += `| Total Scenarios | ${report.reduce((sum, r) => sum + r.totalScenarios, 0)} |\n`;
  markdown += `| Passing Scenarios | ${report.reduce((sum, r) => sum + r.passingScenarios, 0)} |\n`;
  markdown += `| Failing Scenarios | ${report.reduce((sum, r) => sum + r.failingScenarios, 0)} |\n\n`;

  markdown += `## Feature Status\n\n`;
  markdown += `| Feature | Owner | Scenarios | Status |\n`;
  markdown += `|---------|-------|-----------|--------|\n`;
  
  for (const r of report) {
    const status = r.failingScenarios === 0 ? '✅' : r.passingScenarios === 0 ? '❌' : '⚠️';
    markdown += `| ${r.feature} | ${r.owner} | ${r.passingScenarios}/${r.totalScenarios} | ${status} |\n`;
  }

  markdown += `\n## Feature Details\n\n`;
  
  for (const r of report) {
    markdown += `### ${r.feature}\n\n`;
    markdown += `**Owner:** ${r.owner}\n\n`;
    markdown += `**Business Value:** ${r.businessValue}\n\n`;
    markdown += `**Last Executed:** ${r.lastExecuted.toISOString()}\n\n`;
    
    if (r.failingScenarios > 0) {
      markdown += `**⚠️ Failing Scenarios:** ${r.failingScenarios}\n\n`;
    }
  }

  return markdown;
}

function findAllFeatureFiles(dir: string): string[] {
  const results: string[] = [];
  const entries = fs.readdirSync(dir, { withFileTypes: true });
  
  for (const entry of entries) {
    const fullPath = path.join(dir, entry.name);
    if (entry.isDirectory()) {
      results.push(...findAllFeatureFiles(fullPath));
    } else if (entry.name.endsWith('.feature')) {
      results.push(fullPath);
    }
  }
  
  return results;
}

// Execute and output report
const report = generateLivingDocumentationReport();
fs.writeFileSync('./docs/living-documentation.md', report);
console.log('Living documentation generated: docs/living-documentation.md');
```

---

## Executable API Specifications with OpenAPI

```yaml
# openapi/executable-spec.yaml
openapi: 3.0.3
info:
  title: E-Commerce API
  version: 1.0.0
  description: |
    This is an executable specification for the E-Commerce API.
    All endpoints are validated against actual behavior.

paths:
  /api/orders:
    post:
      summary: Create a new order
      description: Creates a new order from the provided cart items
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderResponse'
          x-scenario: order_created_successfully
        '400':
          description: Invalid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          x-scenario: invalid_order_request
        '401':
          description: Unauthorized
          x-scenario: unauthenticated_request
        '409':
          description: Conflict - insufficient inventory
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/InventoryError'
          x-scenario: insufficient_inventory

components:
  schemas:
    CreateOrderRequest:
      type: object
      required:
        - customerId
        - items
      properties:
        customerId:
          type: string
          description: Unique identifier of the customer
          x-example: "CUST-123"
        items:
          type: array
          items:
            $ref: '#/components/schemas/OrderItem'
          minItems: 1
        shippingAddress:
          $ref: '#/components/schemas/Address'
      x-scenarios:
        - valid_order
        - empty_order
        - missing_customer

    OrderItem:
      type: object
      required:
        - productId
        - quantity
      properties:
        productId:
          type: string
          x-example: "PROD-456"
        quantity:
          type: integer
          minimum: 1
          x-example: 2

    OrderResponse:
      type: object
      properties:
        orderId:
          type: string
          x-example: "ORD-789"
        status:
          type: string
          enum:
            - pending
            - confirmed
            - shipped
            - delivered
        total:
          type: number
          format: decimal
        createdAt:
          type: string
          format: date-time

# Execute these specifications as tests
x-test-scenarios:
  - name: order_created_successfully
    request:
      method: POST
      path: /api/orders
      body:
        customerId: "CUST-123"
        items:
          - productId: "PROD-456"
            quantity: 2
    expected:
      status: 201
      body:
        orderId: exists
        status: "confirmed"

  - name: invalid_order_request
    request:
      method: POST
      path: /api/orders
      body:
        customerId: "CUST-123"
        items: []
    expected:
      status: 400
```

---

## Pact: Contract Testing as Executable Specification

```typescript
// consumer.test.ts
import { Pact, Matchers } from '@pact-foundation/pact';
import { like, eachLike } from '@pact-foundation/pact/dsl/matchers';

describe('Order Service Consumer', () => {
  const provider = new Pact({
    consumer: 'order-service-web',
    provider: 'inventory-service',
    port: 1234,
  });

  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());

  describe('when checking inventory', () => {
    it('can process an availability response', async () => {
      provider.addInteraction({
        state: 'inventory has products',
        uponReceiving: 'a request for product availability',
        withRequest: {
          method: 'GET',
          path: '/api/inventory/check',
          query: { productIds: 'PROD-001,PROD-002' },
        },
        willRespondWith: {
          status: 200,
          headers: {
            'Content-Type': 'application/json',
          },
          body: {
            results: eachLike({
              productId: Matchers.string('PROD-001'),
              available: Matchers.boolean(true),
              quantity: Matchers.integer(10),
            }),
          },
        },
      });

      const client = new InventoryClient('http://localhost:1234');
      const result = await client.checkAvailability(['PROD-001', 'PROD-002']);

      expect(result).toEqual({
        results: [{ productId: 'PROD-001', available: true, quantity: 10 }],
      });
    });
  });
});

// provider.verification.ts
import { Verifier } from '@pact-foundation/pact';

describe('Inventory Service Verification', () => {
  it('validates the contract with the expectations of the consumer', async () => {
    const verifier = new Verifier({
      provider: 'inventory-service',
      providerBaseUrl: 'http://localhost:3000',
      pactUrls: ['./pacts/order-service-web-inventory-service.json'],
    });

    await verifier.verifyProvider();
  });
});
```

---

## Executable Specifications Best Practices

### 1. Start with Business Outcomes
Begin by identifying what the business wants to achieve, then derive specifications that validate those outcomes.

### 2. Use Real Examples
Avoid abstract examples in favor of real scenarios that represent actual use cases and edge cases.

### 3. Keep Specifications Executable
Every example should be automatable. Avoid descriptions that can't be turned into tests.

### 4. Collaborate on Specifications
Involve business stakeholders in creating and reviewing specifications to ensure accuracy and completeness.

### 5. Use Ubiquitous Language
Apply domain terminology consistently across specifications, bridging the gap between business and technical language.

### 6. Maintain Single Source of Truth
Let specifications drive development, not the other way around. The specifications should be authoritative.

### 7. Generate Documentation
Use the executable specifications to generate human-readable documentation that stays in sync with code.

### 8. Run Continuously
Execute specifications continuously to detect regressions and ensure specifications remain valid.

### 9. Version Specifications
Treat specifications as version-controlled artifacts that evolve with the system.

### 10. Use for Communication
Specifications serve as a communication medium between technical and business stakeholders.

---

## Executable Specifications Anti-Patterns

### 1. Testing Implementation
Writing specifications that test how something is implemented rather than what it does, making specifications brittle.

### 2. Over-Abstracted Scenarios
Creating scenarios so abstract they don't test anything meaningful, producing meaningless passing tests.

### 3. Missing Assertions
Writing scenarios that describe actions but not expected outcomes, failing to validate behavior.

### 4. Hard-Coded Values Everywhere
Using arbitrary values instead of meaningful examples that illustrate the domain.

### 5. No Business Involvement
Creating specifications without business stakeholder input, resulting in technically correct but business-irrelevant tests.

### 6. Stale Specifications
Allowing specifications to drift from actual system behavior, defeating their purpose.

### 7. Testing Third-Party Code
Writing specifications for external services or libraries instead of your integration points.

### 8. Excessive Mocking
Mocking so much that the specifications don't test real behavior, only mocked interactions.

### 9. No Maintenance Plan
Creating specifications without a plan to maintain them, leading to accumulated technical debt.

### 10. Treating Specifications as Tests Only
Using specifications solely for testing without leveraging their value as documentation.

---

## Executable Specifications in AI-Assisted Development

When working with AI assistants for executable specifications, follow these practices:

### 1. Generate Initial Specifications
Provide domain context and ask AI to help draft initial Gherkin scenarios or examples.

### 2. Identify Edge Cases
Ask AI to review your specifications and suggest edge cases or scenarios you might have missed.

### 3. Generate Step Definitions
Automatically generate step definition stubs from Gherkin scenarios.

### 4. Validate Completeness
Have AI verify that your specifications cover all the acceptance criteria.

### 5. Generate Documentation
Use AI to transform specifications into various documentation formats.

### 6. Translate to Other Formats
Convert specifications between Gherkin, OpenAPI, JSON Schema, or other formats.

### 7. Maintain Consistency
Use AI to ensure consistent terminology and structure across all specifications.

### 8. Analyze Test Results
Use AI to analyze failing specifications and suggest potential causes.

---

## Tools for Executable Specifications

| Tool | Purpose | Format |
|------|---------|--------|
| **Cucumber** | BDD Framework | Gherkin |
| **SpecFlow** | BDD for .NET | Gherkin |
| **Behave** | BDD for Python | Gherkin |
| **Jest** | JavaScript Testing | Code |
| **Pact** | Contract Testing | JSON/Pact |
| **OpenAPI** | API Specifications | YAML/JSON |
| **Postman/Newman** | API Testing | Collection |
| **Gauge** | Multi-format | Markdown |
| **Concordion** | HTML Specifications | HTML |

---

## References and Further Reading

1. Adzic, Gojko. "Specification by Example: How Successful Teams Deliver the Right Software." Manning, 2011.
2. Wynne, Matt, and Aslak Hellesøy. "The Cucumber Book: Behaviour-Driven Development for Testers and Developers." Pragmatic Programmers, 2012.
3. "Living Documentation." Cucumber Documentation.
4. "Contract Testing with Pact." Pactflow.
5. "OpenAPI Specification." OpenAPI Initiative.
