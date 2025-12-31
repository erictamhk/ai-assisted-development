# Living Documentation

## Core Concept

Living documentation is documentation that evolves automatically with the codebase, always reflecting the current state of the system. Unlike traditional documentation that quickly becomes outdated, living documentation is generated from the code itself, including tests, specifications, and code comments, ensuring that documentation and implementation remain synchronized.

The term was popularized by Cyrille Martraire in his 2019 book "Living Documentation: Continuous Documentation with Tests." His core insight is that documentation should not be a separate artifact maintained by hand, but rather a byproduct of the development process itself, generated from existing artifacts like tests, specifications, and code.

Living documentation addresses the persistent problem of documentation decay. In most projects, documentation starts out accurate but quickly diverges from reality as the code changes. This divergence leads to confusion, bugs, and wasted time. Living documentation solves this by making documentation a natural output of the development workflow.

The key principle is that documentation should be derived from the same sources as the code itself. When tests serve as specifications, running tests both validates the system and generates documentation. When code includes semantic markup, tools can extract and present that information in useful formats.

---

## Living Documentation Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  LIVING DOCUMENTATION ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    SOURCE OF TRUTH                               │    │
│  │                                                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │    │
│  │  │    CODE     │ │   TESTS     │ │  SPECS      │                │    │
│  │  │             │ │             │ │             │                │    │
│  │  │  • Classes  │ │  • Unit     │ │  • Gherkin  │                │    │
│  │  │  • Modules  │ │  • BDD      │ │  • OpenAPI  │                │    │
│  │  │  • Functions│ │  • Contract │ │  • Schema   │                │    │
│  │  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘                │    │
│  │         │               │               │                         │    │
│  └─────────┼───────────────┼───────────────┼─────────────────────────┘    │
│            │               │               │                              │
│            ▼               ▼               ▼                              │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    DOCUMENTATION BUILDERS                        │    │
│  │                                                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │    │
│  │  │   JSDoc/    │ │   Test      │ │   Spec      │                │    │
│  │  │   TSDoc     │ │   Reports   │ │   Reports   │                │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘                │    │
│  │                                                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │    │
│  │  │   API       │ │   BDD       │ │   Architecture│               │    │
│  │  │   Docs      │ │   Scenarios │ │   Diagrams  │                │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘                │    │
│  │                                                                 │    │
│  └─────────────────────────────┬───────────────────────────────────┘    │
│                                │                                         │
│                                ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    OUTPUT FORMATS                                │    │
│  │                                                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │    │
│  │  │   HTML      │ │   Markdown  │ │   PDF       │                │    │
│  │  │   Docs      │ │   Files     │ │   Reports   │                │    │
│  │  └─────────────┘ └─────────────┘ └──────┬──────┘                │    │
│  │                                           │                        │    │
│  │  ┌─────────────┐ ┌─────────────┐          │                        │    │
│  │  │   Wiki      │ │   Confluence│          │                        │    │
│  │  │   Pages     │ │   Export    │          │                        │    │
│  │  └─────────────┘ └─────────────┘          │                        │    │
│  │                                           │                        │    │
│  │         ┌─────────────────────────────────┘                        │    │
│  │         │                                                          │    │
│  │         ▼                                                          │    │
│  │  ┌─────────────────────────────────────────────────────────────┐  │    │
│  │  │               LIVING DOCUMENTATION PORTAL                    │  │    │
│  │  │                                                               │  │    │
│  │  │   • Always up-to-date                                        │  │    │
│  │  │   • Searchable                                               │  │    │
│  │  │   • Cross-referenced                                         │  │    │
│  │  │   • Generated continuously                                   │  │    │
│  │  └─────────────────────────────────────────────────────────────┘  │  │
│  │                                                                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## API Documentation with TypeDoc

```typescript
/**
 * Living Documentation using TSDoc
 */

/**
 * Represents a product in the e-commerce system.
 * 
 * @remarks
 * This entity is part of the Product Aggregate and should be accessed
 * only through the ProductRepository.
 * 
 * @example
 * ```typescript
 * const product = new Product({
 *   sku: 'SKU-123',
 *   name: 'Widget',
 *   price: new Money(29.99, Currency.USD)
 * });
 * ```
 * 
 * @see {@link ProductRepository} for access patterns
 * @see {@link ProductAggregate} for aggregate boundaries
 */
export class Product {
  /**
   * Unique identifier for the product
   * 
   * @remarks
   * The SKU is assigned during product creation and cannot be changed.
   * This ensures stable references across the system.
   */
  private readonly sku: ProductSku;

  /**
   * Human-readable product name
   * 
   * @maxLength 200 characters
   * @pattern ^[\w\s\-']+$
   */
  private readonly name: string;

  /**
   * Product pricing information
   * 
   * @see {@link Money} for currency handling
   */
  private readonly price: Money;

  /**
   * Current inventory quantity
   * 
   * @minimum 0
   * @defaultValue 0
   */
  private quantity: number;

  /**
   * Product category for organization
   */
  private readonly category: Category;

  /**
   * Creates a new Product instance.
   * 
   * @param input - Product creation parameters
   * @returns A new Product instance
   * 
   * @throws {@link ValidationError}
   * Thrown when input validation fails
   * 
   * @example
   * ```typescript
   * const product = Product.create({
   *   sku: 'SKU-456',
   *   name: 'Gadget Pro',
   *   price: new Money(99.99, Currency.USD),
   *   category: Category.Electronics
   * });
   * ```
   */
  static create(input: ProductInput): Product {
    const sku = ProductSku.fromString(input.sku);
    const price = input.price;
    
    return new Product(sku, input.name, price, 0, input.category);
  }

  /**
   * Updates the product price.
   * 
   * @param newPrice - The new price to set
   * 
   * @returns Product with updated price
   * 
   * @precondition
   * - Price must be greater than zero
   * 
   * @postcondition
   * - Product price is updated
   * - Price change is recorded in product history
   */
  updatePrice(newPrice: Money): Product {
    if (newPrice.lessThanOrEqual(Money.zero())) {
      throw new ValidationError('Price must be positive');
    }

    return new Product(this.sku, this.name, newPrice, this.quantity, this.category);
  }

  /**
   * Checks if the product is in stock.
   * 
   * @returns true if quantity > 0
   * 
   * @example
   * ```typescript
   * if (product.isInStock()) {
   *   // Add to cart
   * }
   * ```
   */
  isInStock(): boolean {
    return this.quantity > 0;
  }

  /**
   * Reserves inventory for an order.
   * 
   * @param quantity - Quantity to reserve
   * @returns Reservation result with success status
   * 
   * @throws {@link InsufficientInventoryError}
   * Thrown when insufficient inventory is available
   */
  reserveInventory(quantity: number): ReservationResult {
    if (this.quantity < quantity) {
      return {
        success: false,
        reason: 'Insufficient inventory',
        available: this.quantity,
      };
    }

    this.quantity -= quantity;
    return { success: true, reserved: quantity };
  }
}

/**
 * Input parameters for creating a Product
 * 
 * @category Input Types
 */
export interface ProductInput {
  /** Unique stock keeping unit */
  sku: string;
  
  /** Product display name */
  name: string;
  
  /** Product price */
  price: Money;
  
  /** Product category */
  category: Category;
}

/**
 * Result of an inventory reservation operation
 * 
 * @category Domain Types
 */
export interface ReservationResult {
  /** Whether the reservation was successful */
  success: boolean;
  
  /** Quantity reserved (if successful) */
  reserved?: number;
  
  /** Reason for failure (if unsuccessful) */
  reason?: string;
  
  /** Available inventory (if failed) */
  available?: number;
}
```

---

## BDD Scenarios as Documentation

```gherkin
# features/inventory/inventory-management.feature
@documentation @inventory @smoke
Feature: Inventory Management
  As a store manager
  I want to track and manage product inventory
  So that I can ensure products are available for customers

  # This feature serves as both specification and documentation
  # Run with: cucumber --profile documentation

  Background:
    Given the following products exist in inventory:
      | sku       | name         | quantity | price |
      | ELEC-001  | Smartphone   | 50       | 599.99|
      | ELEC-002  | Headphones   | 100      | 149.99|
      | HOME-001  | Coffee Maker | 30       | 79.99 |

  @smoke
  Scenario: View current inventory levels
    When I request the inventory report
    Then I should see all products with their quantities
    And the total product count should be 180
    And the total inventory value should be $63,496.50

  Scenario: Add new inventory stock
    Given product "ELEC-001" has 50 units in stock
    When I add 25 units to "ELEC-001"
    Then "ELEC-001" should have 75 units in stock
    And the inventory value should increase by $14,999.75

  Scenario: Remove inventory for shipped orders
    Given customer places an order for 2 "ELEC-001" units
    When the order is marked as shipped
    Then "ELEC-001" inventory should decrease by 2 units
    And the order status should be "shipped"

  @edge-case
  Scenario: Prevent negative inventory
    Given product "HOME-001" has 5 units in stock
    When I attempt to remove 10 units
    Then an error "Cannot reduce below zero" should be displayed
    And the inventory should remain at 5 units

  @edge-case
  Scenario: Handle inventory for concurrent orders
    Given product "ELEC-002" has 10 units in stock
    When 5 customers simultaneously order 3 units each
    Then 3 orders should be fulfilled
    And 2 orders should be declined due to insufficient inventory

  @documentation
  Scenario: Inventory thresholds and alerts
    Given a low stock threshold of 20 units
    When product "ELEC-001" quantity drops to 15
    Then a "LOW_STOCK" alert should be triggered
    And the store manager should receive a notification

  @documentation
  Scenario: Inventory valuation report
    When I generate the inventory valuation report
    Then I should see:
      | category | item_count | total_value |
      | ELEC     | 150        | $44,996.50  |
      | HOME     | 30         | $2,399.70   |
```

---

## Architecture Decision Records (ADRs)

```markdown
# ADR-001: Adopt Event Sourcing for Order Management

## Status
**Accepted** | Date: 2024-01-15

## Context
We need to implement the order management system. The requirements include:
- Complete audit trail of all order changes
- Ability to replay order history for debugging
- Support for order state machine transitions
- Integration with multiple downstream systems

## Decision
We will use **Event Sourcing** to persist all order state changes as a sequence of events.

### Implementation
```typescript
class Order {
  private events: OrderEvent[] = [];

  apply(event: OrderEvent): void {
    this.events.push(event);
    this.transitionState(event);
  }

  getStateHistory(): OrderEvent[] {
    return [...this.events];
  }
}
```

## Consequences

### Positive
- ✅ Complete audit trail without separate logging
- ✅ Ability to reconstruct any order state at any point in time
- ✅ Natural fit for order state machine
- ✅ Easy integration with external systems via event consumers

### Negative
- ❌ Increased complexity in data access patterns
- ❌ Need for event schema evolution strategies
- ❌ Potential performance implications for large order histories

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| **Current State Only** | Simple, familiar | No audit trail, harder debugging |
| **Snapshot + Events** | Better performance | More complexity |

## Related
- Related to: [ADR-002] CQRS Implementation
- Implements: RFC-001 Order Management Requirements
- Supersedes: ADR-000 (previous approach)
```

---

## Living Documentation with Custom Generators

```typescript
// scripts/generate-docs.ts
import * as fs from 'fs';
import * as path from 'path';
import { parse } from '@typescript-eslint/typescript-estree';

interface DocumentationConfig {
  sourceDir: string;
  outputDir: string;
  templates: TemplateConfig;
}

interface TemplateConfig {
  api: string;
  architecture: string;
  domain: string;
}

class LivingDocumentationGenerator {
  private config: DocumentationConfig;

  constructor(config: DocumentationConfig) {
    this.config = config;
  }

  async generateAll(): Promise<void> {
    await this.generateAPIDocumentation();
    await this.generateArchitectureDocumentation();
    await this.generateDomainDocumentation();
    await this.generateIndex();
  }

  private async generateAPIDocumentation(): Promise<void> {
    const files = this.findTypeScriptFiles(this.config.sourceDir);
    const apis = [];

    for (const file of files) {
      const ast = parse(fs.readFileSync(file, 'utf8'));
      const classes = this.extractClasses(ast);
      
      for (const cls of classes) {
        const doc = this.generateClassDocumentation(cls, file);
        apis.push(doc);
      }
    }

    const output = this.renderTemplate('api', { apis });
    this.writeFile('api.md', output);
  }

  private async generateArchitectureDocumentation(): Promise<void> {
    const architecture = {
      layers: this.detectLayers(),
      dependencies: this.analyzeDependencies(),
      patterns: this.identifyPatterns(),
      boundaries: this.findBoundaries(),
    };

    const output = this.renderTemplate('architecture', architecture);
    this.writeFile('architecture.md', output);
  }

  private async generateDomainDocumentation(): Promise<void> {
    const domain = {
      aggregates: this.findAggregates(),
      entities: this.findEntities(),
      valueObjects: this.findValueObjects(),
      domainEvents: this.findDomainEvents(),
      services: this.findDomainServices(),
    };

    const output = this.renderTemplate('domain', domain);
    this.writeFile('domain.md', output);
  }

  private async generateIndex(): Promise<void> {
    const index = {
      lastUpdated: new Date().toISOString(),
      sections: [
        { title: 'API Reference', file: 'api.md', description: 'Complete API documentation' },
        { title: 'Architecture', file: 'architecture.md', description: 'System architecture overview' },
        { title: 'Domain Model', file: 'domain.md', description: 'Domain-driven design documentation' },
        { title: 'ADRs', file: 'adrs/index.md', description: 'Architecture Decision Records' },
      ],
    };

    const output = this.renderTemplate('index', index);
    this.writeFile('README.md', output);
  }

  private findTypeScriptFiles(dir: string): string[] {
    const results: string[] = [];
    const entries = fs.readdirSync(dir, { withFileTypes: true });

    for (const entry of entries) {
      const fullPath = path.join(dir, entry.name);
      if (entry.isDirectory()) {
        results.push(...this.findTypeScriptFiles(fullPath));
      } else if (entry.name.endsWith('.ts') && !entry.name.endsWith('.test.ts')) {
        results.push(fullPath);
      }
    }

    return results;
  }

  private extractClasses(ast: any): any[] {
    return ast.body.filter((node: any) => 
      node.type === 'ClassDeclaration'
    );
  }

  private generateClassDocumentation(cls: any, file: string): object {
    return {
      name: cls.id.name,
      location: file,
      methods: cls.body.body.map((method: any) => ({
        name: method.key.name,
        visibility: this.getVisibility(method),
        parameters: method.params.map((p: any) => ({
          name: p.name,
          type: this.getType(p.typeAnnotation),
        })),
        returns: this.getType(method.returnType),
        decorators: method.decorators?.map((d: any) => d.expression.name),
      })),
      properties: cls.body.body
        .filter((node: any) => node.type === 'PropertyDeclaration')
        .map((prop: any) => ({
          name: prop.key.name,
          type: this.getType(prop.typeAnnotation),
          visibility: this.getVisibility(prop),
        })),
      heritage: cls.decorators?.map((d: any) => ({
        type: d.expression.name,
        arguments: d.expression.arguments?.map((a: any) => a.value),
      })),
    };
  }

  private detectLayers(): object {
    return {
      application: this.findApplicationLayer(),
      domain: this.findDomainLayer(),
      infrastructure: this.findInfrastructureLayer(),
      interfaces: this.findInterfaceLayer(),
    };
  }

  private findAggregates(): object[] {
    const files = this.findTypeScriptFiles(path.join(this.config.sourceDir, 'domain'));
    const aggregates: object[] = [];

    for (const file of files) {
      const content = fs.readFileSync(file, 'utf8');
      if (content.includes('@Aggregate')) {
        const className = this.extractClassName(content);
        aggregates.push({
          name: className,
          file,
          roots: this.findAggregateRoots(content),
          entities: this.findAggregateEntities(content),
        });
      }
    }

    return aggregates;
  }

  private writeFile(filename: string, content: string): void {
    const outputPath = path.join(this.config.outputDir, filename);
    fs.mkdirSync(path.dirname(outputPath), { recursive: true });
    fs.writeFileSync(outputPath, content);
    console.log(`Generated: ${outputPath}`);
  }

  private renderTemplate(templateName: string, data: any): string {
    const template = this.config.templates[templateName as keyof TemplateConfig];
    // Template rendering logic
    return `# ${templateName.toUpperCase()} Documentation\n\n${JSON.stringify(data, null, 2)}`;
  }
}

// Configuration
const config: DocumentationConfig = {
  sourceDir: './src',
  outputDir: './docs',
  templates: {
    api: './templates/api.md',
    architecture: './templates/architecture.md',
    domain: './templates/domain.md',
  },
};

// Run generation
const generator = new LivingDocumentationGenerator(config);
generator.generateAll().then(() => {
  console.log('Living documentation generation complete');
});
```

---

## Testing Documentation Coverage

```typescript
// scripts/check-docs-coverage.ts
import * as fs from 'fs';
import * as path from 'path';

interface CoverageReport {
  totalClasses: number;
  documentedClasses: number;
  totalMethods: number;
  documentedMethods: number;
  totalFunctions: number;
  documentedFunctions: number;
  coverage: {
    classes: number;
    methods: number;
    functions: number;
  };
}

function checkDocumentationCoverage(sourceDir: string): CoverageReport {
  const files = findTypeScriptFiles(sourceDir);
  
  let totalClasses = 0;
  let documentedClasses = 0;
  let totalMethods = 0;
  let documentedMethods = 0;
  let totalFunctions = 0;
  let documentedFunctions = 0;

  for (const file of files) {
    const content = fs.readFileSync(file, 'utf8');
    const ast = parse(content);

    for (const node of ast.body) {
      if (node.type === 'ClassDeclaration') {
        totalClasses++;
        if (hasTSDocComment(node.leadingComments)) {
          documentedClasses++;
        }

        for (const member of node.body.body) {
          if (member.type === 'MethodDefinition') {
            totalMethods++;
            if (hasTSDocComment(member.leadingComments)) {
              documentedMethods++;
            }
          }
        }
      }

      if (node.type === 'FunctionDeclaration') {
        totalFunctions++;
        if (hasTSDocComment(node.leadingComments)) {
          documentedFunctions++;
        }
      }
    }
  }

  return {
    totalClasses,
    documentedClasses,
    totalMethods,
    documentedMethods,
    totalFunctions,
    documentedFunctions,
    coverage: {
      classes: (documentedClasses / totalClasses) * 100,
      methods: (documentedMethods / totalMethods) * 100,
      functions: (documentedFunctions / totalFunctions) * 100,
    },
  };
}

function findTypeScriptFiles(dir: string): string[] {
  const results: string[] = [];
  const entries = fs.readdirSync(dir, { withFileTypes: true });

  for (const entry of entries) {
    const fullPath = path.join(dir, entry.name);
    if (entry.isDirectory()) {
      results.push(...findTypeScriptFiles(fullPath));
    } else if (entry.name.endsWith('.ts')) {
      results.push(fullPath);
    }
  }

  return results;
}

function hasTSDocComment(comments: any[]): boolean {
  return comments?.some((c: any) => 
    c.value.includes('/**') && c.value.includes('*/')
  ) || false;
}

// Generate coverage report
const report = checkDocumentationCoverage('./src');

console.log('Documentation Coverage Report');
console.log('='.repeat(50));
console.log(`Classes: ${report.documentedClasses}/${report.totalClasses} (${report.coverage.classes.toFixed(1)}%)`);
console.log(`Methods: ${report.documentedMethods}/${report.totalMethods} (${report.coverage.methods.toFixed(1)}%)`);
console.log(`Functions: ${report.documentedFunctions}/${report.totalFunctions} (${report.coverage.functions.toFixed(1)}%)`);
console.log('='.repeat(50));

// Fail if coverage is below threshold
const MIN_COVERAGE = 80;
if (report.coverage.classes < MIN_COVERAGE || report.coverage.methods < MIN_COVERAGE) {
  console.error(`❌ Documentation coverage below ${MIN_COVERAGE}% threshold`);
  process.exit(1);
} else {
  console.log(`✅ Documentation coverage meets threshold`);
}
```

---

## Living Documentation Best Practices

### 1. Derive Documentation from Code
Documentation should be generated from the code itself, not written separately. This ensures synchronization.

### 2. Use Markup in Code
Add TSDoc/JSDoc comments to code elements. These become automatically generated documentation.

### 3. Make Tests Serve Double Duty
Tests should validate behavior and serve as executable specifications and documentation.

### 4. Version Documentation
Treat documentation as a versioned artifact that changes with the codebase.

### 5. Generate Continuously
Run documentation generation in CI/CD to ensure documentation is always current.

### 6. Structure for Discoverability
Organize documentation so that users can find what they need quickly.

### 7. Include Examples
Use code examples in documentation to show how to use APIs and features.

### 8. Cross-Reference Content
Link related documentation to help users navigate complex systems.

### 9. Track Documentation Coverage
Monitor how much of your codebase is documented and set coverage targets.

### 10. Treat Documentation as Code
Apply the same quality standards, reviews, and tooling to documentation as to code.

---

## Living Documentation Anti-Patterns

### 1. Documentation as Afterthought
Creating documentation after development is complete, when it will already be outdated.

### 2. Static Documentation
Using Word documents or PDFs that cannot be easily updated and quickly become stale.

### 3. Duplicate Information
Maintaining separate documentation and tests that can diverge over time.

### 4. Missing Documentation Tests
Not verifying that documentation is accurate, allowing errors to persist.

### 5. Over-Documentation
Documenting trivial code elements while missing important domain concepts.

### 6. Out-of-Context Documentation
Providing documentation without explaining how pieces fit together.

### 7. No Search Functionality
Creating documentation that is difficult to navigate or search.

### 8. Ignoring Documentation in Code Reviews
Not reviewing documentation quality with the same rigor as code.

### 9. No Ownership
Having no clear responsibility for documentation maintenance.

### 10. Documentation in Wrong Format
Using formats that are not searchable, versionable, or easy to maintain.

---

## Living Documentation in AI-Assisted Development

When working with AI assistants for living documentation, follow these practices:

### 1. Generate Initial Templates
Ask AI to help create documentation templates and structures for your project.

### 2. Improve Existing Documentation
Use AI to review and enhance existing documentation for clarity and completeness.

### 3. Generate Examples
Ask AI to generate code examples and use cases for documentation.

### 4. Check Consistency
Use AI to verify terminology consistency across documentation.

### 5. Create Indexes
Ask AI to generate indexes and cross-references for documentation.

### 6. Translate to Multiple Formats
Convert documentation to different formats using AI assistance.

### 7. Summarize Complex Topics
Use AI to create executive summaries of complex technical documentation.

### 8. Generate Visual Diagrams
Use AI to describe and generate diagrams from documentation.

---

## Tools for Living Documentation

| Tool | Purpose | Format |
|------|---------|--------|
| **TypeDoc** | TypeScript API docs | HTML |
| **JSDoc** | JavaScript API docs | HTML |
| **Docusaurus** | Documentation site | MDX |
| **Sphinx** | Python docs | reStructuredText |
| **GitBook** | Modern documentation | Markdown |
| **MkDocs** | Static site generator | Markdown |
| **Storybook** | Component documentation | React |
| **Swagger/OpenAPI** | API documentation | YAML/JSON |
| **Cucumber Reports** | BDD documentation | HTML |
| **Compodoc** | Angular documentation | HTML |

---

## Living Documentation Architecture

Living documentation is built on the principle that documentation should be derived from the same sources as the code itself. This creates a single source of truth that stays synchronized with the codebase.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  LIVING DOCUMENTATION ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    SOURCE OF TRUTH                               │    │
│  │                                                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │    │
│  │  │    CODE     │ │   TESTS     │ │  SPECS      │                │    │
│  │  │             │ │             │ │             │                │    │
│  │  │  • Classes  │ │  • Unit     │ │  • Gherkin  │                │    │
│  │  │  • Modules  │ │  • BDD      │ │  • OpenAPI  │                │    │
│  │  │  • Functions│ │  • Contract │ │  • Schema   │                │    │
│  │  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘                │    │
│  │         │               │               │                         │    │
│  └─────────┼───────────────┼───────────────┼─────────────────────────┘    │
│            │               │               │                              │
│            ▼               ▼               ▼                              │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    DOCUMENTATION BUILDERS                        │    │
│  │                                                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │    │
│  │  │   JSDoc/    │ │   Test      │ │   Spec      │                │    │
│  │  │   TSDoc     │ │   Reports   │ │   Reports   │                │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘                │    │
│  │                                                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │    │
│  │  │   API       │ │   BDD       │ │   Architecture│               │    │
│  │  │   Docs      │ │   Scenarios │ │   Diagrams  │                │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘                │    │
│  │                                                                 │    │
│  └─────────────────────────────┬───────────────────────────────────┘    │
│                                │                                         │
│                                ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    OUTPUT FORMATS                                │    │
│  │                                                                 │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                │    │
│  │  │   HTML      │ │   Markdown  │ │   PDF       │                │    │
│  │  │   Docs      │ │   Files     │ │   Reports   │                │    │
│  │  └─────────────┘ └─────────────┘ └──────┬──────┘                │    │
│  │                                           │                        │    │
│  │  ┌─────────────┐ ┌─────────────┐          │                        │    │
│  │  │   Wiki      │ │   Confluence│          │                        │    │
│  │  │   Pages     │ │   Export    │          │                        │    │
│  │  └─────────────┘ └─────────────┘          │                        │    │
│  │                                           │                        │    │
│  │         ┌─────────────────────────────────┘                        │    │
│  │         │                                                          │    │
│  │         ▼                                                          │    │
│  │  ┌─────────────────────────────────────────────────────────────┐  │    │
│  │  │               LIVING DOCUMENTATION PORTAL                    │  │    │
│  │  │                                                               │  │    │
│  │  │   • Always up-to-date                                        │  │    │
│  │  │   • Searchable                                               │  │    │
│  │  │   • Cross-referenced                                         │  │    │
│  │  │   • Generated continuously                                   │  │    │
│  │  └─────────────────────────────────────────────────────────────┘  │    │
│  │                                                                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Documentation Sources and Their Outputs

| Source Type | Documentation Generated | Tools |
|-------------|------------------------|-------|
| Code Comments | API documentation, Usage guides | TypeDoc, JSDoc |
| Unit Tests | Function behavior, Edge cases | Test reports |
| BDD Scenarios | Feature documentation, User guides | Cucumber reports |
| OpenAPI Specs | API reference, Integration guides | Swagger UI |
| Architecture ADRs | Design decisions, Rationales | Markdown, GitBook |

---

## Tests as Living Documentation

BDD tests and executable specifications serve as living documentation that is always in sync with the codebase.

### BDD Tests as Documentation

```typescript
/**
 * User Registration Feature
 * 
 * @see {@link UserRegistration.test.ts} for complete behavior specification
 * 
 * ## Requirements
 * - Email must be unique
 * - Password must be at least 8 characters
 * - Password must contain uppercase, lowercase, number, and special character
 * - Verification email must be sent
 * 
 * ## Examples
 */
describe('User Registration', () => {
  
  it('should register user with valid credentials', async () => {
    const result = await registerUser({
      email: 'newuser@example.com',
      password: 'SecurePass123!',
      name: 'John Doe'
    });
    
    expect(result.success).toBe(true);
    expect(result.user.email).toBe('newuser@example.com');
  });
});
```

### README Integration with Tests

```markdown
## User Registration

See [User Registration Tests](src/features/user/tests/integration/UserRegistration.test.ts) 
for complete behavior specification.

### Valid Registration Scenarios
- [ ] New user registration with valid email and password
- [ ] Registration with unique email address
- [ ] Automatic username generation

### Invalid Registration Scenarios  
- [ ] Registration with duplicate email (Error: EMAIL_EXISTS)
- [ ] Registration with weak password (Error: WEAK_PASSWORD)
- [ ] Registration with missing required fields (Error: VALIDATION_ERROR)
```

---

## Collaboration and Living Documentation

Living documentation is fundamentally about collaboration - making knowledge accessible and maintainable across the team.

### Documentation Ownership

| Documentation Type | Owner | Update Trigger |
|-------------------|-------|----------------|
| API Documentation | Development Team | Code changes |
| BDD Scenarios | Product + QA | Feature changes |
| Architecture Decisions | Architects | Design changes |
| User Guides | Technical Writers | Release updates |
| Runbooks | Operations | Process changes |

### Documentation Review Process

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 DOCUMENTATION REVIEW WORKFLOW                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                │
│  │   DOCUMENT  │────►│    AI       │────►│   HUMAN     │                │
│  │   CHANGE    │     │   REVIEW    │     │   REVIEW    │                │
│  └─────────────┘     └─────────────┘     └─────────────┘                │
│        │                                       │                        │
│        │                                       │                        │
│        ▼                                       ▼                        │
│  ┌─────────────┐                         ┌─────────────┐                │
│  │  Automated  │                         │   APPROVED  │                │
│  │  Generation │                         │   / REJECT  │                │
│  └─────────────┘                         └─────────────┘                │
│                                                │                        │
│                                                ▼                        │
│                                         ┌─────────────┐                │
│                                         │  PUBLISHED  │                │
│                                         │   TO WIKI   │                │
│                                         └─────────────┘                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Cross-Team Documentation

Living documentation supports collaboration across different teams:

- **Shared Language**: Tests and specs provide a common vocabulary
- **Onboarding**: New team members can understand the system through documentation
- **Knowledge Transfer**: Reduces dependency on individual knowledge
- **Stakeholder Visibility**: Non-technical stakeholders can understand system behavior

---

## References and Further Reading

1. Martraire, Cyrille. "Living Documentation: Continuous Documentation with Tests." Addison-Wesley, 2019.
2. "TypeDoc Documentation." TypeDoc.
3. "Living Documentation." Martin Fowler.
4. "Documenting Architecture Decisions." Michael Nygard.
5. "TSDoc: TypeScript Documentation Standard." Microsoft.
6. ref/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md - BDD methodology
7. ref/engineering/specification_by_example/SPECIFICATION_BY_EXAMPLE.md - Specification by Example
8. doc/behavior-driven-development.md - BDD comprehensive guide
9. doc/executable-specifications.md - Executable specifications guide

1. Martraire, Cyrille. "Living Documentation: Continuous Documentation with Tests." Addison-Wesley, 2019.
2. "TypeDoc Documentation." TypeDoc.
3. "Living Documentation." Martin Fowler.
4. "Documenting Architecture Decisions." Michael Nygard.
5. "TSDoc: TypeScript Documentation Standard." Microsoft.
