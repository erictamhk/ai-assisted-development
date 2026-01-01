# Requirements and Specification in Software Engineering

## Overview

Requirements and specification documents form the foundation of successful software projects. They define **what** the system should do, **how** it should behave, and **why** certain decisions were made. In the context of AI-assisted development, clear requirements become even more critical as they serve as the primary input for AI coding assistants.

## Types of Requirements

### Functional Requirements

Functional requirements describe the specific behaviors and features a system must exhibit:

```gherkin
Feature: User Authentication

  Scenario: Successful login with valid credentials
    Given the user is on the login page
    When the user enters "valid@email.com" and "correctpassword"
    Then the user should be redirected to the dashboard
    And the user should see their name displayed

  Scenario: Login failure with invalid credentials
    Given the user is on the login page
    When the user enters "invalid@email.com" and "wrongpassword"
    Then the user should see an error message
    And the user should remain on the login page
```

### Non-Functional Requirements

Non-functional requirements define quality attributes:

| Category | Examples | Metrics |
|----------|----------|---------|
| Performance | Response time, throughput | < 200ms response time |
| Scalability | Load handling, growth | Support 10,000 concurrent users |
| Security | Authentication, data protection | 256-bit encryption |
| Availability | Uptime, fault tolerance | 99.9% uptime (SLA) |
| Maintainability | Code quality, documentation | 80% test coverage |

### Domain Requirements

Domain requirements capture business-specific rules:

```typescript
// Example: E-commerce pricing rules
interface PricingRule {
  apply(item: CartItem): Money;
  priority: number;
  conditions: RuleCondition[];
}

class BulkDiscountRule implements PricingRule {
  priority = 1;
  conditions = [{ type: 'quantity', operator: '>=', value: 10 }];
  
  apply(item: CartItem): Money {
    if (item.quantity >= 10) {
      return item.unitPrice.multiply(0.9); // 10% discount
    }
    return item.unitPrice;
  }
}
```

## Specification Documents

### Software Requirements Specification (SRS)

A comprehensive SRS document typically includes:

```
1. Introduction
   - Purpose and scope
   - Intended audience
   - Definitions and acronyms

2. Overall Description
   - Product perspective
   - User characteristics
   - Product constraints
   - Assumptions and dependencies

3. Functional Requirements
   - User class descriptions
   - Functional specifications
   - Use cases

4. Non-Functional Requirements
   - Performance requirements
   - Security requirements
   - Reliability requirements
   - Availability requirements

5. Interface Requirements
   - User interfaces
   - Hardware interfaces
   - Software interfaces
   - Communication interfaces

6. Appendices
   - Data dictionary
   - Analysis models
   - Supporting information
```

### User Stories and Acceptance Criteria

Modern agile approaches prefer lightweight specifications:

```markdown
## User Story: Shopping Cart Checkout

**As a** registered customer
**I want to** complete my purchase using saved payment methods
**So that** I can quickly check out without entering card details

### Acceptance Criteria

1. User can select from saved payment methods
   - WHEN user clicks "Checkout"
   - THEN saved payment options are displayed

2. User can add new payment method during checkout
   - WHEN user selects "Add New Card"
   - THEN card entry form is shown

3. Payment processing handles errors gracefully
   - WHEN payment fails
   - THEN error message is displayed
   - AND user can retry or choose different payment
```

## Requirements in AI-Assisted Development

### Writing AI-Friendly Requirements

```markdown
## Good Requirements for AI
- ✅ Clear, specific language
- ✅ Defined acceptance criteria
- ✅ Explicit edge cases
- ✅ Expected error conditions
- ✅ Input/output format specifications

## Poor Requirements for AI
- ❌ Vague descriptions ("fast", "user-friendly")
- ❌ Missing error handling
- ❌ Ambiguous acceptance criteria
- ❌ Undefined input constraints
```

### Example: AI-Ready Specification

```markdown
## Feature: Currency Conversion API

### Requirements

1. **Endpoint**: POST /api/convert
2. **Input Format**:
   ```json
   {
     "from": "USD",  // ISO 4217 currency code
     "to": "EUR",    // ISO 4217 currency code
     "amount": 100.00 // Decimal, positive
   }
   ```

3. **Output Format**:
   ```json
   {
     "from": "USD",
     "to": "EUR",
     "originalAmount": 100.00,
     "convertedAmount": 92.50,
     "rate": 0.925,
     "timestamp": "2024-01-15T10:30:00Z"
   }
   ```

4. **Error Responses**:
   - 400: Invalid currency code or amount
   - 429: Rate limit exceeded
   - 503: Exchange rate service unavailable

5. **Business Rules**:
   - Maximum conversion amount: 1,000,000
   - Cached rates expire after 60 seconds
   - Round to 2 decimal places
```

## Requirements Validation

### Review Checklist

```markdown
## Requirements Quality Checklist

### Completeness
- [ ] All user needs are captured
- [ ] Edge cases are identified
- [ ] Error conditions are specified
- [ ] Assumptions are documented

### Consistency
- [ ] No contradictory requirements
- [ ] Terminology is consistent
- [ ] Priorities are assigned

### Testability
- [ ] Each requirement has acceptance criteria
- [ ] Criteria are measurable
- [ ] Success can be objectively verified

### Traceability
- [ ] Requirements have unique IDs
- [ ] Links to business objectives
- [ ] Backlog items traceable to requirements
```

## Tools and Techniques

| Tool/Technique | Purpose | Use Case |
|---------------|---------|----------|
| UserStoryMap | Visual story organization | Sprint planning |
| Impact Mapping | Goal-aligned requirements | Strategic planning |
| Example Mapping | Example discovery | BDD collaboration |
| Gherkin | Executable specifications | Test automation |
| OpenAPI | API specifications | Backend development |

## Best Practices

1. **Involve Stakeholders Early**: Engage users, business analysts, and developers in requirements gathering

2. **Use Concrete Examples**: Replace abstract requirements with specific, testable examples

3. **Prioritize Ruthlessly**: Not all requirements are equal - identify MVP vs. enhancements

4. **Maintain Living Documents**: Requirements evolve; keep documentation updated

5. **Automate Validation**: Link requirements to tests through executable specifications

6. **Consider AI Context**: Structure requirements for optimal AI understanding

## AI Coding Assistant Prompts

When working with AI assistants, structure requirements clearly:

```markdown
## Template for AI Requirements

**Feature Name**: [Descriptive name]

**Goal**: [One-sentence objective]

**Input Specification**:
- Format: [JSON/parameters/files]
- Validation rules: [List constraints]

**Output Specification**:
- Format: [JSON/response type]
- Success conditions: [What constitutes success]

**Error Cases**:
- [Case 1]: [Condition] → [Expected response]
- [Case 2]: [Condition] → [Expected response]

**Dependencies**:
- [External services]
- [Database tables]
- [API endpoints]

**Acceptance Criteria**:
1. [ ] [Testable criterion]
2. [ ] [Testable criterion]
```

## References

- IEEE 830-2014 - Recommended Practice for Software Requirements Specifications
- ATAM (Architecture Tradeoff Analysis Method)
- INVEST principles for user stories

---

## Requirements Template from Reference Projects

### Functional Requirements Structure

```markdown
# [Project Name] Requirements

## Project Overview

[Brief description of what the application does]

## Functional Requirements

### Command Summary (for CLI applications)
- `command1` - Description of what it does
- `command2` - Description of what it does

### [Module/Subsystem 1]

#### [Function 1.1]
- Description of the function
- Input parameters and validation rules
- Expected output
- Error conditions and messages

#### [Function 1.2]
- [Same structure]

### [Module/Subsystem 2]

[Same structure]

### Error Handling

- Invalid commands show appropriate error messages
- Non-existing resources trigger specific error messages
- Empty inputs are validated and rejected
- Duplicate entries are prevented with clear error messages
```

### Non-Functional Requirements Structure

```markdown
## Non-Functional Requirements

### 1. Architecture

#### [Architecture Pattern] Compliance
- Clear separation between layers:
    - [Layer 1]: [Description]
    - [Layer 2]: [Description]
    - [Layer 3]: [Description]

#### Domain-Driven Design
- Use of [Aggregate Roots]
- Value Objects for [identities and domain concepts]
- Immutable entities where appropriate
- Read-only wrappers for external access

### 2. Technology Stack

- **Language**: [Version]
- **Build Tool**: [Name]
- **Testing**: [Framework]
- **Web Framework**: [Name] (if applicable)
- **Database**: [Type] (for production)
- **Support Libraries**: [List]

### 3. Interface Requirements

#### CLI Interface
- Command-line based interaction
- Commands format: `command <parameters>`
- Real-time feedback for all operations

#### Web Interface (if applicable)
- RESTful API endpoints
- JSON request/response format
- Authentication mechanism

### 4. Code Quality

- Comprehensive unit test coverage
- Integration tests for major workflows
- Clear separation of concerns
- SOLID principles adherence
- Minimal code duplication

### 5. Performance

- Efficient [operations]
- Scalable [patterns] for future enhancement
- Lightweight [application] startup

## System Constraints

1. [Constraint 1]
2. [Constraint 2]
3. [Constraint 3]

## Future Considerations

1. [Feature 1]
2. [Feature 2]
3. [Feature 3]
```

---

## Spec Parsing Template for AI Assistants

When given a spec file, AI assistants should follow this parsing procedure:

### Step 1: Extract ALL Components

```markdown
## Components to Implement from Spec

### 1. Use Cases
- [ ] List all use cases mentioned

### 2. Services
- [ ] List all services needed

### 3. DTOs (Data Transfer Objects)
- [ ] List each DTO from spec

### 4. Projections
- [ ] List projection interfaces
- [ ] List projection implementations

### 5. Mappers (🚨 CRITICAL - OFTEN MISSED!)
- [ ] **MUST CHECK**: Does spec have a "mappers" section?
- [ ] List all mappers from spec
- [ ] Note package location: `[aggregate].usecase.port` (NOT adapter!)
- [ ] Note: Mappers convert between entities and DTOs
- [ ] **WARNING**: If spec says "must be generated" or "critical": true, this is MANDATORY!

### 6. Repositories
- [ ] List any custom repositories

### 7. Tests
- [ ] List required test files
```

### Step 2: Create Implementation Order

1. Domain entities/value objects (if needed)
2. **Mappers** (do NOT skip!)
3. DTOs
4. Use Case interfaces
5. Projections (interface then implementation)
6. Services
7. Tests

### Step 3: Validation Checklist

Before marking task complete:
- [ ] All DTOs from spec created?
- [ ] All Mappers from spec created?
- [ ] All Projections created (both interface AND implementation)?
- [ ] Mapper is injected and used in Projection?
- [ ] All tests passing?

### Common Mistakes to Avoid

1. ❌ Implementing DTO conversion directly in Projection instead of using Mapper
2. ❌ Forgetting to create Mapper when spec mentions it
3. ❌ Creating only Projection interface without implementation
4. ❌ Missing DTOs mentioned in spec

### Example Spec Analysis

Given a spec with:
```json
{
  "mappers": [
    {"name": "ProductMapper", "description": "..."}
  ],
  "projections": [...],
  "dataTransferObjects": [...]
}
```

MUST create:
1. ✅ ProductMapper class
2. ✅ Use ProductMapper in Projection implementation
3. ✅ All DTOs listed
4. ✅ Both Projection interface AND implementation

---

## Complex Aggregate Specification Template

Use this template for specifying complex aggregates with multiple entities.

### 1. Aggregate Overview

```yaml
aggregate:
  name: [AggregateRootName]
  description: [Business purpose and responsibility]
  bounded_context: [Context name]
  invariants:
    - [Business rule 1 that must always be true]
    - [Business rule 2 that must always be true]
```

### 2. Entity Hierarchy

```yaml
entities:
  root:
    name: [AggregateRootName]
    id_type: [IdType]
    properties:
      - name: [propertyName]
        type: [type]
        description: [purpose]
        constraints: [validation rules]
    
  children:
    - name: [EntityName]
      parent: [ParentEntityName]
      id_type: [IdType or embedded]
      cardinality: [one-to-one | one-to-many]
      properties:
        - name: [propertyName]
          type: [type]
          description: [purpose]
      invariants:
        - [Entity-specific business rules]
```

### 3. Value Objects

```yaml
value_objects:
  - name: [ValueObjectName]
    properties:
      - name: [propertyName]
        type: [type]
        validation: [rules]
    used_by: [List of entities using this VO]
```

### 4. Domain Events

```yaml
domain_events:
  - name: [EventName]
    trigger: [What action causes this event]
    properties:
      - name: [propertyName]
        type: [type]
        source: [Where value comes from]
```

### 5. Commands and Business Operations

```yaml
commands:
  - name: [CommandName]
    description: [What this command does]
    parameters:
      - name: [paramName]
        type: [type]
        required: [true/false]
    validations:
      - [Validation rule]
    side_effects:
      - [What happens as result]
    events: [List of events emitted]
```

### 6. Aggregate Relationships

```yaml
relationships:
  - type: [association | composition | aggregation]
    from: [EntityName]
    to: [EntityName]
    cardinality: [1..1 | 1..* | 0..* | etc]
    navigation: [unidirectional | bidirectional]
    cascade: [operations that cascade]
```

### Example: Workflow Aggregate with Lanes

```yaml
aggregate:
  name: Workflow
  description: Represents a workflow board with stages and swimlanes
  bounded_context: Workflow Management
  invariants:
    - A workflow must have at least one stage
    - Stage order must be unique within a workflow
    - Swimlane names must be unique within a workflow

entities:
  root:
    name: Workflow
    id_type: WorkflowId
    properties:
      - name: boardId
        type: String
        description: Reference to the board
        constraints: not null
      - name: name
        type: String
        description: Display name
        constraints: not blank, max 100 chars
      - name: lanes
        type: List<Lane>
        description: Collection of stages and swimlanes
        constraints: not empty
    
  children:
    - name: Lane
      parent: Workflow
      id_type: embedded
      cardinality: one-to-many
      properties:
        - name: laneId
          type: String
        - name: type
          type: LaneType (enum: STAGE, SWIMLANE)
        - name: name
          type: String
      invariants:
        - Lane IDs must be unique within workflow
        
    - name: Stage
      parent: Lane (inheritance)
      properties:
        - name: order
          type: int
          description: Position in workflow (0-based)
          constraints: >= 0, unique within workflow
        - name: wipLimit
          type: Integer (optional)
          description: Work in progress limit
      invariants:
        - Stages must have sequential order without gaps

value_objects:
  - name: LaneType
    properties:
      - name: value
        type: String enum (STAGE, SWIMLANE)
    used_by: [Lane]

domain_events:
  - name: WorkflowCreated
    trigger: Create workflow command
    properties:
      - name: workflowId
        type: String
      - name: name
        type: String

commands:
  - name: CreateWorkflow
    description: Creates a new workflow
    parameters:
      - name: boardId
        type: String
        required: true
      - name: name
        type: String
        required: true
    validations:
      - Name must not be blank
    events: [WorkflowCreated]
```

### Tips for AI Communication with Complex Specifications

1. **Start with the Big Picture**: Describe the business purpose before diving into technical details

2. **Use Concrete Examples**:
   ```
   "A Workflow contains multiple Lanes. A Lane can be either a Stage (horizontal) or Swimlane (vertical). For example: 'To Do', 'In Progress', 'Done' are Stages."
   ```

3. **Specify Constraints Clearly**:
   ```
   "Stages must maintain sequential order (0, 1, 2...) with no gaps. When a stage is removed, subsequent stages must be reordered."
   ```

4. **Define Business Rules**:
   ```
   "A card can be in exactly one stage and optionally one swimlane at any time."
   ```

5. **Clarify Relationships**:
   ```
   "Order OWNS OrderItems (composition) - deleting an order deletes all its items. Order REFERENCES Customer (association) - deleting a customer doesn't automatically delete orders."
   ```
