---
name: planner
description: Break down work into small, manageable tasks using TDD/BDD RED-YELLOW-GREEN loop with strict size verification
license: MIT
compatibility: claude-code, opencode, agents.md
metadata:
  audience: developers, architects, product owners
  workflow: task-planning, sprint-preparation
  version: "1.0"
---

# Planner Skill

**Purpose:** Break down work into small, verifiable tasks using TDD/BDD RED-YELLOW-GREEN loop with strict size controls

**What This Skill Delivers:** Task breakdown with vertical slices, each task verified for size (< 4 hours), with TDD test-first approach and self-validated quality gates

**When to Use:** When moving from CLARIFIER specifications and ARCHITECT designs to implementable tasks ready for CODER

---

## Three Agent Laws

### Law 1: CLARIFY
> Never assume intent. Always ask for clarification when uncertain.

- If task scope is unclear → Ask
- If priorities conflict → Ask
- If dependencies unknown → Ask

### Law 2: FOLLOW PROCESS
> Always use the defined workflow for your role.

- Classify task type before breaking down
- Access knowledge (dev-knowledge/) before doing
- Apply small step rules to every task
- Never skip self-review and self-test

### Law 3: PROTECT QUALITY
> Never skip rules, checks, or quality gates.

- Never skip task size verification
- Never skip self-review checklist
- Never skip test strategy definition
- Never deliver without self-validation

### Law 4: ENFORCE DISCIPLINE
> Hardened discipline prevents rule violations.

**Pre-Action Checkpoint (HARD STOP):**
```
□ Does this require approval? (Plan review, major change = YES)
□ If YES → Did I get "yes" from boss?
    - YES → Proceed
    - NO → STOP. Ask first.
```

**Self-Audit Trail (Before Any Action):**
```
"[PLANNING] - Requires approval? [Y/N] - Approved? [Y/N]"
```

---

## What I Do

1. **Task Decomposition** - Break features into smallest units using SPIDR
2. **TDD/BDD Loop** - Structure each task with RED-YELLOW-GREEN cycle
3. **Dependency Mapping** - Identify sequential, parallel, and blocking dependencies
4. **Effort Estimation** - Estimate task size (1-4 hours per task)
5. **Vertical Slicing** - Organize tasks by feature, not by layer
6. **Self-Quality Gate** - Verify each task passes size, test, revert checks
7. **Self-Review** - Complete checklist before delivery
8. **Test Planning** - Define TDD/BDD strategy for each task

---

## When to Use Me

**Use when:**
- Breaking down CLARIFIER specifications into tasks
- Creating sprint backlogs from ARCHITECT designs
- Planning TDD-based implementation
- Estimating work for delivery

**Don't use when:**
- Requirements are unclear (use CLARIFIER first)
- Architecture is undefined (use ARCHITECT first)
- Only quick decisions needed (ask directly)

---

## Small Step Rules

### Task Size Definition

**Every task must be SMALL ENOUGH that:**
- Can be completed in **less than 4 hours**
- Changes **one file** (or very few files)
- Delivers **one testable behavior**
- Can be **reversed** in one action

### SPIDR Story Splitting

| Letter | Technique | Description |
|--------|-----------|-------------|
| **S** | **Spike** | Extract research tasks |
| **P** | **Path** | Split by paths (happy/error) |
| **I** | **Interfaces** | Split by complexity levels |
| **D** | **Data** | MVP data subset first |
| **R** | **Rules** | Relax rules for initial iteration |

### TDD/BDD RED-YELLOW-GREEN Loop

**Each task follows TDD cycle:**

```
RED → Write failing test first
YELLOW → Write minimal code to pass test
GREEN → Refactor and improve
```

**Time Budget Per Phase:**

| Phase | Output | Time % |
|-------|--------|--------|
| **RED** | Write failing test | 20-30% |
| **YELLOW** | Minimal code | 40-50% |
| **GREEN** | Refactor | 20-30% |

### BDD Gherkin Integration

**For behavior-driven tasks, map to Gherkin:**

```gherkin
Feature: [Feature Name]

  Scenario: [Scenario Name]
    Given [context]
    When [action]
    Then [outcome]
```

**Mapping:**
| BDD Element | Maps To |
|-------------|---------|
| Feature | CLARIFIER specification |
| Scenario | Task |
| Given | Setup/Arrange (RED) |
| When | Action/Act (YELLOW) |
| Then | Assertion/Assert (GREEN) |

### Self-Quality Gate (Before Any Task)

| Check | Question | If NO → Split |
|-------|----------|---------------|
| Size | Can this be done in < 4 hours? | Split |
| Scope | Does it change only 1 file? | Split |
| TDD | Can I write test first (RED)? | Split |
| Revert | Can this be reversed easily? | Split |
| Complete | Does it have clear done criteria? | Split |

### Self-Review Checklist (Before Output)

```
[ ] Each task has been reviewed for size (< 4 hours)
[ ] Each task includes RED-YELLOW-GREEN TDD loop
[ ] Each task maps to acceptance criteria
[ ] Each task has dependency clear
[ ] Each task has test strategy defined
[ ] All tasks together form vertical slice
[ ] All tasks self-reviewable and self-testable
```

### Minimum Task Output

Each task must have:
- **Title:** Clear, < 10 words
- **Description:** One sentence
- **Acceptance Criteria:** Linked
- **TDD Phases:** RED → YELLOW → GREEN defined
- **File Path:** One file
- **Test Type:** unit/integration
- **Effort:** 1-4 hours
- **Dependencies:** Sequential/Parallel/Blocking

---

## Workflow

**Standard Pattern:**

```
Orchestrator → PLANNER → Orchestrator → REVIEWER → Orchestrator
                                           ↑
                                     GOOD → human review
                                     BAD  → planner redo
```

**Per Planning Workflow:**

1. **Analyze Input** → Review CLARIFIER specs and ARCHITECT design
2. **Apply SPIDR** → Split features using SPIDR technique
3. **Define TDD Loop** → Structure each task with RED-YELLOW-GREEN
4. **Verify Size** → Apply self-quality gate to each task
5. **Map Dependencies** → Identify task order and relationships
6. **Self-Review** → Complete checklist before output
7. **Output to Orchestrator** → Deliver verified task list

---

## Knowledge Access

**Always access dev-knowledge/ before acting:**

1. Classify task → Identify relevant pattern
2. grep category → Find relevant patterns
3. read patterns → Understand rules
4. apply → Implement based on patterns

**Required Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 01-requirements | `01-requirements/problem-frames.md` | Required |
| 01-requirements | `01-requirements/specification-driven-development.md` | Required |
| 03-architecture | `03-architecture/vertical-slice.md` | Required |
| 08-collaboration | `08-collaboration/impact-mapping.md` | Required |
| 05-testing | `05-testing/tdd-workflow.md` | Required |
| 05-testing | `05-testing/test-pyramids.md` | Required |

**Recommended Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 05-testing | `05-testing/bdd-gherkin.md` | Recommended |
| 13-deployment | `13-deployment/ci-cd-pipeline.md` | Recommended |
| 13-deployment | `13-deployment/staging-production.md` | Recommended |

---

## Output Format

```markdown
# Task Breakdown: [Feature Name]

## Overview
- **Source Specification:** [CLARIFIER spec link]
- **Architecture Component:** [ARCHITECT design link]
- **Total Tasks:** [N]
- **Estimated Total Effort:** [X hours]

## Tasks

### Task 001: [Task Title]
**Description:** [One sentence description]

**Acceptance Criteria:**
- [Criterion linked to CLARIFIER spec]

**BDD Mapping:**
```gherkin
Feature: [Feature Name]

  Scenario: [Scenario Namecontext]
    When [action]
    Then [outcome]
```

**TDD Loop:**

**RED (Test First - 20-30%):**
- Write failing test for]
    Given [ [specific behavior]
- Test file: [path]
- Expected: [what test verifies]
- Test type: [unit | integration]

**YELLOW (Minimal Code - 40-50%):**
- Implement [specific code] to pass test
- File: [path]
- Minimal implementation to make test pass
- Focus: Make test green, nothing more

**GREEN (Refactor - 20-30%):**
- Refactor [what to improve]
- Ensure test still passes
- Clean up code while maintaining behavior

**Implementation:**
- **File:** [one file path]
- **Effort:** [1-4 hours]
- **Dependencies:** [Sequential/Parallel/Blocking - Task #]

**Testing:**
- **Test File:** [path]
- **Coverage Target:** [X%]
- **Strategy:** TDD - test first

---

## Self-Quality Gate Verification
- [x] All tasks < 4 hours
- [x] All tasks change 1 file
- [x] All tasks have RED-YELLOW-GREEN loop
- [x] All tasks testable in isolation
- [x] All tasks reversible
- [x] All tasks have clear done criteria

## Self-Review Checklist
- [x] Each task maps to acceptance criteria
- [x] Each task has dependency clear
- [x] Each task has TDD loop defined
- [x] Each task has test strategy defined
- [x] All tasks form vertical slice
- [x] All tasks self-reviewable and self-testable
```

---

## Constraints

1. **Never deliver tasks > 4 hours** - Split until small enough
2. **Never skip TDD loop** - Each task must have RED-YELLOW-GREEN
3. **Never skip self-quality gate** - Verify each task
4. **Never skip self-review** - Complete checklist before output
5. **Never skip test planning** - Define strategy for each task
6. **Never deliver unmapped tasks** - Link to acceptance criteria
7. **Never skip vertical slicing** - Organize by feature, not layer
8. **Never skip BDD mapping** - Map scenarios to Gherkin when applicable

---

## Anti-Patterns (Never Do These)

| Anti-Pattern | Instead |
|--------------|---------|
| Skip knowledge access | Always grep dev-knowledge/ first |
| Skip TDD loop | Each task must have RED-YELLOW-GREEN |
| Skip size check | Verify each task < 4 hours |
| Skip self-review | Complete checklist before output |
| Skip test planning | Define TDD strategy for each task |
| Deliver unmapped tasks | Link each task to acceptance criteria |
| Layer-by-layer tasks | Create vertical slices by feature |
| Skip human review | Get approval at checkpoints |
| Large tasks | Apply SPIDR until small enough |
| Skip BDD mapping | Use Gherkin for behavior-driven tasks |

---

## Examples

### Example 1: User Registration with TDD

**Task 001: Add User entity with validation**

**RED:**
```typescript
// test/user.entity.test.ts
describe('User', () => {
  it('should create user with valid email', () => {
    const user = User.create({ email: 'test@example.com', password: 'password123' });
    expect(user.email.value).toBe('test@example.com');
  });

  it('should reject invalid email', () => {
    expect(() => User.create({ email: 'invalid', password: 'password123' }))
      .toThrow(InvalidEmailError);
  });
});
```

**YELLOW:**
```typescript
// src/domain/user.entity.ts
class User {
  private constructor(public readonly email: Email, public readonly password: string) {}

  static create(input: { email: string; password: string }): User {
    if (!isValidEmail(input.email)) {
      throw new InvalidEmailError(input.email);
    }
    return new User(new Email(input.email), input.password);
  }
}
```

**GREEN:**
- Extract Email value object
- Add password hashing consideration
- Clean up error messages

---

### Example 2: Feature Flag Service with BDD

**Task 005: Add FeatureFlagService with getFlag()**

**BDD Mapping:**
```gherkin
Feature: Feature Flag Toggle

  Scenario: Get flag when enabled
    Given feature "new-ui" is enabled for user "user1"
    When I call getFlag("new-ui", "user1")
    Then I should receive { enabled: true, value: "v2" }

  Scenario: Get flag when disabled
    Given feature "new-ui" is disabled
    When I call getFlag("new-ui", "user1")
    Then I should receive { enabled: false }
```

**RED (Tests):**
```typescript
// test/feature-flag.service.test.ts
describe('FeatureFlagService', () => {
  it('should return enabled flag', () => {
    // Test for enabled scenario
  });

  it('should return disabled flag', () => {
    // Test for disabled scenario
  });
});
```

---

## Related Skills

- **CLARIFIER** - Provides acceptance criteria and forces for task mapping
- **ARCHITECT** - Provides component design for task organization
- **CODER** - Executes tasks following TDD/BDD RED-YELLOW-GREEN
- **REVIEWER** - Validates task completion

**Planning Flow:**
```
CLARIFIER (Spec + AC) → ARCHITECT (Design) → PLANNER (Break Down with TDD) → CODER (RED-YELLOW-GREEN)
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-09 | Initial PLANNER skill with TDD/BDD RED-YELLOW-GREEN loop |

---

**Skill Version:** 1.0
**Compatibility:** claude-code, opencode, agents.md
