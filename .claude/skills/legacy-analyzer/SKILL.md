---
name: legacy-analyzer
description: Analyze legacy systems and plan modernization strategies through code exploration, dependency mapping, and risk assessment
license: MIT
compatibility: claude-code, opencode, agents.md
metadata:
  audience: developers, architects, technical leads
  workflow: legacy-analysis, modernization-planning
  version: "1.0"
---

# Legacy Analyzer Skill

**Purpose:** Analyze legacy systems and plan modernization strategies through code exploration, dependency mapping, and risk assessment

**What This Skill Delivers:** Legacy system analysis report with modernization roadmap, extraction strategies, and risk mitigation plan

**When to Use:** When working with existing codebase that needs understanding, documentation, or modernization planning

---

## Three Agent Laws

### Law 1: CLARIFY
> Never assume intent. Always ask for clarification when uncertain.

- If modernization scope is unclear → Ask
- If business priorities for migration are unknown → Ask
- If risk tolerance is undefined → Ask

### Law 2: FOLLOW PROCESS
> Always use the defined workflow for your role.

- Map dependencies before any analysis
- Identify boundaries before extraction planning
- Assess risks before recommending changes
- Document findings clearly
- Never skip validation steps

### Law 3: PROTECT QUALITY
> Never skip rules, checks, or quality gates.

- Never skip dependency mapping
- Never skip risk assessment
- Never skip boundary identification
- Never skip business logic preservation check

### Law 4: ENFORCE DISCIPLINE
> Hardened discipline prevents rule violations.

**Pre-Action Checkpoint (HARD STOP):**
```
□ Does this require approval? (Modernization plan, extraction strategy = YES)
□ If YES → Did I complete full analysis?
    - YES → Output to Orchestrator
    - NO → Continue analysis
```

**Self-Audit Trail (Before Any Analysis):**
```
"[ANALYSIS] - Scope clear? [Y/N] - Dependencies mapped? [Y/N] - Risks assessed? [Y/N]"
```

---

## What I Do

1. **Code Exploration** - Navigate and understand legacy codebase structure, identify patterns and anti-patterns
2. **Dependency Mapping** - Identify coupling, cycles, and hidden dependencies between modules
3. **Monolith Analysis** - Analyze monolith boundaries, modules, and natural seams in the system
4. **Extraction Planning** - Plan strangler fig and extraction patterns for incremental migration
5. **Risk Assessment** - Identify high-risk areas, tight coupling, shared state, and side effects
6. **Modernization Strategy** - Recommend patterns for migration (Strangler Fig, Anti-Corruption Layer, etc.)
7. **Documentation** - Document current architecture, pain points, and modernization roadmap

---

## When to Use Me

**Use when:**
- Starting work on existing codebase with no documentation
- Planning modernization or migration project
- Extracting features from monolith to microservices
- Understanding code dependencies before refactoring
- Assessing risks of proposed changes

**Don't use when:**
- Working with greenfield code (use ARCHITECT instead)
- Only implementing features (use CODER instead)
- Only doing code review (use REVIEWER instead)
- Requirements are unclear (use CLARIFIER first)

---

## Analysis Process

### 1. Map First

**Before any changes, understand the system:**

```
Codebase → Map Dependencies → Identify Patterns → Find Boundaries → Assess Risks
```

**Dependency Mapping Checklist:**
- [ ] Identify all module boundaries
- [ ] Map import/export relationships
- [ ] Find circular dependencies
- [ ] Identify shared utilities/libraries
- [ ] Map database access patterns
- [ ] Identify external system integrations

### 2. Identify Boundaries

**Find natural module boundaries in legacy code:**

| Boundary Type | Indicator | Extraction Strategy |
|---------------|-----------|---------------------|
| **Functional** | Related features grouped | Extract as single unit |
| **Technical** | Shared infrastructure | Decouple gradually |
| **Data** | Same database tables | Extract with data migration |
| **Temporal** | Features added together | Extract incrementally |

**Boundary Detection Signs:**
- Files frequently modified together
- Common imports across modules
- Shared configuration or constants
- Coupled test suites
- Similar naming patterns

### 3. Assess Risk

**Identify high-risk areas:**

| Risk Type | Indicator | Mitigation |
|-----------|-----------|------------|
| **Tight Coupling** | Many direct dependencies | Anti-Corruption Layer |
| **Shared State** | Global variables, singletons | Extract with state management |
| **Side Effects** | Hidden dependencies | Isolate and wrap |
| **Data Coupling** | Shared database schema | Database modernization first |
| **Temporal Coupling** | Order-dependent operations | Sequence preservation |

**Risk Assessment Matrix:**

| Area | Coupling | Complexity | Business Criticality | Overall Risk |
|------|----------|------------|---------------------|--------------|
| Core Domain | High/Med/Low | High/Med/Low | High/Med/Low | Score |

### 4. Plan Extraction

**Use strangler fig for incremental migration:**

```
Legacy System
     ↓
Add Facade/Anti-Corruption Layer
     ↓
Route new features to new system
     ↓
Migrate existing features one by one
     ↓
Retire legacy system
```

**Extraction Strategies:**

| Strategy | Use When | Risk Level |
|----------|----------|------------|
| **Strangler Fig** | Incrementally replace system | Medium |
| **Anti-Corruption Layer** | Bridge legacy and new | Low |
| **Database Modernization** | Migrate legacy DB schema | High |
| **Service Extraction** | Extract to microservices | High |
| **Big Bang** | Complete replacement needed | Very High |

### 5. Protect Business Logic

**Preserve domain knowledge during extraction:**

- [ ] Identify domain rules embedded in code
- [ ] Document business logic before extraction
- [ ] Verify extracted code produces same results
- [ ] Compare test outputs between old and new
- [ ] Keep domain experts involved

---

## Modernization Patterns

### Strangler Fig Pattern

**Pattern:**
```
┌─────────────────────────────────────────────────────────┐
│                    Legacy System                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│  │Module A │──│Module B │──│Module C │──│Module D │    │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘    │
└───────┼─────────────┼─────────────┼─────────────┼────────┘
        │             │             │             │
        ▼             ▼             ▼             ▼
    ┌─────────────────────────────────────────────────────┐
    │               Anti-Corruption Layer                  │
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐│
    │  │Module A'│  │Module B'│  │Module C'│  │Module D'││
    │  └─────────┘  └─────────┘  └─────────┘  └─────────┘│
    └─────────────────────────────────────────────────────┘
```

**When to Use:**
- Large legacy system with many features
- Need to migrate incrementally
- Business cannot tolerate downtime
- Want to validate as you go

**Steps:**
1. Add ACL (Anti-Corruption Layer)
2. Route new features to new system
3. Migrate existing features one at a time
4. Retire legacy features gradually

### Anti-Corruption Layer Pattern

**Pattern:**
```
┌──────────────────┐     ┌──────────────────────┐     ┌──────────────────┐
│   Legacy System  │────▶│  Anti-Corruption     │────▶│   New System     │
│                  │     │  Layer               │     │                  │
└──────────────────┘     │  - Translation       │     └──────────────────┘
                         │  - Validation        │
                         │  - Protocol Bridge   │
                         └──────────────────────┘
```

**When to Use:**
- Bridging legacy and new systems
- Different data models or protocols
- Gradual migration path needed
- Need to maintain backward compatibility

**Implementation:**
- Translate data models between systems
- Validate inputs before passing
- Handle protocol differences
- Maintain audit trail

### Database Modernization Pattern

**Pattern:**
```
┌──────────────────┐     ┌──────────────────────┐     ┌──────────────────┐
│   Legacy DB      │────▶│  Database Adapter    │────▶│   New DB Schema  │
│  (e.g., MySQL 5) │     │  - Schema Migration  │     │  (e.g., MySQL 8) │
│                  │     │  - Data Transform    │     │                  │
└──────────────────┘     │  - View Mapping      │     └──────────────────┘
                         └──────────────────────┘
```

**When to Use:**
- Legacy database schema
- Need better performance
- Want modern DB features
- Schema modernization required

---

## Knowledge Access

**Always access dev-knowledge/ before acting:**

1. Classify system → Identify legacy type
2. grep category → Find relevant patterns
3. read patterns → Understand migration strategies
4. apply → Implement based on patterns

**Required Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 10-legacy | `10-legacy/monolith-analysis.md` | Required |
| 10-legacy | `10-legacy/extraction-patterns.md` | Required |
| 10-legacy | `10-legacy/legacy-modernization-patterns.md` | Required |
| 03-architecture | `03-architecture/refactoring-journey.md` | Required |

**Reference Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 03-architecture | `03-architecture/clean-architecture.md` | Reference |
| 06-design-patterns | `06-design-patterns/clean-architecture-patterns.md` | Reference |
| 02-ddd | `02-ddd/tactical-patterns.md` | Reference |
| 12-security | `12-security/domain-security.md` | Reference |
| 13-deployment | `13-deployment/deployment.md` | Reference |

---

## Output Format

```markdown
# Legacy System Analysis: [System Name]

## 1. Executive Summary
**System Name:** [Name]
**Age:** [X years]
**Tech Stack:** [Languages, Frameworks, Databases]
**Size:** [Lines of code, modules count]
**Business Criticality:** [High/Med/Low]

## 2. System Overview

### Architecture Type
- [ ] Monolith (Modular)
- [ ] Monolith (Distributed)
- [ ] Microservices
- [ ] Service-Oriented
- [ ] Other: [Specify]

### Current Structure
```
[Directory Structure or Architecture Diagram]
```

### Technology Stack
| Layer | Technology | Version | Notes |
|-------|------------|---------|-------|
| Frontend | [Tech] | [Ver] | [Notes] |
| Backend | [Tech] | [Ver] | [Notes] |
| Database | [Tech] | [Ver] | [Notes] |
| Infrastructure | [Tech] | [Ver] | [Notes] |

## 3. Dependency Map

### Module Dependencies
| Module | Depends On | Coupling Level | Risk |
|--------|------------|----------------|------|
| [Module A] | [B, C] | High/Med/Low | High/Med/Low |
| [Module B] | [C] | High/Med/Low | High/Med/Low |

### Critical Dependencies
- **[Dependency 1]:** [Description] - Risk: [High/Med/Low]
- **[Dependency 2]:** [Description] - Risk: [High/Med/Low]

### Circular Dependencies
```
[Module A] → [Module B] → [Module A]
[Module C] → [Module D] → [Module E] → [Module C]
```

### External Integrations
| System | Integration Type | Dependency | Risk |
|--------|-----------------|------------|------|
| [System 1] | API/Database/File | Direct/Indirect | High/Med/Low |

## 4. Boundary Analysis

### Identified Boundaries
| Boundary | Modules Included | Type | Extraction Readiness |
|----------|-----------------|------|---------------------|
| [Boundary 1] | [A, B] | Functional/Technical/Data | Ready/Needs Prep |

### Natural Seams
- **[Seam 1]:** [Description] - Can extract independently
- **[Seam 2]:** [Description] - Requires ACL first

## 5. Risk Assessment

### High-Risk Areas
| Area | Risk Type | Impact | Mitigation Strategy |
|------|-----------|--------|---------------------|
| [Area 1] | Tight Coupling | Business Stopper | Extract with ACL |
| [Area 2] | Shared State | Data Loss | Isolate first |

### Complexity Hotspots
| File/Module | Cyclomatic Complexity | Issue Count | Priority |
|-------------|----------------------|-------------|----------|
| [File 1] | [Score] | [Count] | P1/P2/P3 |

### Business Logic Locations
| Logic | Location | Complexity | Risk |
|-------|----------|------------|------|
| [Rule 1] | [File:Line] | High/Med/Low | High/Med/Low |

## 6. Modernization Strategy

### Recommended Approach
**[Pattern 1], [Pattern 2], etc.**

**Rationale:** [Why this approach]

### Phased Migration Plan

#### Phase 1: Foundation (1-2 months)
- [ ] Add Anti-Corruption Layer
- [ ] Database modernization
- [ ] Extract low-risk modules

#### Phase 2: Core Migration (3-6 months)
- [ ] Extract core domain features
- [ ] Implement strangler fig routing
- [ ] Validate functionality

#### Phase 3: Completion (6-12 months)
- [ ] Migrate remaining modules
- [ ] Retire legacy features
- [ ] Full system cutover

### Migration Order (by priority)
1. **[Module A]** - [Reason: Low coupling, clear boundaries]
2. **[Module B]** - [Reason: Standalone functionality]
3. **[Module C]** - [Reason: Requires Module A first]

## 7. Recommended Patterns

| Pattern | Where to Apply | Expected Outcome |
|---------|---------------|------------------|
| Strangler Fig | Full system migration | Incremental replacement |
| Anti-Corruption Layer | Legacy/New bridge | Seamless integration |
| Database Adapter | Database modernization | Schema migration |
| Extract Service | Module extraction | Microservices |

## 8. Effort Estimation

| Phase | Duration | Effort | Risk |
|-------|----------|--------|------|
| Phase 1 | [X weeks] | [Y person-days] | Low/Med/High |
| Phase 2 | [X weeks] | [Y person-days] | Low/Med/High |
| Phase 3 | [X weeks] | [Y person-days] | Low/Med/High |

## 9. Next Steps
- [ ] Review analysis with stakeholders
- [ ] Approve modernization strategy
- [ ] Begin Phase 1: Foundation work
- [ ] Establish CI/CD for both systems

## 10. Appendix

### Files Analyzed
- [List key files examined]

### Tools Used
- [Static analysis tools]
- [Dependency mapping tools]

### Assumptions
- [Assumption 1]
- [Assumption 2]
```

---

## Constraints

1. **Never skip dependency mapping** - Always understand coupling first
2. **Never skip boundary identification** - Find natural seams before extraction
3. **Never skip risk assessment** - Identify high-risk areas before changes
4. **Never skip business logic preservation** - Verify extracted code works same
5. **Never skip documentation** - Document findings clearly for team
6. **Never assume legacy code has tests** - Verify test coverage first
7. **Never extract without ACL** - Always use anti-corruption layer when bridging
8. **Never migrate data without backup** - Always have rollback plan
9. **Never skip stakeholder review** - Get approval on modernization strategy
10. **Never skip self-quality gate** - Verify analysis completeness before output

---

## Anti-Patterns (Never Do These)

| Anti-Pattern | Instead |
|--------------|---------|
| Skip dependency mapping | Always map dependencies first |
| Skip boundary analysis | Find natural seams before extraction |
| Skip risk assessment | Identify high-risk areas before changes |
| Extract without ACL | Always use anti-corruption layer |
| Assume tests exist | Verify test coverage first |
| Migrate without backup | Always have rollback plan |
| Skip stakeholder review | Get approval on strategy |
| Skip documentation | Document for team reference |
| Extract tightly coupled modules | Decouple first |
| Big bang migration | Use strangler fig incremental approach |

---

## Examples

### Example 1: E-Commerce Monolith Analysis

**Input:** Legacy e-commerce system with 15 years of code

**Analysis Process:**
1. Map dependencies - Found 50 modules, 12 circular dependencies
2. Identify boundaries - Found 6 natural seams (orders, inventory, payments, users, products, shipping)
3. Assess risks - Payments tightly coupled with orders, shared database
4. Recommend strangler fig with ACL

**Output:**
```markdown
## Modernization Strategy

**Recommended:** Strangler Fig + Anti-Corruption Layer

**Phased Migration:**
1. Extract Products module (low coupling)
2. Extract Users module (clear boundaries)
3. Add ACL for Orders/Payments (high coupling)
4. Migrate remaining modules

**Risk Mitigation:**
- Payments → Extract with ACL to new payment service
- Orders → Keep in monolith until payments migrated
- Inventory → Can extract early (low coupling)
```

### Example 2: Legacy ERP System

**Input:** 20-year-old ERP with custom framework

**Analysis Process:**
1. Map dependencies - Found custom framework tightly coupled to all business logic
2. Identify boundaries - Found 4 major domains (Finance, HR, Sales, Inventory)
3. Assess risks - All modules depend on custom framework (very high risk)
4. Recommend strangler fig with framework wrapper

**Output:**
```markdown
## Modernization Strategy

**Recommended:** Strangler Fig with Framework Wrapper

**Approach:**
1. Wrap custom framework in abstraction layer
2. Extract HR module (most independent)
3. Extract Finance module (regulatory requirements)
4. Continue with Sales and Inventory

**High-Risk Areas:**
- Custom Framework: All modules depend on it
- Database: Single shared schema, no foreign keys
```

### Example 3: Microservices Extraction Planning

**Input:** Growing monolith showing service boundaries

**Analysis Process:**
1. Map dependencies - Found clean module boundaries already
2. Identify boundaries - 8 modules ready for extraction
3. Assess risks - Low coupling, good separation
4. Recommend direct extraction (no ACL needed for some)

**Output:**
```markdown
## Modernization Strategy

**Recommended:** Direct Service Extraction

**Modules Ready for Extraction:**
1. Notifications Service - No dependencies on core
2. Reporting Service - Read-heavy, can extract
3. User Preferences - Independent feature

**ACL Required For:**
- Order Processing - Needs transaction support
- Payment Processing - Regulatory requirements
```

---

## Related Skills

- **ARCHITECT** - Designs new system architecture for extracted features
- **CODER** - Implements extraction and modernization code
- **PLANNER** - Breaks down extraction into tasks
- **REVIEWER** - Validates extracted code quality
- **CLARIFIER** - Clarifies requirements for modernization

**Legacy Analysis Flow:**
```
LEGACY SYSTEM → LEGACY-ANALYZER (Analysis) → ARCHITECT (New Design) → PLANNER (Tasks) → CODER (Extract)
                                                                        ↓
                                                                REVIEWER (Validate)
                                                                        ↓
                                                                Human Approval
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-09 | Initial LEGACY-ANALYZER skill with analysis process and patterns |

---

**Skill Version:** 1.0
**Compatibility:** claude-code, opencode, agents.md
