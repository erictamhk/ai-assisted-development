# Problem Frames Approach

## Core Concept

The Problem Frames Approach, developed by Michael Jackson in his 2001 book "Software Requirements & Specifications," is a systematic method for analyzing and structuring software requirements. The approach provides a framework for understanding the relationship between the problem domain and the machine that will be built to address it.

The key insight of Problem Frames is that software problems can be decomposed into well-understood categories, each with characteristic structures and solution patterns. By identifying which problem frame applies to a given situation, analysts can leverage proven analysis techniques and avoid common pitfalls.

Problem Frames emphasizes the distinction between the problem domain (the part of the world that has the problem), the machine (the software or system being built), and the interface (how the machine connects to the problem domain). This clear separation helps teams focus on understanding the domain before designing solutions.

Unlike many requirements methods that focus on capturing user needs, Problem Frames focuses on understanding the structure of the problem itself. This helps ensure that requirements address the actual problem rather than assumed solutions, leading to more robust and maintainable software.

---

## The Three Worlds Framework

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    THE THREE WORLDS FRAMEWORK                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│       ┌─────────────────────────────────────────────────────────┐       │
│       │                                                         │       │
│       │                    MACHINE (M)                          │       │
│       │                                                         │       │
│       │     The software or system we are going to build        │       │
│       │                                                         │       │
│       │     ┌─────────────────────────────────────────────┐     │       │
│       │     │                                             │     │       │
│       │     │  ┌─────────────────────────────────────┐   │     │       │
│       │     │  │                                     │   │     │       │
│       │     │  │           SOFTWARE                  │   │     │       │
│       │     │  │                                     │   │     │       │
│       │     │  │     • Business Logic               │   │     │       │
│       │     │  │     • Data Processing              │   │     │       │
│       │     │  │     • User Interface               │   │     │       │
│       │     │  │                                     │   │     │       │
│       │     │  └─────────────────────────────────────┘   │     │       │
│       │     │                                             │     │       │
│       │     └─────────────────────────────────────────────┘     │       │
│       │                                                         │       │
│       └─────────────────────────────────────────────────────────┘       │
│                    │                             │                      │
│                    │        INTERFACE            │                      │
│                    │                             │                      │
│                    ▼                             ▼                      │
│       ┌─────────────────────────┐   ┌─────────────────────────┐        │
│       │                         │   │                         │        │
│       │   PROBLEM DOMAIN (D)    │   │  REQUIREMENTS (R)       │        │
│       │                         │   │                         │        │
│       │  The part of the world  │   │  What the machine must  │        │
│       │  that has the problem   │   │  do to solve it         │        │
│       │                         │   │                         │        │
│       │  • Existing systems     │   │  • Functional           │        │
│       │  • Physical phenomena   │   │  • Performance          │        │
│       │  • Business processes   │   │  • Security             │        │
│       │  • User behaviors       │   │  • Reliability          │        │
│       │                         │   │                         │        │
│       └─────────────────────────┘   └─────────────────────────┘        │
│                                                                         │
│         M ∩ D = Interface Phenomena (shared concerns)                   │
│         R ⊆ M: Requirements are properties of the machine               │
│         D → R: Domain properties constrain requirements                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Concepts

**Problem Domain (D)**: The part of the world that exhibits the problem. This includes existing systems, physical phenomena, business processes, and user behaviors that the machine will interact with.

**Machine (M)**: The software or system being built. The machine is designed to modify or control the problem domain to achieve desired effects.

**Requirements (R)**: A specification of what the machine must do. Requirements are properties that the machine must satisfy, expressed in terms of observable phenomena.

**Interface Phenomena (M ∩ D)**: The points where the machine and problem domain interact. These include shared information, control signals, and physical connections.

---

## Problem Frame Types

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    PROBLEM FRAME TYPES                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  1. INFORMATION PROBLEM FRAME                                   │    │
│  │                                                                 │    │
│  │  Context: Capturing, storing, and retrieving information        │    │
│  │                                                                 │    │
│  │         ┌─────────────┐                                         │    │
│  │         │   INPUT     │                                         │    │
│  │         │  DOMAIN     │                                         │    │
│  │         └──────┬──────┘                                         │    │
│  │                │                                                │    │
│  │                ▼                                                │    │
│  │         ┌─────────────┐    ┌─────────────┐                     │    │
│  │         │   MACHINE   │───►│   OUTPUT    │                     │    │
│  │         │             │    │  DOMAIN     │                     │    │
│  │         └─────────────┘    └─────────────┘                     │    │
│  │                                                                 │    │
│  │  Examples: Database systems, document management, archives      │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  2. CONTROL PROBLEM FRAME                                       │    │
│  │                                                                 │    │
│  │  Context: Controlling physical processes or devices              │    │
│  │                                                                 │    │
│  │         ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │    │
│  │         │   CONTROLLED│───►│   MACHINE   │───►│  CONTROLLER │   │    │
│  │         │   DOMAIN    │    │             │    │  DOMAIN     │   │    │
│  │         └─────────────┘    └─────────────┘    └─────────────┘   │    │
│  │                                                                 │    │
│  │  Examples: Embedded systems, industrial automation, robotics    │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  3. TRANSFORMATION PROBLEM FRAME                                │    │
│  │                                                                 │    │
│  │  Context: Transforming input data to output data                │    │
│  │                                                                 │    │
│  │         ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │    │
│  │         │    INPUT    │───►│   MACHINE   │───►│   OUTPUT    │   │    │
│  │         │    DATA     │    │             │    │    DATA     │   │    │
│  │         └─────────────┘    └─────────────┘    └─────────────┘   │    │
│  │                                                                 │    │
│  │  Examples: Compilers, data converters, report generators        │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  4. REQUIREMENTS PROBLEM FRAME                                  │    │
│  │                                                                 │    │
│  │  Context: Enforcing rules and constraints                        │    │
│  │                                                                 │    │
│  │         ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │    │
│  │         │  REQUIRED   │    │   MACHINE   │    │   BINDING   │   │    │
│  │         │  BEHAVIOR   │    │             │    │   DOMAIN    │   │    │
│  │         └─────────────┘    └─────────────┘    └─────────────┘   │    │
│  │                                                                 │    │
│  │  Examples: Validation systems, rule engines, compliance tools   │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  5. WORKFLOW PROBLEM FRAME                                      │    │
│  │                                                                 │    │
│  │  Context: Coordinating activities across multiple actors        │    │
│  │                                                                 │    │
│  │         ┌───────────────────────────────────────────────────┐   │    │
│  │         │              WORKFLOW MACHINE                     │   │    │
│  │         │                                                   │   │    │
│  │         │   State: Ready ──► Active ──► Completed ──► Error│   │    │
│  │         │                                                   │   │    │
│  │         └───────────────────────────────────────────────────┘   │    │
│  │                                                                 │    │
│  │  Examples: Business process management, orchestration systems   │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Problem Frames Analysis Process

```
┌─────────────────────────────────────────────────────────────────────────┐
│              PROBLEM FRAMES ANALYSIS WORKFLOW                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│  │   BOUND     │───►│   IDENTIFY  │───►│   MAP TO    │                 │
│  │   PROBLEM   │    │   FRAMES    │    │   PATTERNS  │                 │
│  │             │    │             │    │             │                 │
│  │  Define the │    │  Determine  │    │  Apply      │                 │
│  │  scope and  │    │  which      │    │  established│                 │
│  │  boundaries │    │  frame(s)   │    │  solutions  │                 │
│  │             │    │  apply      │    │             │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│         │                  │                  │                         │
│         ▼                  ▼                  ▼                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│  │  CONTEXT     │    │  DECOMPOSE  │    │  SPECIFY    │                 │
│  │  DIAGRAM     │    │  PROBLEM    │    │  REQUIRE-   │                 │
│  │              │    │             │    │  MENTS      │                 │
│  │  Visual      │    │  Break into │    │  Define     │                 │
│  │  representation│   │  sub-prob-  │    │  detailed   │                 │
│  │  of the      │    │  lems       │    │  require-   │                 │
│  │  problem     │    │             │    │  ments      │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│                                                                     │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│  │ VERIFY       │───►│   VALIDATE  │───►│   ITERATE   │                 │
│  │ SOLUTION     │    │   WITH      │    │             │                 │
│  │              │    │   STAKE-    │    │             │                 │
│  │  Ensure      │    │   HOLDERS   │    │             │                 │
│  │  solution    │    │             │    │             │                 │
│  │  meets       │    │  Review     │    │             │                 │
│  │  requirements│    │  analysis   │    │             │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Step 1: Bound the Problem
Define the scope and boundaries of the problem. Identify what is included and what is excluded from consideration.

### Step 2: Identify Frames
Determine which problem frame(s) apply to the situation. Some problems may combine multiple frames.

### Step 3: Create Context Diagram
Build a visual representation showing the machine, problem domains, and their interfaces.

### Step 4: Decompose the Problem
Break complex problems into smaller sub-problems, each corresponding to a frame or combination of frames.

### Step 5: Specify Requirements
For each sub-problem, specify requirements in terms of observable phenomena and domain properties.

### Step 6: Verify Solution
Ensure that proposed solutions address all specified requirements and respect domain constraints.

### Step 7: Validate with Stakeholders
Review the analysis with stakeholders to ensure understanding and capture missing requirements.

---

## Context Diagram Notation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CONTEXT DIAGRAM NOTATION                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌───────────────────┐                                                  │
│  │                   │                                                  │
│  │   MACHINE (M)     │        Rectangular box = Machine                 │
│  │                   │        (System being built)                      │
│  │                   │                                                  │
│  └───────────────────┘                                                  │
│                                                                         │
│      ┌───────────────────┐                                              │
│      │                   │                                              │
│      │   DOMAIN (D)      │        Rounded box = Domain                  │
│      │                   │        (Problem domain)                      │
│      │                   │                                              │
│      └───────────────────┘                                              │
│                                                                         │
│      ┌───────────────────┐                                              │
│      │                   │                                              │
│      │  REQUIREMENTS (R) │        Dashed box = Requirements             │
│      │                   │        (What must be achieved)               │
│      │                   │                                              │
│      └───────────────────┘                                              │
│                                                                         │
│      ════════════════════════════════════════════════════════          │
│                     (Shared phenomena)                                  │
│      Simple line = Shared phenomenon between M and D                    │
│                                                                         │
│      ───────────────────────────────────────────────────────           │
│                     (Connection)                                        │
│      Arrow = Causal connection or data flow                            │
│                                                                         │
│      .  .  .  .  .  .  .  .  .  .  .  .  .  .  .  .  .  .  .           │
│                     (Annotation)                                        │
│      Dots = Domain properties constrain requirements                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Problem Frames Example: Order Processing System

### Step 1: Identify the Problem Frame

```
┌─────────────────────────────────────────────────────────────────────────┐
│              PROBLEM FRAME: INFORMATION + WORKFLOW                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  This problem combines the Information Problem Frame (capturing and      │
│  storing order data) with the Workflow Problem Frame (managing the       │
│  order lifecycle through multiple states).                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Step 2: Context Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ORDER PROCESSING CONTEXT DIAGRAM                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│                              ┌───────────────────────┐                   │
│                              │                       │                   │
│                              │     ORDER SYSTEM      │                   │
│                              │         (M)           │                   │
│                              │                       │                   │
│                              └───────────────────────┘                   │
│                                       │                                 │
│                 ┌─────────────────────┼─────────────────────┐           │
│                 │                     │                     │           │
│                 ▼                     ▼                     ▼           │
│     ┌───────────────────┐   ┌───────────────────┐  ┌─────────────────┐ │
│     │                   │   │                   │  │                 │ │
│     │   CUSTOMER        │   │    INVENTORY      │  │  PAYMENT        │ │
│     │   DOMAIN (D1)     │   │    DOMAIN (D2)    │  │  DOMAIN (D3)    │ │
│     │                   │   │                   │  │                 │ │
│     │ • Order requests  │   │ • Stock levels    │  │ • Payment       │ │
│     │ • Customer info   │   │ • Product catalog │  │   processing    │ │
│     │ • Delivery addr   │   │ • Warehousing     │  │ • Transactions  │ │
│     └───────────────────┘   └───────────────────┘  └─────────────────┘ │
│                 │                     │                     │           │
│                 └─────────────────────┼─────────────────────┘           │
│                                       │                                 │
│                                       ▼                                 │
│                              ┌───────────────────────┐                   │
│                              │                       │                   │
│                              │    REQUIREMENTS (R)   │                   │
│                              │                       │                   │
│                              │  • Valid orders must  │                   │
│                              │    be fulfilled       │                   │
│                              │  • Payment before     │                   │
│                              │    shipping           │                   │
│                              │  • Stock reserved     │                   │
│                              │    for orders         │                   │
│                              │                       │                   │
│                              └───────────────────────┘                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Step 3: Problem Decomposition

```markdown
# Problem Decomposition: Order Processing System

## Sub-problem 1: Order Capture (Information Frame)
**Frame**: Information Problem
**Phenomena**: 
- Customer submits order (D1 → M)
- System stores order data (M)
- Order confirmation returned (M → D1)

**Requirements**:
- R1: Every order must capture customer, items, and delivery address
- R2: Orders must be assigned unique identifiers
- R3: Order data must be durably stored

## Sub-problem 2: Order Validation (Requirements Frame)
**Frame**: Requirements Problem
**Phenomena**:
- Machine validates order against business rules (M)
- Validation results returned (M)

**Requirements**:
- R4: Order must have at least one item
- R5: Items must be in stock
- R6: Customer must have valid payment method

## Sub-problem 3: Inventory Reservation (Control Frame)
**Frame**: Control Problem
**Phenomena**:
- Machine receives order (M)
- Machine controls inventory domain (M → D2)
- Inventory reserved (D2)

**Requirements**:
- R7: Stock must be reserved within 10 seconds of valid order
- R8: Reservation must be atomic with order creation
- R9: Stock level must decrease when reserved

## Sub-problem 4: Order Workflow (Workflow Frame)
**Frame**: Workflow Problem
**Phenomena**:
- Order transitions through states (M)

**States**:
- PENDING → VALIDATED → PAID → PROCESSING → SHIPPED → DELIVERED

**Requirements**:
- R10: Order must pass validation before payment
- R11: Order must be paid before shipping
- R12: Tracking info must be provided when shipped

## Sub-problem 5: Payment Processing (Transformation Frame)
**Frame**: Transformation Problem
**Phenomena**:
- Payment details input (D1 → M)
- Payment processed (M)
- Payment result output (M → D1, D3)

**Requirements**:
- R13: Payment must be authorized within 5 seconds
- R14: Failed payments must cancel order
- R15: Transaction records must be retained for 7 years
```

### Step 4: Domain Properties

```markdown
# Domain Properties (Constraints on Requirements)

## Customer Domain (D1)
- DP1: Customer address must be valid (format and deliverable)
- DP2: Customer account must be in good standing
- DP3: Each customer has unique identifier

## Inventory Domain (D2)
- DP4: Stock levels are accurate within 5 seconds
- DP5: Products have unique SKUs
- DP6: Stock cannot go negative

## Payment Domain (D3)
- DP7: Payment gateway is available 99.9% of time
- DP8: Transactions are processed within 30 seconds timeout
- DP9: Payment cards follow PCI-DSS requirements

## Derived Constraints
- DC1: Order total = sum(item prices) + shipping - discounts
- DC2: Available stock = physical stock - reserved orders
- DC3: Order value cannot exceed customer credit limit
```

---

## Problem Frames in TypeScript

```typescript
// problem-frames.types.ts

/**
 * Represents a machine domain in the Problem Frames model.
 * The machine is the software system being developed.
 */
interface Machine {
  name: string;
  responsibilities: string[];
  interfaces: Interface[];
}

/**
 * Represents a problem domain.
 * Domains are parts of the world that exhibit the problem.
 */
interface Domain {
  name: string;
  type: 'input' | 'output' | 'controlled' | 'bypass';
  properties: DomainProperty[];
  phenomena: Phenomenon[];
}

/**
 * A shared phenomenon between machine and domain.
 * These are the observable points of interaction.
 */
interface Phenomenon {
  name: string;
  owner: 'machine' | 'domain' | 'shared';
  type: 'event' | 'state' | 'command' | 'data';
  description: string;
}

/**
 * A requirement that the machine must satisfy.
 * Requirements are properties of the machine in context.
 */
interface Requirement {
  id: string;
  description: string;
  sourceRequirement: string;
  domainProperties: string[];
  phenomena: string[];
  priority: 'must' | 'should' | 'could';
}

/**
 * A property of the domain that constrains solutions.
 * Domain properties are given and cannot be changed by the machine.
 */
interface DomainProperty {
  id: string;
  description: string;
  certainty: 'certain' | 'likely' | 'assumed';
  source: string;
}

// Problem Frame Types
type ProblemFrameType = 
  | 'information'
  | 'control'
  | 'transformation'
  | 'requirements'
  | 'workflow'
  | 'reactive';

/**
 * A problem frame identifies the structure of a sub-problem.
 */
interface ProblemFrame {
  type: ProblemFrameType;
  machine: Machine;
  domains: Domain[];
  requirements: Requirement[];
  domainProperties: DomainProperty[];
  contextDiagram: string;
}

/**
 * Problem Analysis encapsulates the complete analysis of a problem.
 */
class ProblemFrameAnalysis {
  private frames: Map<string, ProblemFrame> = new Map();
  private domains: Map<string, Domain> = new Map();
  private requirements: Map<string, Requirement> = new Map();
  private domainProperties: Map<string, DomainProperty> = new Map();

  addDomain(domain: Domain): void {
    this.domains.set(domain.name, domain);
  }

  addFrame(frame: ProblemFrame): void {
    this.frames.set(frame.type, frame);
  }

  addRequirement(requirement: Requirement): void {
    this.requirements.set(requirement.id, requirement);
  }

  addDomainProperty(property: DomainProperty): void {
    this.domainProperties.set(property.id, property);
  }

  getFrame(type: ProblemFrameType): ProblemFrame | undefined {
    return this.frames.get(type);
  }

  validateRequirements(): { valid: boolean; issues: string[] } {
    const issues: string[] = [];

    for (const req of this.requirements.values()) {
      // Check that referenced domain properties exist
      for (const dpId of req.domainProperties) {
        if (!this.domainProperties.has(dpId)) {
          issues.push(`Requirement ${req.id} references non-existent domain property ${dpId}`);
        }
      }

      // Check that referenced phenomena exist
      for (const phenomenon of req.phenomena) {
        const phenomenonExists = Array.from(this.domains.values()).some(
          d => d.phenomena.some(p => p.name === phenomenon)
        );
        if (!phenomenonExists) {
          issues.push(`Requirement ${req.id} references non-existent phenomenon ${phenomenon}`);
        }
      }
    }

    return { valid: issues.length === 0, issues };
  }

  generateContextDiagram(): string {
    let diagram = 'graph TD\n';
    
    // Add machine
    diagram += '    M[Order Processing System]\n';
    
    // Add domains
    for (const [name, domain] of this.domains) {
      const className = domain.type.charAt(0).toUpperCase() + domain.type.slice(1);
      diagram += `    D${name}[${name}]\n`;
      diagram += `    M -.-> D${name}\n`;
    }

    return diagram;
  }
}

// Example Analysis: Order Processing System
function createOrderProcessingAnalysis(): ProblemFrameAnalysis {
  const analysis = new ProblemFrameAnalysis();

  // Add domains
  analysis.addDomain({
    name: 'Customer',
    type: 'input',
    properties: [
      { id: 'DP1', description: 'Customer has unique ID', certainty: 'certain', source: 'System' },
      { id: 'DP2', description: 'Customer address is valid format', certainty: 'certain', source: 'Input validation' },
    ],
    phenomena: [
      { name: 'order_request', owner: 'domain', type: 'event', description: 'Customer submits order' },
      { name: 'customer_info', owner: 'domain', type: 'data', description: 'Customer details' },
    ],
  });

  analysis.addDomain({
    name: 'Inventory',
    type: 'controlled',
    properties: [
      { id: 'DP3', description: 'Stock levels are accurate', certainty: 'assumed', source: 'Inventory system' },
      { id: 'DP4', description: 'Products have unique SKUs', certainty: 'certain', source: 'Catalog' },
    ],
    phenomena: [
      { name: 'stock_level', owner: 'domain', type: 'state', description: 'Current stock' },
      { name: 'reserve_stock', owner: 'shared', type: 'command', description: 'Reservation request' },
    ],
  });

  // Add requirements
  analysis.addRequirement({
    id: 'R1',
    description: 'System must capture complete order information',
    sourceRequirement: 'Order capture',
    domainProperties: ['DP1', 'DP2'],
    phenomena: ['order_request', 'customer_info'],
    priority: 'must',
  });

  analysis.addRequirement({
    id: 'R2',
    description: 'System must reserve stock within 10 seconds',
    sourceRequirement: 'Inventory management',
    domainProperties: ['DP3'],
    phenomena: ['stock_level', 'reserve_stock'],
    priority: 'must',
  });

  return analysis;
}
```

---

## Problem Frames Best Practices

### 1. Start with Problem, Not Solution
Focus on understanding the problem before designing solutions. The frame analysis helps ensure you're solving the right problem.

### 2. Separate Machine from Domain
Clearly distinguish between what the machine will do and what exists in the domain. This prevents confusing requirements with implementation.

### 3. Identify Domain Properties
Explicitly capture domain properties as they constrain what the machine can achieve. Don't assume domain properties can be changed.

### 4. Use Multiple Frames
Most real problems combine multiple frames. Decompose into sub-problems, each matching a frame type.

### 5. Draw Context Diagrams
Visual representations help communicate the analysis and identify missing elements.

### 6. Validate with Stakeholders
Review the analysis with domain experts to ensure understanding is correct.

### 7. Trace Requirements to Phenomena
Each requirement should map to observable phenomena to ensure testability.

### 8. Document Constraints Explicitly
Domain properties and derived constraints should be explicitly documented.

### 9. Iterate the Analysis
Problem understanding deepens over time. Update the analysis as understanding grows.

### 10. Connect to Design
The problem frame analysis should inform design decisions, not be a separate deliverable.

---

## Problem Frames Anti-Patterns

### 1. Solution-First Thinking
Starting with what you think the solution should be instead of understanding the problem.

### 2. Ignoring Domain Properties
Treating domain properties as changeable when they are constraints on the solution.

### 3. Over-Abstracting
Creating abstract frames that don't correspond to actual problem structures.

### 4. Single Frame for Complex Problems
Trying to fit complex problems into a single frame when decomposition is needed.

### 5. Unverified Phenomena
Specifying phenomena that don't correspond to observable events or states.

### 6. Mixing Requirements and Design
Expressing requirements in terms of implementation rather than desired behavior.

### 7. Incomplete Frame Analysis
Stopping after identifying the frame without completing the full analysis.

### 8. No Stakeholder Review
Conducting analysis without validation from domain experts.

### 9. Forgetting Interface Phenomena
Missing the shared phenomena between machine and domain that define the interface.

### 10. No Traceability
Failing to trace requirements to phenomena, domain properties, and ultimately test cases.

---

## Problem Frames in AI-Assisted Development

When working with AI assistants for Problem Frames analysis, follow these practices:

### 1. Describe the Problem Context
Provide the AI with business context and ask for help identifying which frames apply.

### 2. Generate Initial Decomposition
Ask AI to help decompose complex problems into sub-problems that match frames.

### 3. Identify Missing Phenomena
Have AI review your analysis to suggest phenomena that might be missing.

### 4. Document Domain Properties
Ask AI to help identify domain properties based on domain knowledge.

### 5. Generate Requirements
Transform problem descriptions into formal requirements expressed in terms of phenomena.

### 6. Validate Completeness
Use AI to verify that all aspects of the problem are covered by the analysis.

### 7. Create Visual Diagrams
Ask AI to generate context diagram descriptions or diagrams from your analysis.

### 8. Connect to Implementation
Work with AI to translate problem frame requirements into technical specifications.

---

## References and Further Reading

1. Jackson, Michael. "Software Requirements & Specifications: A Lexicon of Practice, Principles and Prejudices." Addison-Wesley, 1995.
2. Jackson, Michael. "Problem Frames: Analysing and Structuring Software Development Problems." Addison-Wesley, 2001.
3. "The Problem Frames Approach." Requirements Engineering Community.
4. "Domain-Driven Design and Problem Frames." Eric Evans.
5. "Structured Analysis and System Specification." Tom DeMarco.
