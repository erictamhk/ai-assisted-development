# AI Agent Limitations: Why Instructions Are Ignored

## The Core Problem

This document addresses a fundamental challenge in AI-assisted development: **why AI agents consistently fail to follow complex instructions, rules, and guidelines**, even when those instructions are explicitly provided in AGENTS.md or similar files.

The user's observation is accurate and widely shared among practitioners:

> Real-world projects contain massive amounts of rules, regulations, guidelines, and principles (often exceeding 1M+ tokens) covering coding style, conventions, constitutions, and guidelines. A simple AGENTS.md file cannot capture all this complexity, yet AI agents are expected to follow these rules perfectly.

This document explores why this expectation is fundamentally misaligned with how AI agents work, and provides practical strategies for working within these constraints.

---

## The Fundamental Architecture Problem

### Context Window Limitations

Modern AI models have hard limits on how many tokens they can process in a single inference:

| Model | Context Window |
|-------|---------------|
| GPT-4o | 128K tokens |
| Claude 3.5 Sonnet | 200K tokens |
| Claude 4 (beta) | 1M tokens |
| Gemini 1.5 Pro | 1M-2M tokens |

**The critical insight**: A typical enterprise project contains far more relevant information than any context window can hold:

```
Enterprise Context vs. Context Window
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Codebase (monorepo):     10M - 100M+ tokens
Documentation:           1M - 10M tokens
Coding standards:        100K - 500K tokens
Architecture decisions:  100K - 300K tokens
Team conventions:        50K - 200K tokens
Regulatory requirements: 100K - 1M tokens
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Available context:       128K - 200K tokens

Gap: 10M+ tokens vs. 200K available = 50x less than needed
```

### The Attention Scarcity Problem

Even when information fits within the context window, models don't process it equally:

```
Context Information Flow
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Input Tokens:    [System Prompt] [AGENTS.md] [Code Files] [History] [Task]
                 ├──────────────────────────────────────────────────────┤
Attention Paid:   ████████████    ██████████    ████    ████    █████████████
                  (High)         (Medium)      (Low)    (Low)    (Variable)
                  ├──────────────────────────────────────────────────────┤
                  Position bias: Beginning and end get most attention
                  "Lost in the middle" phenomenon affects middle content
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Research Finding (Anthropic Context Rot Study)**:
As context length increases, model accuracy decreases even when staying within token limits. This is called **context rot** - information gets "lost" even when technically present.

---

## Why AI Agents Skip Instructions

### 1. The Attention Budget Problem

```
Token Allocation in a Typical AGENTS.md+Task Request
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AGENTS.md (50K tokens):
  ├─ Project overview        5K  ████ (often skimmed)
  ├─ Tech stack             3K  ███   (sometimes read)
  ├─ Coding standards      15K  ████████████ (partially read)
  ├─ Testing requirements  10K  ████████ (partially read)
  └─ Security guidelines   17K  ████████████ (partially read)

Task Request (10K tokens):
  ├─ What to do             2K  ████ (read)
  ├─ Current code          5K  ████████ (read)
  └─ Expected output       3K  █████ (read)

History (40K tokens):
  ├─ Previous messages    30K  ███████ (summarized/skipped)
  └─ Tool outputs         10K  ███ (often ignored)

Relevant Code Files (60K tokens):
  └─ Actual code to modify 60K  ██████████ (selected portions read)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Total: 160K tokens (at capacity!)

What actually influences output: ~20-30K tokens of focused attention
```

### 2. The "Lost in the Middle" Phenomenon

Research consistently shows that LLMs exhibit **position bias** - they pay more attention to information at the beginning and end of prompts:

```
Information Recall Probability by Position
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Position in Context:
Beginning ████████████████████████████████  (highest recall)
          ██████████████████████████████
          ████████████████████████████
          ██████████████████████████
          █████████████████████████
          ███████████████████████
          █████████████████████
          ████████████████████
          ███████████████████
          ██████████████████
          █████████████████
Middle    █████████████████  (lowest recall)
          █████████████████
          ██████████████████
          ███████████████████
          ████████████████████
          █████████████████████
          ███████████████████████
          █████████████████████████
          ████████████████████████████
          ████████████████████████████████ (end)
End       ████████████████████████████████
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Result: Information in the middle of long contexts is often forgotten
```

### 3. Context Overflow and Forced Truncation

When context exceeds limits, systems must truncate:

```
Truncation Strategies (what gets lost)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Strategy 1: Naive truncation
├─ Oldest messages dropped first
├─ Critical setup instructions lost
└─ AGENTS.md content may be truncated

Strategy 2: Sliding window
├─ Keep only recent N messages
├─ Historical context lost
└─ Project-wide rules forgotten

Strategy 3: Smart summarization
├─ Summaries lose nuance
├─ Specific rules become vague
└─ "Specific" becomes "general"

Strategy 4: Priority-based removal
├─ Some rules marked "important"
├─ But all rules seem important
└─ Subjective decisions needed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 4. Token Economy Forces Trade-offs

Every token in context displaces another. This creates inevitable choices:

```
Token Budget Allocation (200K context)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Option A: Maximize instructions
├─ AGENTS.md content: 100K (50%)
├─ Task context:      50K (25%)
├─ Code files:        40K (20%)
└─ History:           10K (5%)

Option B: Maximize relevant code
├─ AGENTS.md content: 30K (15%)
├─ Task context:      20K (10%)
├─ Code files:       130K (65%)
└─ History:           20K (10%)

Option C: Maximize conversation
├─ AGENTS.md content: 20K (10%)
├─ Task context:      15K (8%)
├─ Code files:        25K (12%)
└─ History:          140K (70%)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

No option is optimal - each sacrifices something important
```

---

## The Multi-Agent System Failure Taxonomy

Research from UC Berkeley, Stanford, and other institutions has systematically identified why AI agent systems fail. The **MAST** (Multi-Agent System Failure Taxonomy) identifies 14 failure modes across 3 categories:

### Category 1: System Design Issues

| Failure Mode | Description | Frequency |
|--------------|-------------|-----------|
| Context overflow | Information exceeds capacity | Very High |
| Information loss | Critical details forgotten | High |
| Instruction contradiction | Conflicting rules | Medium |
| Goal drift | Task scope expands unintentionally | Medium |
| State corruption | Agent loses track of progress | High |

### Category 2: Inter-Agent Misalignment

| Failure Mode | Description | Frequency |
|--------------|-------------|-----------|
| Tool misuse | Wrong tool for task | High |
| Output incompatibility | Sub-agent outputs don't integrate | Medium |
| Assumption mismatch | Different agents assume different things | High |
| Communication loss | Messages between agents dropped | Medium |

### Category 3: Task Verification

| Failure Mode | Description | Frequency |
|--------------|-------------|-----------|
| Hallucination | Agent invents non-existent information | High |
| Incomplete execution | Task partially done | High |
| Quality degradation | Output quality drops over time | Very High |
| Rule violation | Instructions not followed | Very High |

**Key Finding**: The most common failure mode is **rule violation** - agents failing to follow explicitly stated instructions, even when those instructions are prominently displayed.

---

## Why Rules Are Violated Even When Present

### Root Cause 1: Attention is Finite

```
Rule Processing During Code Generation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Rules in AGENTS.md (50 rules, ~15K tokens):
  ████████████████████████████████████████ 50 rules declared
  ████████████████████████████            ~15 rules actually processed
  ██████████████                          ~6 rules deeply considered
  ████████                                ~3 rules actively applied

What Actually Happens:
  ✓ Rule 1: Use TypeScript strict mode     (Seen, applied)
  ✓ Rule 2: Write tests for new code       (Seen, applied)
  ✓ Rule 3: Follow project coding style    (Seen, partially applied)
  ✗ Rule 4: Use Result types for errors    (Skipped - too many rules before it)
  ✗ Rule 5: Never use any type             (Skipped - buried in middle)
  ✗ Rule 6: Document public APIs            (Skipped - position in middle)
  ✗ Rule 7: Use specific error codes        (Skipped - too many rules)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Result: Only 50-60% of rules are reliably followed
```

### Root Cause 2: Competing Priorities

When rules conflict, models must choose:

```
Rule Conflict Example
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AGENTS.md Rule A:
  "Write comprehensive tests for all new code"

AGENTS.md Rule B:
  "Complete tasks quickly to meet deadlines"

Task:
  "Add a simple utility function"

Agent's internal conflict:
  ├─ Comprehensive tests = 2 hours
  ├─ Quick implementation = 15 minutes
  └─ Which rule has priority? (AGENTS.md doesn't say!)

Common Outcome: Agent guesses, often choosing "quick"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Root Cause 3: Ambiguity in Application

Many rules require judgment that models may not have:

```
Vague Rule Example
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AGENTS.md:
  "Follow the existing code patterns"

Problem: What patterns?
  ├─ File A: Uses classes
  ├─ File B: Uses functions
  ├─ File C: Uses both
  └─ Which one is "the pattern"?

Agent's uncertainty:
  ├─ Guesses based on most recent file
  ├─ May choose wrong pattern
  └─ Pattern not explicitly stated
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Root Cause 4: Prompt Distraction

```
Conversation History Pollution
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Turn 1: User asks to add feature X
Turn 2: Agent asks clarification question
Turn 3: User answers question
Turn 4: Agent implements feature X (mostly correct)
Turn 5: User asks about unrelated feature Y
Turn 6: Agent implements feature Y (different patterns)
Turn 7: User asks to modify feature X again
Turn 8: Agent has mixed context from features X and Y

Result: Feature X changes don't match original patterns
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## The Practical Reality Check

### What AGENTS.md CAN Do

```
AGENTS.md Effectiveness Spectrum
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Highly Effective:
  ✓ Project overview (what the project IS)
  ✓ Tech stack identification
  ✓ Build/test commands
  ✓ Directory structure
  ✓ Basic naming conventions

Moderately Effective:
  ~ Code style preferences
  ~ Testing requirements
  ~ Error handling patterns
  ~ Documentation expectations

Less Effective:
  ✗ Detailed coding standards (many rules)
  ✗ Complex architectural principles
  ✗ Regulatory/compliance requirements
  ✗ Security policies
  ✗ Team-specific conventions

Ineffective:
  ✗ Everything that exceeds ~30K tokens
  ✗ Rules requiring judgment calls
  ✗ Rules that contradict each other
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### What Real Projects Actually Need

```
Enterprise Project Rule Inventory
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Coding Standards (varies by language/framework):
  ├─ Style guides                    50-200 rules
  ├─ Linting configurations          100-500 rules
  ├─ TypeScript strictness           20-50 rules
  └─ Code organization              30-100 rules

Architecture Principles:
  ├─ Layer separation               10-30 rules
  ├─ Dependency rules               20-50 rules
  ├─ API design standards           20-50 rules
  └─ Data handling patterns         30-100 rules

Security Requirements:
  ├─ Authentication patterns        20-50 rules
  ├─ Authorization rules            30-100 rules
  ├─ Data protection               50-200 rules
  └─ Compliance requirements       100-500 rules

Testing Standards:
  ├─ Test organization             10-30 rules
  ├─ Coverage requirements         5-10 rules
  ├─ Mocking patterns              20-50 rules
  └─ Integration requirements      30-100 rules

Documentation Requirements:
  ├─ API documentation             10-30 rules
  └─ Code comments                20-50 rules

Operational Guidelines:
  ├─ Deployment procedures         50-200 rules
  ├─ Monitoring requirements       20-50 rules
  └─ Incident response            30-100 rules

Total: 500-2000+ rules = 1M+ tokens worth of guidance
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Reality: AGENTS.md can effectively convey ~50-100 of these rules
```

---

## Why AI Agent Work Can Still Be Used

Despite limitations, AI agent outputs can be valuable when properly managed:

### 1. Human-in-the-Loop Verification

```
Workflow: AI Generate → Human Review → Production
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AI Output Quality     Human Review           Final Quality
     70%      →     Catches 25% errors    →    95% acceptable
     80%      →     Catches 15% errors    →    97% good
     90%      →     Catches 8% errors     →    98% excellent

Key insight: Human review catches what AI missed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 2. Iterative Refinement

```
Multi-Pass Approach
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Pass 1: AI generates initial implementation
  └─ Catches basic requirements
  
Pass 2: AI reviews its own code against rules
  └─ Catches obvious violations
  
Pass 3: AI runs automated checks (lint, test)
  └─ Catches technical errors
  
Pass 4: Human reviews final output
  └─ Catches subtle issues
  
Pass 5: AI addresses human feedback
  └─ Final polish

Result: Quality approaches 100% through iteration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 3. Scope Limitation

```
Working Within Constraints
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Instead of: "Follow ALL project rules"
Do: "Follow these 5 specific rules for this change"

Instead of: "Understand the entire codebase"
Do: "Focus on these 3 files and follow these patterns"

Instead of: "Implement feature X with all best practices"
Do: "Implement feature X, here are the exact patterns to follow"

Key insight: Narrow scope = higher compliance
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Strategies for Working Within Limitations

### Strategy 1: Tiered Rule System

```markdown
# AGENTS.md with Tiered Rules

## TIER 1: CRITICAL (Always Follow)
These rules are enforced and breaking them will cause CI failures:
- Use TypeScript strict mode
- Run linting before committing
- Write tests for new functions

## TIER 2: IMPORTANT (Usually Follow)
Deviations should be reviewed but won't fail CI:
- Use functional patterns over classes
- Document public APIs
- Handle errors explicitly

## TIER 3: GUIDANCE (Consider Following)
Style preferences and suggestions:
- Prefer const over let
- Use arrow functions for callbacks
- Sort imports alphabetically

## See Also
- Full coding standards: docs/standards.md
- Security requirements: docs/security.md
- Architecture patterns: docs/patterns.md
```

### Strategy 2: Rule Prioritization in Context

```markdown
## HIGH PRIORITY - MUST FOLLOW
1. **Security First**: Never commit secrets or API keys
2. **Testing Required**: All new functions must have unit tests
3. **Type Safety**: Never use `any` type

## MEDIUM PRIORITY - SHOULD FOLLOW
4. **Error Handling**: Use Result types for fallible operations
5. **Documentation**: Document all exported functions
6. **Code Review**: Request review before merging

## REFERENCE - LOOK UP AS NEEDED
- Full style guide: docs/style-guide.md
- API patterns: docs/api-standards.md
- Testing patterns: docs/testing.md
```

### Strategy 3: Specific Pattern Examples

```markdown
## Correct Error Handling Pattern
```typescript
// ✅ DO THIS
type Result<T, E = Error> = 
  | { ok: true; value: T }
  | { ok: false; error: E };

async function fetchUser(id: string): Promise<Result<User>> {
  try {
    const user = await db.users.find(id);
    return { ok: true, value: user };
  } catch (error) {
    return { ok: false, error: new UserNotFoundError(id) };
  }
}
```

## ❌ DON'T DO THIS
```typescript
// ❌ DON'T use exceptions for expected errors
async function fetchUser(id: string): Promise<User> {
  const user = await db.users.find(id);
  if (!user) throw new Error('User not found');
  return user;
}
```
```

### Strategy 4: Automated Enforcement

```
Rule Enforcement Pipeline
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AI-Generated Code
       ↓
   Linting (100+ rules automatically checked)
       ↓
   Type Checking (TypeScript strict)
       ↓
   Test Coverage Check (80% minimum)
       ↓
   Security Scan (secret detection)
       ↓
   Human Code Review
       ↓
   CI/CD Pipeline
       ↓
Production (only if all automated checks pass)

Result: Rules are enforced by automation, not by AI memory
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Strategy 5: Incremental Context Loading

```typescript
// Instead of: Load ALL rules at once
const allRules = loadAGENTSMD(); // 50K tokens

// Do: Load rules on-demand
async function getRelevantRules(task: string): Promise<string> {
  const taskType = classifyTask(task);
  
  switch (taskType) {
    case 'new-feature':
      return rulesDatabase.getRules([
        'typescript-strict',
        'testing-required',
        'error-handling',
        'documentation'
      ]);
    case 'bug-fix':
      return rulesDatabase.getRules([
        'testing-required',
        'regression-prevention',
        'changelog'
      ]);
    case 'refactoring':
      return rulesDatabase.getRules([
        'breaking-changes',
        'test-updates',
        'documentation'
      ]);
  }
}
```

---

## The Honest Assessment

### Why Simple Prompts Don't Solve the Problem

```
Prompt Effectiveness vs. Complexity
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Prompt: "Follow the project rules"
  └─ Effectiveness: ~20% (vague, no specific rules)
  
Prompt: "Follow these rules: 1, 2, 3..."
  └─ Effectiveness: ~40% (specific, but many rules lost)
  
Prompt: "Follow AGENTS.md file"
  └─ Effectiveness: ~50% (file loaded, but rules in middle ignored)
  
Prompt: "Use these 5 specific rules for this task"
  └─ Effectiveness: ~80% (narrow, focused, present)
  
Prompt: "Automated enforcement checks your code"
  └─ Effectiveness: ~95% (external enforcement, not AI memory)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Key insight: External enforcement > AI memory
```

### The Fundamental Truth

AI agents are **not** reliable rule followers when:
1. Rules exceed attention capacity (~20-30K tokens of effective attention)
2. Rules are in the middle of long contexts (lost in the middle)
3. Rules compete or contradict each other
4. Rules require judgment that training didn't cover

**The solution is NOT better prompts** - it's:
1. **Narrower scope** - fewer rules, specific to task
2. **External enforcement** - automation checks what AI forgets
3. **Iterative verification** - multiple passes catch errors
4. **Human review** - catch what automation misses
5. **Tiered approach** - critical rules automated, guidance rules suggested

---

## Practical Recommendations for Real Projects

### For Project Setup

```markdown
# AGENTS.md (Keep under 20K tokens)

## Project Overview
Brief description

## Critical Rules (Automated Enforcement)
1. TypeScript strict mode - CI will fail if violated
2. Test coverage 80% - CI will fail if violated
3. No secrets in code - CI will fail if violated

## Key Patterns (With Examples)
- Error handling: see docs/error-handling.md
- API design: see docs/api-patterns.md
- Testing: see docs/testing.md

## Commands
- `pnpm test` - Run all tests
- `pnpm lint` - Run linter
- `pnpm typecheck` - Run TypeScript

## See Full Documentation
- Coding standards: docs/standards/
- Architecture: docs/architecture/
- Security: docs/security/
```

### For Code Reviews

```
AI-Assisted Code Review Workflow
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. AI generates code
   ↓
2. Automated checks (lint, type, tests)
   ↓
   ├─ Fails? → AI fixes
   └─ Passes? → Continue
3. AI self-review against AGENTS.md
   ↓
4. Human code review
   ↓
   ├─ Issues? → AI fixes
   └─ Approve? → Merge
5. CI final validation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Every rule that's automated doesn't need AI to remember it
```

### For Team Practices

```
Rule Management Best Practices
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Keep AGENTS.md < 20K tokens
2. Automate rule enforcement where possible
3. Link to detailed docs for complex rules
4. Prioritize rules (critical > important > guidance)
5. Use code examples for important patterns
6. Update rules based on failure patterns
7. Measure rule compliance rates
8. Iterate on prompt effectiveness
9. Accept AI will miss ~20-40% of rules
10. Have humans catch what AI misses
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Conclusion

The user's observation is fundamentally correct: **AI agents cannot reliably follow complex rule sets** because of fundamental architectural limitations:

1. **Context window limits** prevent all rules from being present
2. **Attention scarcity** means even present rules are not all processed
3. **Context rot** causes degradation as context grows
4. **Rule conflicts** require judgment AI may not have

**AGENTS.md is not a complete solution** - it's one tool among many. Effective AI-assisted development requires:

- **Narrow, specific prompts** rather than comprehensive rule documents
- **Automated enforcement** to catch what AI forgets
- **Human review** for judgment-intensive decisions
- **Iterative refinement** to catch errors over multiple passes
- **Realistic expectations** about AI capabilities

The future of AI-assisted development is not "AI that follows all rules perfectly" but "AI that generates good first drafts that humans and automation refine into production quality."

---

## References and Further Reading

1. Anthropic Engineering: "Effective Context Engineering for AI Agents" - https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
2. MAST Paper: "Why Do Multi-Agent LLM Systems Fail?" - arXiv:2503.13657
3. "Context Rot" Research - https://research.trychroma.com/context-rot
4. Factory AI: "The Context Window Problem" - https://factory.ai/news/context-window-problem
5. "How Long Contexts Fail" - https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html
