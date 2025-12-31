# Specification-Driven Development

## Overview

Specification-Driven Development (SDD) is an approach where detailed specifications serve as the primary artifact that drives the entire development process. Unlike traditional "code-first" approaches, SDD emphasizes creating comprehensive, unambiguous specifications before implementation begins. This approach is particularly valuable in AI-assisted development, where clear specifications directly inform AI code generation.

## Core Principles

### 1. Specification as Source of Truth

```markdown
## Specification Hierarchy

┌─────────────────────────────────────┐
│     Business Requirements           │  ← Strategic goals
├─────────────────────────────────────┤
│     Feature Specifications          │  ← What to build
├─────────────────────────────────────┤
│     Technical Specifications        │  ← How it works
├─────────────────────────────────────┤
│     API Contracts                  │  ← Interface definitions
├─────────────────────────────────────┤
│     Test Specifications            │  ← Acceptance criteria
└─────────────────────────────────────┘
```

### 2. Formal Specification Languages

```gherkin
## Gherkin for Behavioral Specifications
Feature: Order Processing System

  Background:
    Given the inventory service is available
    And the payment gateway is operational

  Scenario: Successful order placement
    Given a customer has items in their cart totaling $150
    When the customer submits the order
    Then the inventory should be reserved
    And payment should be processed
    And order confirmation should be sent

  Scenario: Payment failure handling
    Given a customer has a pending order
    When payment is rejected by the gateway
    Then the inventory reservation should be released
    And the customer should be notified
    And the order status should be "payment_failed"
```

```typescript
// TypeScript Interface Specifications
interface OrderProcessingSpec {
  processOrder(input: OrderInput): Promise<OrderResult>;
  
  // Preconditions
  preconditions: {
    inventoryAvailable: (items: OrderItem[]) => Promise<boolean>;
    customerValid: (customerId: string) => Promise<boolean>;
    paymentMethodValid: (payment: PaymentMethod) => boolean;
  };
  
  // Postconditions
  postconditions: {
    onSuccess: (result: OrderResult) => void;
    onFailure: (error: OrderError) => void;
  };
  
  // Invariants
  invariants: {
    orderTotal: (items: OrderItem[], payment: Payment) => boolean;
    orderId: (order: Order) => boolean;
  };
}
```

### 3. Executable Specifications

```python
# Python with behave for executable specifications
from behave import given, when, then
from orders import OrderProcessor

@given("the order processor is initialized")
def step_order_processor_initialized(context):
    context.processor = OrderProcessor(
        inventory_service=context.inventory,
        payment_service=context.payment,
        notification_service=context.notification
    )

@when("the customer submits an order with {item_count} items")
def step_customer_submits_order(context, item_count):
    context.order = context.processor.create_order(
        items=context.items[:int(item_count)],
        customer_id=context.customer_id
    )

@then("the order should be confirmed with status {status}")
def step_order_confirmed_with_status(context, status):
    assert context.order.status == status
    context.processor.audit_log.record(
        event="order_confirmed",
        order_id=context.order.id,
        status=status
    )
```

## SDD Workflow

```
┌────────────────────────────────────────────────────────────────┐
│                    SDD Development Cycle                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   1. REQUIREMENT GATHERING                                      │
│      ↓ Capture business needs and user stories                  │
│   2. SPECIFICATION CREATION                                     │
│      ↓ Write formal, unambiguous specifications                 │
│   3. SPECIFICATION REVIEW                                       │
│      ↓ Stakeholder validation and approval                      │
│   4. AI-Assisted IMPLEMENTATION                                 │
│      ↓ Generate code from specifications                        │
│   5. AUTOMATED VERIFICATION                                     │
│      ↓ Execute specifications as tests                          │
│   6. ITERATION                                                  │
│      ↓ Refine specifications based on feedback                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

## Specification Types

### API Specifications (OpenAPI)

```yaml
openapi: 3.1.0
info:
  title: Order Processing API
  version: 1.0.0
paths:
  /orders:
    post:
      summary: Create a new order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/OrderInput'
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderResponse'
        '400':
          $ref: '#/components/responses/ValidationError'
        '409':
          $ref: '#/components/responses/ConflictError'

components:
  schemas:
    OrderInput:
      type: object
      required:
        - customerId
        - items
      properties:
        customerId:
          type: string
          format: uuid
          description: Unique customer identifier
        items:
          type: array
          minItems: 1
          maxItems: 100
          items:
            $ref: '#/components/schemas/OrderItem'
      additionalProperties: false
```

### Data Contracts

```typescript
// TypeScript Data Contract Specifications
interface UserRegistrationContract {
  validate(input: unknown): ValidationResult<UserRegistrationData>;
  
  constraints: {
    email: {
      format: 'email';
      maxLength: 254;
      pattern: '/^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$/';
    };
    password: {
      minLength: 8;
      maxLength: 128;
      requires: ['uppercase', 'lowercase', 'digit', 'special'];
    };
    name: {
      minLength: 1;
      maxLength: 100;
      pattern: '/^[a-zA-Z\\s\\-\\'\']+$/';
    };
  };
  
  transformation: {
    trim: ['email', 'name'];
    lowercase: ['email'];
    hashPassword: ['password'];
  };
}
```

### Behavioral Specifications

```gherkin
Feature: Shopping Cart Checkout Process

  Background:
    Given the shopping cart contains at least one item
    And the user is authenticated
    And the shipping address is validated

  Rule: Complete checkout requires payment method

    Example: Checkout without payment method
      Given the cart is ready for checkout
      When the user attempts to complete checkout
      Then the system should show an error message
      And the error should indicate payment is required
      And the checkout should not be processed

    Example: Checkout with valid payment method
      Given the cart is ready for checkout
      And a valid payment method is selected
      When the user clicks "Complete Checkout"
      Then the order should be created
      And the payment should be processed
      And the inventory should be reserved

  Rule: Inventory availability affects checkout

    Example: All items available
      Given all cart items are in stock
      When the user completes checkout
      Then the order should be created with status "confirmed"

    Example: Some items out of stock
      Given some cart items are out of stock
      When the user completes checkout
      Then the order should be created with status "partial"
      And unavailable items should be listed
```

## AI Integration with Specifications

### Prompt Engineering for SDD

```markdown
## AI Specification Generation Prompt Template

### Context
[Describe the system being built and its purpose]

### Input Specifications
1. **Data Structures**:
   - [List entities with their properties and types]
   
2. **Business Rules**:
   - [List validation rules and constraints]
   
3. **Workflows**:
   - [Describe the main user journeys]

### Output Requirements
Generate:
1. OpenAPI specification (YAML)
2. TypeScript interfaces
3. Gherkin feature files with scenarios
4. Validation function signatures

### Constraints
- Use [TypeScript/Python/Java]
- Follow [project conventions]
- Include error handling for all edge cases
- Add JSDoc/Docstring documentation
```

### Code Generation from Specifications

```python
# Specification-driven code generator
class CodeGenerator:
    def generate_from_spec(self, spec: APISpec) -> GeneratedCode:
        """Generate implementation from OpenAPI specification."""
        
        models = self._generate_models(spec.components.schemas)
        endpoints = self._generate_endpoints(spec.paths)
        validators = self._generate_validators(spec.components.schemas)
        tests = self._generate_tests(spec)
        
        return GeneratedCode(
            models=models,
            endpoints=endpoints,
            validators=validators,
            tests=tests
        )
    
    def _generate_models(self, schemas: dict) -> list[ModelFile]:
        """Generate data models from schemas."""
        return [
            self._typescript_model(name, schema)
            for name, schema in schemas.items()
        ]
```

## Benefits of Specification-Driven Development

| Benefit | Description |
|---------|-------------|
| **Clarity** | Specifications eliminate ambiguity |
| **Testability** | Executable specs serve as tests |
| **AI-Compatibility** | Clear specs = better AI code generation |
| **Documentation** | Specifications are living docs |
| **Traceability** | Requirements map to code |
| **Change Management** | Impact analysis through specs |

## Challenges and Solutions

### Challenge: Keeping Specifications Updated

```markdown
## Solution: Living Specifications

1. **Version Control**: Treat specs like code
2. **CI/CD Integration**: Validate specs in pipeline
3. **Automated Sync**: Generate specs from code annotations
4. **Regular Reviews**: Include spec review in ceremonies
```

### Challenge: Balancing Detail vs. Flexibility

```markdown
## Guidelines for Specification Granularity

**Be Specific About:**
- Input/output formats and constraints
- Error conditions and messages
- Business rules and validation logic
- API endpoints and HTTP methods

**Be Flexible About:**
- Implementation details
- Internal architecture
- Performance optimizations (within bounds)
- Error handling mechanisms (within contracts)
```

## Tools for Specification-Driven Development

| Category | Tools |
|----------|-------|
| API Design | OpenAPI/Swagger, GraphQL Schema |
| Behavioral Specs | Gherkin, Cucumber, SpecFlow |
| Data Contracts | JSON Schema, Protocol Buffers, Avro |
| Documentation | Slate, ReDoc, Docusaurus |
| Code Generation | OpenAPI Generator, Swagger Codegen |
| Validation | Ajv, Yup, Zod, io-ts |

## SDD in AI-Assisted Teams

```markdown
## Workflow for AI-Assisted SDD

### Sprint Planning
1. PO writes feature specifications
2. Team reviews and clarifies
3. AI generates skeleton code and tests
4. Developers implement and refine

### Daily Development
1. Developer reviews specs
2. AI generates initial implementation
3. Developer validates against specs
4. Developer adds edge case handling
5. Tests validate spec compliance

### Definition of Done
- [ ] All specifications have passing tests
- [ ] Code matches specification contracts
- [ ] Documentation reflects specifications
- [ ] Stakeholders approve specifications
```

## Best Practices

1. **Start with Examples**: Concrete examples clarify abstract requirements

2. **Use Formal Notation**: Gherkin, OpenAPI, or TypeScript interfaces

3. **Make Specifications Executable**: Link specs to automated tests

4. **Version Everything**: Apply Git workflows to specifications

5. **Automate Generation**: Generate boilerplate from specs

6. **Review Specifications**: Include spec review in pull requests

7. **Keep AI in Mind**: Structure specs for optimal AI consumption

## References

- "Specification by Example" by Gojko Adzic
- "Domain-Driven Design" by Eric Evans
- OpenAPI Specification (https://spec.openapis.org/)
- Cucumber Documentation (https://cucumber.io/)
