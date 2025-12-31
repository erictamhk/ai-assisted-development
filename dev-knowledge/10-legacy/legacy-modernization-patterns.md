# Legacy Modernization Patterns

This document provides comprehensive patterns and strategies for modernizing legacy applications, transitioning from monoliths to Clean Architecture, and managing technical debt.

---

## Part 1: Technical Debt Assessment

### 1.1 Identifying Technical Debt

Technical debt manifests in many forms. Identifying it is the first step toward managing it.

**Code-Level Debt:**
| Indicator | Description | Impact |
|-----------|-------------|--------|
| Complex methods | Methods > 50 lines or high cyclomatic complexity | Hard to understand and modify |
| Duplicate code | Repeated logic across files | Maintenance burden, inconsistencies |
| Large classes | Classes > 300 lines | God class anti-pattern |
| Missing tests | Low code coverage | Risk of regressions |
| Outdated code | Deprecated APIs, old patterns | Security vulnerabilities |

**Architecture-Level Debt:**
| Indicator | Description | Impact |
|-----------|-------------|--------|
| Tight coupling | Components can't be separated | Impossible to extract features |
| Circular dependencies | A → B → C → A | Cascade of changes |
| Missing layers | No clear separation of concerns | Spaghetti code |
| Single point of failure | No redundancy | System fragility |
| Performance bottlenecks | No scalability path | User experience issues |

**Process-Level Debt:**
| Indicator | Description | Impact |
|-----------|-------------|--------|
| No CI/CD | Manual deployments | Deployment errors |
| Missing documentation | No up-to-date docs | Knowledge loss |
| Long feedback loop | Delayed testing | Quality issues |
| No code review | Unchecked changes | Quality and security risks |

### 1.2 Technical Debt Quantification

**Debt Scoring Matrix:**

| Category | Weight | Score (1-10) |
|----------|--------|--------------|
| Code Complexity | 25% | |
| Test Coverage | 20% | |
| Architecture Health | 25% | |
| Security Posture | 15% | |
| Documentation | 15% | |

**Total Debt Score:** _____ / 10

**Interpretation:**
- 1-3: Healthy codebase, minor debt
- 4-5: Moderate debt, address soon
- 6-7: Significant debt, prioritize remediation
- 8-10: Critical debt, immediate action needed

### 1.3 Creating a Debt Backlog

```
Technical Debt Backlog Template

| ID | Debt Item | Category | Impact | Effort | Priority | Status |
|----|-----------|----------|--------|--------|----------|--------|
| TD-001 | Method X is too complex | Code | High | 2 days | P1 | TODO |
| TD-002 | Missing tests for UserService | Test | High | 3 days | P1 | TODO |
```

---

## Part 2: Strangler Fig Pattern

### 2.1 Pattern Overview

The Strangler Fig Pattern enables incremental migration by gradually replacing specific functionality with new implementations.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STRANGLER FIG PATTERN                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│    LEGACY MONOLITH                                                      │
│    ┌─────────────────────────────────────────────────────────────┐     │
│    │                     [Feature A]                              │     │
│    │                     [Feature B]                              │     │
│    │                     [Feature C]                              │     │
│    │                     [Feature D]                              │     │
│    │                     [Feature E]                              │     │
│    └─────────────────────────────────────────────────────────────┘     │
│                              │                                          │
│                              │                                          │
│              ┌───────────────┼───────────────┐                          │
│              │               │               │                          │
│              ▼               ▼               ▼                          │
│      ┌───────────┐   ┌───────────┐   ┌───────────┐                     │
│      │NEW A      │   │NEW B      │   │NEW C      │                     │
│      │(replaces  │   │(replaces  │   │(replaces  │                     │
│      │ Feature A)│   │ Feature B)│   │ Feature C)│                     │
│      └───────────┘   └───────────┘   └───────────┘                     │
│              │               │               │                          │
│              └───────────────┴───────────────┘                          │
│                              │                                          │
│                              ▼                                          │
│                    ┌───────────────────┐                                │
│                    │NEW MICROSERVICE(S)│                                │
│                    │CONTAINING A, B, C │                                │
│                    └───────────────────┘                                │
│                                                                         │
│    LEGACY MONOLITH (Shrinking)                                          │
│    ┌─────────────────────────────┐                                      │
│    │          [Feature D]        │                                      │
│    │          [Feature E]        │                                      │
│    └─────────────────────────────┘                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Implementation Steps

**Phase 1: Identify Candidates**
1. Map all features and dependencies
2. Identify loosely coupled features
3. Prioritize by extraction difficulty
4. Select low-risk starting point

**Phase 2: Create Facade**
```typescript
// LegacyFacade.ts
class LegacyFacade {
  private newService: NewService;
  private legacyService: LegacyService;

  async processFeatureA(input: Input): Promise<Output> {
    // Route to new or legacy implementation
    return this.newService.process(input);
  }

  async processFeatureB(input: Input): Promise<Output> {
    // Still using legacy
    return this.legacyService.process(input);
  }
}
```

**Phase 3: Incremental Replacement**
1. Implement new feature alongside old
2. Use feature flags to control rollout
3. Monitor metrics and errors
4. Gradually increase traffic to new implementation

**Phase 4: Decommission Legacy**
1. Verify all traffic moved to new
2. Remove legacy code
3. Update documentation
4. Clean up feature flags

### 2.3 Anti-Patterns to Avoid

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| Big Bang Rewrite | Replacing everything at once | Incremental approach |
| Database Coupling | New and old sharing database | Separate databases |
| Inconsistent Interfaces | Different API styles | Contract testing |
| No Rollback Plan | No way to revert | Feature flags, rollback strategy |

---

## Part 3: Database Migration Strategies

### 3.1 Schema Evolution

**Backward-Compatible Changes:**
- Adding nullable columns
- Adding new tables
- Creating new indexes
- Adding enum values

**Breaking Changes:**
- Removing columns
- Changing column types
- Renaming columns
- Changing constraints

### 3.2 Safe Migration Pattern

```sql
-- 1. Add new column (nullable)
ALTER TABLE users ADD COLUMN new_email VARCHAR(255);

-- 2. Migrate data (in batches)
UPDATE users SET new_email = email WHERE new_email IS NULL LIMIT 1000;

-- 3. Add NOT NULL constraint (with default)
ALTER TABLE users ALTER COLUMN new_email SET NOT NULL;
ALTER TABLE users ALTER COLUMN new_email SET DEFAULT '';

-- 4. Update application code to use new column

-- 5. Remove old column (after verification)
-- ALTER TABLE users DROP COLUMN email;
```

### 3.3 Database Decomposition

**Shared Database → Service-Specific Databases:**

```
BEFORE (Shared Database):
┌────────────────────────────────────────┐
│           SHARED DATABASE               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │ Users   │  │ Orders  │  │Products │ │
│  │ Table   │  │ Table   │  │ Table   │ │
│  └─────────┘  └─────────┘  └─────────┘ │
└────────────────────────────────────────┘

AFTER (Service-Specific Databases):
┌──────────┐   ┌──────────┐   ┌──────────┐
│  Users   │   │  Orders  │   │ Products │
│ Database │   │ Database │   │ Database │
└──────────┘   └──────────┘   └──────────┘
      │              │              │
      └──────────────┼──────────────┘
                     │
              ┌──────┴──────┐
              │  API CALLS  │
              │  (Async)    │
              └─────────────┘
```

---

## Part 4: Testing Strategies for Legacy Migration

### 4.1 Testing Pyramid for Migrations

```
                    ┌─────────────┐
                    │   E2E Tests │    ← Minimum (Critical paths only)
                    ├─────────────┤
                   │  Integration │   ← Moderate (API contracts)
                   │    Tests     │
                   ├─────────────┐
                  │   Unit Tests  │   ← Maximum (Domain logic)
                  │  (Add new)    │
                  └─────────────┘
```

### 4.2 Characterization Testing

Before refactoring, characterize existing behavior:

```typescript
// test/characterization.test.ts
describe('Legacy Feature X', () => {
  it('should return the same output for known inputs', () => {
    const input = createKnownInput();
    const output = legacyService.process(input);
    
    // Document current behavior
    // This becomes the "contract" for refactoring
    expect(output).toMatchSnapshot();
  });
});
```

### 4.3 Contract Testing

For distributed systems:

```typescript
// Consumer-driven contract testing
describe('Order Service Contract', () => {
  it('fulfills contract for OrderService', () => {
    const contract = {
      provider: 'OrderService',
      consumer: 'PaymentService',
      interactions: [
        {
          request: { method: 'POST', path: '/orders' },
          response: { status: 201, body: { orderId: 'uuid' } }
        }
      ]
    };
    
    verifyContract(OrderService, contract);
  });
});
```

---

## Part 5: Migration Checklist

### Pre-Migration Checklist

- [ ] Complete codebase analysis
- [ ] Identify all dependencies
- [ ] Create dependency graph
- [ ] Map data flows
- [ ] Identify bounded contexts
- [ ] Prioritize extraction order
- [ ] Set up CI/CD pipeline
- [ ] Establish baseline metrics
- [ ] Create test coverage baseline
- [ ] Document current architecture
- [ ] Identify key stakeholders
- [ ] Create communication plan

### During Migration Checklist

- [ ] Implement one feature at a time
- [ ] Maintain feature parity
- [ ] Run tests after each change
- [ ] Monitor error rates
- [ ] Track performance metrics
- [ ] Document decisions (ADRs)
- [ ] Update documentation
- [ ] Communicate progress
- [ ] Review security implications
- [ ] Validate data integrity

### Post-Migration Checklist

- [ ] All tests passing
- [ ] Performance meets targets
- [ ] No regression in functionality
- [ ] Documentation updated
- [ ] Remove legacy code
- [ ] Clean up feature flags
- [ ] Decommission old infrastructure
- [ ] Archive legacy documentation
- [ ] Conduct retrospective
- [ ] Update runbooks
- [ ] Celebrate! 🎉

---

## Part 6: Anti-Patterns in Legacy Modernization

### Anti-Pattern 1: Big Bang Rewrite

**Problem:** Replacing entire system at once

**Symptoms:**
- Long development period without delivery
- High risk of failure
- Team burnout
- Business disruption

**Solution:** Incremental migration using Strangler Fig Pattern

### Anti-Pattern 2: Testing After the Fact

**Problem:** Adding tests only at the end

**Symptoms:**
- Fear of making changes
- Undiscovered regressions
- Slow development

**Solution:** Test-first approach, characterization tests

### Anti-Pattern 3: Ignoring Data

**Problem:** Focusing only on code

**Symptoms:**
- Data inconsistencies
- Broken references
- Performance issues

**Solution:** Include data migration in the plan

### Anti-Pattern 4: No Rollback Plan

**Problem:** Assuming migration will succeed

**Symptoms:**
- Extended outages when issues occur
- Pressure to "fix in place"
- Business impact

**Solution:** Feature flags, blue-green deployment, rollback strategy

### Anti-Pattern 5: Scope Creep

**Problem:** Adding features during migration

**Symptoms:**
- Never-ending migration
- Extended timelines
- Scope confusion

**Solution:** Freeze feature changes, focus on migration

---

## References

1. Martin Fowler. "Refactoring: Improving the Design of Existing Code"
2. Eric Evans. "Domain-Driven Design: Tackling Complexity in the Heart of Software"
3. Simon Brown. "Modular Monoliths"
4. "Migration From Monolith To Microservices Using Strangler Pattern" - Brainhub
5. refactor-to-ca-analysis.md - Real-world 14-step refactoring journey
6. doc/clean-architecture-refactoring-journey.md - Step-by-step guide
7. dev-knowledge/10-legacy/monolith-analysis.md - Monolith analysis
8. dev-knowledge/10-legacy/extraction-patterns.md - Extraction patterns
9. dev-knowledge/03-architecture/clean-architecture.md - Clean Architecture
