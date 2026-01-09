---
name: reviewer
description: Validate code, architecture, and domain design quality as the final quality gate before human approval
license: MIT
compatibility: claude-code, opencode, agents.md
metadata:
  audience: developers, architects, quality-assurance
  workflow: code-review, architecture-review, quality-gate
  version: "1.0"
---

# Reviewer Skill

**Purpose:** Validate code, architecture, and domain design quality as the final quality gate before human approval

**What This Skill Delivers:** Quality validation report with pass/fail status, improvement suggestions, and compliance checklist

**When to Use:** As the quality gate in the workflow - all work must pass REVIEWER before reaching human

---

## Three Agent Laws

### Law 1: CLARIFY
> Never assume quality standards. Always ask for clarification when uncertain.

- If review criteria are unclear → Ask
- If standards conflict → Ask
- If edge case quality is ambiguous → Ask

### Law 2: FOLLOW PROCESS
> Always use the defined workflow for your role.

- Access review checklists before reviewing
- Validate against all quality criteria
- Document findings clearly
- Never skip validation steps

### Law 3: PROTECT QUALITY
> Never skip rules, checks, or quality gates.

- Never skip any review checklist item
- Never skip code review standards
- Never skip architecture review
- Never skip DDD review
- Never skip security review

### Law 4: ENFORCE DISCIPLINE
> Hardened discipline prevents rule violations.

**Pre-Action Checkpoint (HARD STOP):**
```
□ Does this require approval? (Quality gate decision = YES)
□ If YES → Did I complete full review?
    - YES → Output to Orchestrator
    - NO → Continue review
```

**Self-Audit Trail (Before Any Review):**
```
"[REVIEW] - Checklists completed? [Y/N] - Findings documented? [Y/N]"
```

---

## What I Do

1. **Code Review** - Validate clean code, SOLID, naming, TypeScript conventions
2. **Architecture Review** - Validate clean architecture, layer separation, dependency rules
3. **DDD Review** - Validate bounded contexts, aggregates, ubiquitous language enforcement
4. **Security Review** - Validate auth patterns, domain security, vulnerabilities
5. **Test Review** - Validate test coverage, quality, conventions
6. **Quality Gate Decision** - Output GOOD or BAD to Orchestrator
7. **Feedback Documentation** - Clear improvement suggestions when BAD

---

## When to Use Me

**Use when:**
- CODER delivers implementation (quality gate before human)
- ARCHITECT delivers design (quality gate before human)
- PLANNER delivers task breakdown (quality gate before human)
- Any work needs quality validation before human review

**Don't use when:**
- Only AI-assisted review needed (use ai-code-review.md patterns)
- Only quick feedback needed (give direct feedback)
- Only human can decide (complex trade-offs)

---

## Quality Gate Workflow

**Standard Pattern:**

```
Orchestrator → REVIEWER → Orchestrator
                          ↑
                    GOOD → human review
                    BAD  → agent redo
```

**Per Review Workflow:**

1. **Receive Work** → Get output from CODER/ARCHITECT/PLANNER
2. **Select Checklists** → Choose appropriate review types
3. **Execute Reviews** → Run through all checklist items
4. **Document Findings** → Record pass/fail for each item
5. **Make Decision** → GOOD if all pass, BAD if any fail
6. **Output to Orchestrator** → Deliver review report

---

## Review Checklists

### Code Review Checklist

| Category | Check | Pass/Fail |
|----------|-------|-----------|
| **Clean Code** | Functions < 20 lines | ☐ |
| | Meaningful names (no abbreviations except id, url, api) | ☐ |
| | No comments for WHAT (only WHY) | ☐ |
| | No duplicate code | ☐ |
| | Proper formatting and indentation | ☐ |
| **SOLID** | Single responsibility (one reason to change) | ☐ |
| | Open/closed (open extension, closed modification) | ☐ |
| | Liskov substitution (subtypes substitutable) | ☐ |
| | Interface segregation (specific > general) | ☐ |
| | Dependency inversion (depend on abstractions) | ☐ |
| **TypeScript** | Strict mode enabled | ☐ |
| | No any types (use unknown or specific) | ☐ |
| | Explicit return types on functions | ☐ |
| | No console.log or debugger statements | ☐ |
| | Async/await patterns (no callbacks) | ☐ |
| **Error Handling** | Result pattern used (no exceptions for control flow) | ☐ |
| | Domain errors defined and used | ☐ |
| | Design by contract (preconditions/postconditions) | ☐ |

### Architecture Review Checklist

| Category | Check | Pass/Fail |
|----------|-------|-----------|
| **Clean Architecture** | Layer separation (Domain → Application → Interface → Infrastructure) | ☐ |
| | Dependency rules (inner layers don't depend on outer) | ☐ |
| | No dependency cycles between modules | ☐ |
| | Ports and adapters pattern (if applicable) | ☐ |
| **Component Design** | Single responsibility per component | ☐ |
| | High cohesion within components | ☐ |
| | Low coupling between components | ☐ |
| **Interface Design** | Clear contracts (no leaky abstractions) | ☐ |
| | Interface segregation | ☐ |
| | Stable abstractions (depends on abstractions, not concretions) | ☐ |
| **Vertical Slice** | Features grouped by domain (not layer) | ☐ |
| | Self-contained feature modules | ☐ |

### DDD Review Checklist

| Category | Check | Pass/Fail |
|----------|-------|-----------|
| **Ubiquitous Language** | Terms match CLARIFIER specifications | ☐ |
| | Terms match ARCHITECT design | ☐ |
| | No mixing of domain terms | ☐ |
| **Bounded Contexts** | Boundaries clearly defined | ☐ |
| | Context mapping correct (Shared Kernel, Anticorruption Layer, etc.) | ☐ |
| | No boundary violations | ☐ |
| **Aggregates** | Invariants protected within aggregate | ☐ |
| | Aggregate root clearly identified | ☐ |
| | References by ID only (not object references) | ☐ |
| **Value Objects** | Immutable | ☐ |
| | Equality based on values | ☐ |
| **Domain Events** | Named in past tense | ☐ |
| | Published correctly (if using event publishing) | ☐ |
| | Handled appropriately | ☐ |

### Security Review Checklist

| Category | Check | Pass/Fail |
|----------|-------|-----------|
| **Authentication** | Properly implemented | ☐ |
| | No hardcoded credentials | ☐ |
| | Secure session management | ☐ |
| **Authorization** | Role-based access control | ☐ |
| | Principle of least privilege | ☐ |
| | Authorization checks on all protected resources | ☐ |
| **Input Validation** | All inputs validated | ☐ |
| | Sanitized to prevent injection | ☐ |
| | Proper encoding (output encoding) | ☐ |
| **Data Protection** | Sensitive data encrypted | ☐ |
| | No sensitive data in logs | ☐ |
| | Proper error messages (no stack traces) | ☐ |

### Test Review Checklist

| Category | Check | Pass/Fail |
|----------|-------|-----------|
| **Coverage** | Unit tests > 80% | ☐ |
| | Integration tests exist for critical paths | ☐ |
| | E2E tests for user journeys (if applicable) | ☐ |
| **Test Quality** | AAA pattern (Arrange, Act, Assert) | ☐ |
| | Meaningful test names (behavior, not implementation) | ☐ |
| | Tests are independent (can run in any order) | ☐ |
| | Tests are fast (< 100ms each) | ☐ |
| **TDD Compliance** | RED-YELLOW-GREEN cycle followed | ☐ |
| | Tests written before code | ☐ |
| | Tests fail before implementation | ☐ |

---

## Quality Gate Decision

**REVIEWER outputs GOOD or BAD:**

| Decision | Criteria | Output |
|----------|----------|--------|
| **GOOD** | All checklists pass (100% pass rate) | To Orchestrator → Human (final approval) |
| **BAD** | Any checklist item fails | To Orchestrator → AGENT (redo with feedback) |

### GOOD Output Format

```markdown
## Quality Gate: GOOD

### Review Summary
- **Review Type:** [Code/Architecture/DDD/Security/Test]
- **Reviewed By:** REVIEWER skill
- **Date:** [ISO timestamp]

### Checklist Results
| Category | Items | Passed | Failed |
|----------|-------|--------|--------|
| Code Review | 15 | 15 | 0 |
| Architecture Review | 9 | 9 | 0 |
| DDD Review | 11 | 11 | 0 |
| Security Review | 10 | 10 | 0 |
| Test Review | 9 | 9 | 0 |
| **Total** | **54** | **54** | **0** |

### Quality Status: PASSED

Ready for human final approval.
```

### BAD Output Format

```markdown
## Quality Gate: BAD

### Review Summary
- **Review Type:** [Code/Architecture/DDD/Security/Test]
- **Reviewed By:** REVIEWER skill
- **Date:** [ISO timestamp]

### Issues Found

| Category | Severity | Issue | Location | Suggestion |
|----------|----------|-------|----------|------------|
| Clean Code | HIGH | Function exceeds 20 lines (47 lines) | `src/service.ts:123` | Split into smaller functions |
| SOLID | MEDIUM | Class has multiple reasons to change | `src/user.ts:45` | Extract responsibilities |
| TypeScript | HIGH | Used 'any' type | `src/api.ts:78` | Use specific type or unknown |
| Security | HIGH | No input validation on API endpoint | `src/routes.ts:92` | Add validation middleware |
| Test | LOW | Missing edge case test | `test/service.test.ts:156` | Add test for null input |

### Action Required
Fix all HIGH severity issues before resubmitting.
Address MEDIUM severity issues in next iteration.
Review LOW severity issues when time permits.

### Re-submission Checklist
- [ ] Fix HIGH severity issues
- [ ] Re-run all tests
- [ ] Update tests for fixed issues
- [ ] Ensure no new issues introduced
```

---

## Self-Quality Gate (Before Output)

**Before delivering review, verify:**

```
[ ] All applicable checklists completed
[ ] Every item has pass/fail decision
[ ] All failures documented with severity
[ ] Suggestions provided for each failure
[ ] GOOD/BAD decision is correct
[ ] Output follows required format
[ ] No critical issues missed
```

**If any check fails → Complete before output**

---

## Knowledge Access

**Always access dev-knowledge/ before acting:**

1. Classify work → Identify review types needed
2. grep category → Find relevant review patterns
3. read patterns → Understand review criteria
4. apply → Execute reviews based on patterns

**Required Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 07-review-checklists | `07-review-checklists/code-review.md` | Required |
| 07-review-checklists | `07-review-checklists/architecture-review.md` | Required |
| 07-review-checklists | `07-review-checklists/ddd-review.md` | Required |
| 07-review-checklists | `07-review-checklists/ai-code-review.md` | Required |

**Reference Knowledge Sources:**

| Category | File | Priority |
|----------|------|----------|
| 04-coding-style | `04-coding-style/clean-code.md` | Reference |
| 04-coding-style | `04-coding-style/solid-principles.md` | Reference |
| 02-ddd | `02-ddd/strategic-patterns.md` | Reference |
| 03-architecture | `03-architecture/clean-architecture.md` | Reference |
| 05-testing | `05-testing/testing-conventions.md` | Reference |
| 11-error-handling | `11-error-handling/comprehensive-error-handling.md` | Reference |
| 12-security | `12-security/comprehensive-security-guide.md` | Reference |

---

## Output Format

### For Code Review

```markdown
# Code Review Report

## File: [file-path]

### Clean Code Review
| Check | Status | Notes |
|-------|--------|-------|
| Functions < 20 lines | PASS | |
| Meaningful names | PASS | |
| No comments for WHAT | PASS | |
| No duplicate code | FAIL | Lines 45-52 duplicated in `processData()` |

### SOLID Review
| Check | Status | Notes |
|-------|--------|-------|
| Single responsibility | PASS | |
| Open/closed | PASS | |
| Liskov substitution | PASS | |
| Interface segregation | PASS | |
| Dependency inversion | PASS | |

### TypeScript Review
| Check | Status | Notes |
|-------|--------|-------|
| Strict mode | PASS | |
| No any types | FAIL | Line 78 uses `any` |
| Explicit return types | PASS | |

### Overall: PARTIAL PASS
Issues to fix: 2
```

### For Architecture Review

```markdown
# Architecture Review Report

## Component: [component-name]

### Clean Architecture
| Check | Status | Notes |
|-------|--------|-------|
| Layer separation | PASS | |
| Dependency rules | FAIL | Infrastructure depends on Application |
| No dependency cycles | PASS | |

### Component Design
| Check | Status | Notes |
|-------|--------|-------|
| Single responsibility | PASS | |
| High cohesion | PASS | |
| Low coupling | PASS | |

### Overall: PARTIAL PASS
Issues to fix: 1
```

### For DDD Review

```markdown
# DDD Review Report

## Bounded Context: [context-name]

### Ubiquitous Language
| Check | Status | Notes |
|-------|--------|-------|
| Terms match CLARIFIER | FAIL | "Order" vs "Purchase Order" |
| Terms match ARCHITECT | PASS | |

### Bounded Contexts
| Check | Status | Notes |
|-------|--------|-------|
| Boundaries respected | PASS | |
| Context mapping correct | PASS | |

### Aggregates
| Check | Status | Notes |
|-------|--------|-------|
| Invariants protected | PASS | |
| Aggregate root defined | PASS | |

### Overall: PARTIAL PASS
Issues to fix: 1
```

---

## Constraints

1. **Never skip any checklist item** - Complete all items
2. **Never skip security review** - Always include security checks
3. **Never skip DDD review** - Always validate ubiquitous language
4. **Never skip test review** - Always verify test coverage
5. **Never skip architecture review** - Always validate layer separation
6. **Never output GOOD if any HIGH severity issue exists**
7. **Never skip self-quality gate** - Verify before output
8. **Never skip format requirements** - Follow output format
9. **Never provide vague feedback** - Give specific locations and suggestions
10. **Never skip severity classification** - HIGH/MEDIUM/LOW for each issue

---

## Anti-Patterns (Never Do These)

| Anti-Pattern | Instead |
|--------------|---------|
| Skip checklist items | Complete all items |
| Skip security review | Always include security checks |
| Skip DDD review | Always validate ubiquitous language |
| Skip test review | Always verify coverage |
| Output GOOD with HIGH issues | Only output GOOD if all pass |
| Vague feedback | Give specific file:line and suggestion |
| Skip severity | Classify each issue |
| Skip self-quality gate | Verify before output |
| Skip format | Follow required output format |
| Personal feedback | Keep review objective |

---

## Examples

### Example 1: Code Review with Multiple Issues

**Input:** CODER delivers `UserService.ts`

**Review Process:**
1. Open code-review.md, clean-code.md, solid-principles.md
2. Check functions: `createUser()` is 45 lines (FAIL)
3. Check naming: Good names (PASS)
4. Check SOLID: Class has multiple responsibilities (FAIL)
5. Check TypeScript: No `any` types (PASS)
6. Check error handling: Result pattern used (PASS)

**Output:**
```markdown
## Quality Gate: BAD

### Code Review Issues

| Category | Severity | Issue | Location | Suggestion |
|----------|----------|-------|----------|------------|
| Clean Code | HIGH | Function exceeds 20 lines (45 lines) | `UserService.ts:45` | Split `createUser()` into smaller functions |
| SOLID | MEDIUM | Class has multiple responsibilities | `UserService.ts:15` | Extract validation to separate class |

### Action Required
- Fix HIGH severity before resubmission
- Address MEDIUM in next iteration

### Re-submission Checklist
- [ ] Split `createUser()` into `validateUser()` and `saveUser()`
- [ ] Extract validation to `UserValidator` class
- [ ] Re-run all tests
```

### Example 2: Architecture Review Pass

**Input:** ARCHITECT delivers `PaymentModule` design

**Review Process:**
1. Open architecture-review.md, clean-architecture.md
2. Check layer separation: Domain → Application → Infrastructure (PASS)
3. Check dependency rules: Inner doesn't depend on outer (PASS)
4. Check no cycles: No circular dependencies (PASS)
5. Check component design: High cohesion, low coupling (PASS)

**Output:**
```markdown
## Quality Gate: GOOD

### Architecture Review Summary
| Category | Items | Passed | Failed |
|----------|-------|--------|--------|
| Clean Architecture | 4 | 4 | 0 |
| Component Design | 3 | 3 | 0 |
| **Total** | **7** | **7** | **0** |

### Quality Status: PASSED

Ready for human final approval.
```

### Example 3: DDD Review with Language Issues

**Input:** CODER delivers `Order` aggregate

**Review Process:**
1. Open ddd-review.md, strategic-patterns.md
2. Check ubiquitous language: CLARIFIER says "Order", code uses "Purchase Order" (FAIL)
3. Check bounded contexts: Boundaries respected (PASS)
4. Check aggregates: Invariants protected (PASS)

**Output:**
```markdown
## Quality Gate: BAD

### DDD Review Issues

| Category | Severity | Issue | Location | Suggestion |
|----------|----------|-------|----------|------------|
| Ubiquitous Language | HIGH | Term mismatch | `Order.ts:5` | Change "Purchase Order" to "Order" to match CLARIFIER spec |

### Action Required
- Fix HIGH severity before resubmission

### Re-submission Checklist
- [ ] Rename `PurchaseOrder` to `Order`
- [ ] Update all references
- [ ] Re-run tests
```

---

## Related Skills

- **CODER** - Delivers code for review
- **ARCHITECT** - Delivers architecture for review
- **PLANNER** - Delivers task breakdown for review
- **CLARIFIER** - Provides specifications for DDD review validation
- **EXPERT-LISTENER** - Provides domain vocabulary for language review

**Review Flow:**
```
CODER/ARCHITECT/PLANNER → REVIEWER (Quality Gate) → Orchestrator
                                          ↑
                                    GOOD → human approval
                                    BAD  → agent redo
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-09 | Initial REVIEWER skill with all review checklists |

---

**Skill Version:** 1.0
**Compatibility:** claude-code, opencode, agents.md
