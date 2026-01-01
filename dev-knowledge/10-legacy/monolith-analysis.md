# Monolith Analysis

This document provides a comprehensive guide for analyzing monolithic applications to identify refactoring opportunities, understand dependencies, and plan migration strategies.

---

## 1. Monolith Characteristics

### 1.1 Defining Monolithic Architecture

A monolithic application is a single unified codebase where all components are interconnected and interdependent. While simpler to develop initially, monoliths often become problematic as they grow.

**Common Monolith Symptoms:**
- Long build times (> 5 minutes)
- Slow test suites (> 30 minutes)
- Fear of making changes ("the code is fragile")
- Difficult onboarding for new developers
- Tight coupling between unrelated features
- Single point of failure for the entire system
- Difficulty scaling specific components

### 1.2 Types of Monoliths

| Type | Characteristics | Refactoring Difficulty |
|------|-----------------|----------------------|
| **Modular Monolith** | Well-organized modules with clear boundaries | Low - Can refactor incrementally |
| **Big Ball of Mud** | No clear structure, tangled dependencies | High - Requires extensive analysis |
| **Distributed Monolith** | Multiple deployments but tightly coupled | Very High - Complex dependency web |
| **Legacy Monolith** | Old technology stack, minimal tests | High - Requires modernization first |

---

## 2. Analysis Framework

### 2.1 Static Code Analysis

Use tools to understand the codebase structure:

```bash
# Dependency analysis
find . -name "*.java" -o -name "*.ts" | xargs wc -l

# Identify large files (potential god classes)
find . -name "*.java" -exec wc -l {} + | sort -rn | head -20

# Find circular dependencies
# Java: Use Structure101 or JDeodorant
# TypeScript: Use madge or dependency-cruiser

# Complexity analysis
# Java: Use SonarQube or CodeComplexity
# TypeScript: Use ts-complex or nico
```

### 2.2 Dependency Mapping

**Steps to Map Dependencies:**

1. **Identify all imports/exports**
```bash
# TypeScript
grep -r "import " --include="*.ts" . | cut -d' ' -f2 | sort | uniq

# Java
grep -r "import " --include="*.java" . | cut -d' ' -f2 | sort | uniq
```

2. **Build dependency matrix**

| Component | Depends On | Dependents | Type |
|-----------|------------|------------|------|
| UserService | UserRepository, EmailService | OrderService, PaymentService | Domain |
| OrderService | UserService, ProductRepository | CheckoutController | Application |
| ProductRepository | Database | OrderService, InventoryService | Infrastructure |

3. **Identify coupling patterns**
- **Tight coupling**: A changes always requires B changes
- **Temporal coupling**: B must run after A
- **Common coupling**: Multiple modules share global state

### 2.3 Bounded Context Identification

Using Domain-Driven Design principles, identify potential bounded contexts:

```typescript
// Monolithic code often has these patterns:
class Order {  // Domain concept - potential Aggregate Root
  private items: Map<string, Item>;  // Could be separate Aggregate
  private customer: Customer;  // Could be separate Aggregate
  
  addItem() { /* ... */ }
  processPayment() { /* ... */ }  // Different domain concept
  generateInvoice() { /* ... */ }  // Could be separate use case
}

// After analysis, identify bounded contexts:
context Order {
  aggregate Order
  aggregate Customer
  aggregate Item
}

context Billing {
  // Invoice generation logic
}
```

---

## 3. Risk Assessment

### 3.1 Technical Debt Inventory

| Category | Assessment Questions | Score (1-5) |
|----------|---------------------|-------------|
| Code Quality | Duplication, complexity, readability | |
| Test Coverage | Unit, integration, e2e coverage | |
| Documentation | README, API docs, architecture docs | |
| Dependencies | Outdated packages, security vulnerabilities | |
| Build/Deploy | Build time, deployment frequency | |

### 3.2 Dependency Risk Matrix

| Component | Stability | Change Frequency | Risk Level |
|-----------|-----------|-----------------|------------|
| Core Domain | High | Low | Low |
| Business Rules | Medium | Medium | Medium |
| External Integrations | Low | High | High |
| UI Layer | Medium | High | Medium |
| Data Access | High | Low | Low |

### 3.3 Refactoring Readiness Checklist

| Readiness Factor | Status | Notes |
|------------------|--------|-------|
| Comprehensive test suite | ☐ | |
| CI/CD pipeline in place | ☐ | |
| Feature flags available | ☐ | |
| Database migration strategy | ☐ | |
| Rollback plan defined | ☐ | |
| Team agreement on approach | ☐ | |
| Stakeholder buy-in | ☐ | |

---

## 4. Analysis Outputs

### 4.1 Monolith Health Score

```
HEALTH SCORE = (Code Quality + Test Coverage + Documentation + Deployability) / 4

Score Breakdown:
- 4.0 - 5.0: Healthy - Can refactor incrementally
- 2.5 - 3.9: Needs Work - Address critical issues first
- 1.0 - 2.4: Critical - Requires significant investment
```

### 4.2 Component Catalog

Create a catalog of all major components:

```markdown
## Component: Order

**Type**: Aggregate Root
**Package**: com.example.orders.entity
**Lines**: 450
**Dependencies**:
  - Item (internal)
  - Customer (internal)
  - DateTime (stdlib)

**Affected By**:
  - OrderController (adapter)
  - OrderService (application)

**Test Coverage**: 85%

**Complexity Score**: Medium
**Coupling Score**: High (tied to Task and Project)

**Recommendations**:
- Extract Task to separate aggregate
- Extract Project to separate aggregate
- Create repository interfaces in domain
```

### 4.3 Extraction Priority Matrix

| Component | Business Value | Refactor Effort | Dependencies | Priority |
|-----------|---------------|-----------------|--------------|----------|
| Order | High | Medium | Low | 1 |
| Customer | High | Low | Low | 2 |
| UserService | Medium | High | Medium | 3 |
| Billing | Low | Medium | High | 4 |

---

## 5. Analysis Techniques

### 5.1 Dependency Graph Generation

```bash
# TypeScript - using madge
npx madge --image graph.svg src/
npx madge --circular src/

# Java - using Structure101 or similar
# Export dependency matrix

# General - using graphviz
find src -name "*.ts" | while read f; do
  grep -h "^import " "$f" 2>/dev/null | \
  sed 's/import //; s/from//' | \
  sed "s/[';]//g" | while read dep; do
    basename "$f" .ts | tr -d '\n'
    echo " -> $dep"
  done
done > deps.txt
dot -Tpng deps.txt -o dependency-graph.png
```

### 5.2 Code Churn Analysis

Identify frequently changed files (indicates high volatility):

```bash
# Git history analysis
git log --name-only --pretty=format: | \
  sort | uniq -c | sort -rn | head -20

# Files with most contributors
git log --format="%an" -- "**/*.java" | \
  sort | uniq -c | sort -rn | head -20
```

### 5.3 Hotspot Analysis

Identify code hotspots that need attention:

```
Hotspot Criteria:
- High complexity (cyclomatic > 10)
- High churn (frequently changed)
- Low test coverage
- High bug count
- High coupling
```

---

## 6. Database Analysis

### 6.1 Schema Analysis

| Check | Status | Notes |
|-------|--------|-------|
| Identify large tables | ☐ | |
| Find circular FK dependencies | ☐ | |
| Identify shared tables | ☐ | |
| Analyze query patterns | ☐ | |
| Document data ownership | ☐ | |

### 6.2 Data Migration Considerations

| Factor | Assessment |
|--------|------------|
| Data volume | GB/TB |
| Transaction requirements | ACID vs eventual |
| Migration window | Available downtime |
| Rollback strategy | Point-in-time recovery |

---

## 7. AI-Assisted Analysis

### 7.1 AI Analysis Prompts

```markdown
# Analyze monolith for extraction opportunities

Analyze the codebase in ./src and provide:

1. **Dependency Analysis**
   - Identify main domain concepts (entities, aggregates)
   - Map dependencies between modules
   - Identify tight coupling patterns

2. **Bounded Context Candidates**
   - Group related code into potential bounded contexts
   - Identify shared kernels
   - Suggest context map

3. **Extraction Recommendations**
   - Rank components by extraction ease
   - Identify prerequisites for each extraction
   - Estimate effort for each extraction

4. **Risk Assessment**
   - Identify high-risk extractions
   - Suggest mitigation strategies
   - Identify critical path dependencies

Focus on: domain/ and usecase/ directories.
```

### 7.2 Automated Analysis Tools

| Tool | Purpose | Language |
|------|---------|----------|
| Structure101 | Dependency analysis | Java |
| SonarQube | Code quality | Multi |
| Dependency Cruiser | Dependency visualization | TypeScript |
| Madge | Module dependency analysis | TypeScript |
| CodeScene | Hotspot analysis | Multi |

---

## 8. Documentation Templates

### 8.1 Monolith Analysis Report Template

```markdown
# Monolith Analysis Report: [Application Name]

## Executive Summary
- Total lines of code: [X]
- Number of modules: [X]
- Test coverage: [X]%
- Health score: [X]/5

## Architecture Overview
[Include architecture diagram]

## Component Inventory
| Component | Type | LOC | Dependencies | Priority |
|-----------|------|-----|--------------|----------|
| | | | | |

## Dependency Analysis
[Include dependency graph]

## Bounded Contexts
### Context 1: [Name]
- Domain concepts: [List]
- Dependencies: [List]
- Extraction effort: Low/Medium/High

### Context 2: [Name]
- ...

## Recommendations
1. [Priority 1]
2. [Priority 2]
3. [Priority 3]

## Risk Register
| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| | | | |

## Next Steps
1. [Immediate action]
2. [Short-term action]
3. [Long-term action]
```

---

## 9. Anti-Patterns to Identify

| Anti-Pattern | Description | Impact |
|--------------|-------------|--------|
| God Class | Class that knows too much | High coupling |
| God Module | Module with too many responsibilities | Hard to extract |
| Circular Dependency | A -> B -> C -> A | Impossible to separate |
| Feature Envy | Method uses more data from other class | Bad encapsulation |
| Data Clumps | Same group of variables appear together | Missing abstraction |
| Primitive Obsession | Using primitives instead of objects | Hard to validate |

---

## 10. References

1. Martin Fowler. "Refactoring: Improving the Design of Existing Code"
2. Eric Evans. "Domain-Driven Design: Tackling Complexity in the Heart of Software"
3. Simon Brown. "Modular Monoliths"
4. refactor-to-ca-analysis.md - Real-world 14-step refactoring journey
5. doc/clean-architecture-refactoring-journey.md - Step-by-step refactoring guide
6. ref/engineering/METHODOLOGIES.md - Development methodologies

---

## Appendix A: Monolith Types Reference

### Type 1: Modular Monolith

**Characteristics:**
- Well-organized modules with clear boundaries
- Clear package structure by domain or feature
- Low coupling between modules
- High cohesion within modules

**Refactoring Difficulty:** Low

**Indicators:**
- Modules can be identified by package structure
- Dependencies between modules are explicit
- Minimal circular dependencies
- Tests can run per module

### Type 2: Big Ball of Mud

**Characteristics:**
- No clear structure or organization
- Tangled dependencies throughout
- "Spaghetti code" with random imports
- Fear of making changes

**Refactoring Difficulty:** High

**Indicators:**
- All code in one package or random packages
- Classes import from many other classes
- Circular dependencies everywhere
- No clear domain boundaries

### Type 3: Distributed Monolith

**Characteristics:**
- Multiple deployments but tightly coupled
- Services that should be independent but share databases
- Changes in one service break others

**Refactoring Difficulty:** Very High

**Indicators:**
- Multiple deployable units but shared database
- Foreign key relationships across service boundaries
- Synchronous calls between all services

### Type 4: Legacy Monolith

**Characteristics:**
- Old technology stack
- Minimal or no automated tests
- Business-critical but hard to maintain
- Knowledge gap (original developers left)

**Refactoring Difficulty:** High

**Indicators:**
- Framework is several versions behind
- No CI/CD or manual deployment
- Tests are integration-only or missing
- Documentation is outdated or missing

---

## Appendix B: Analysis Checklist

### Pre-Analysis Checklist

- [ ] Identify all code files and their locations
- [ ] Determine technology stack and versions
- [ ] Map out build and deployment processes
- [ ] Identify key stakeholders and domain experts
- [ ] Gather existing documentation
- [ ] Understand business context and priorities
- [ ] Set up static analysis tools
- [ ] Establish baseline metrics

### Analysis Checklist

- [ ] Run static analysis (complexity, dependencies)
- [ ] Generate dependency graphs
- [ ] Identify bounded contexts
- [ ] Map coupling patterns
- [ ] Identify aggregate candidates
- [ ] Document API surfaces
- [ ] Assess test coverage
- [ ] Evaluate database dependencies
- [ ] Identify external integrations
- [ ] Profile performance bottlenecks

### Decision Checklist

- [ ] Define target architecture
- [ ] Prioritize extraction candidates
- [ ] Assess risks and dependencies
- [ ] Create migration roadmap
- [ ] Set success criteria
- [ ] Establish rollback plan
- [ ] Define timeline and milestones
- [ ] Allocate resources
