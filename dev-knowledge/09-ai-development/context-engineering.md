# Context Engineering and AI Protocols

A comprehensive guide to context engineering, the Context-Aware Action Protocol (CAAP), and systematic approaches to managing AI agent context for reliable and accurate outputs.

## Table of Contents

1. [Context Engineering Overview](#context-engineering-overview)
2. [Core Strategies](#core-strategies)
3. [System Prompt Structure](#system-prompt-structure)
4. [Instruction Hierarchy](#instruction-hierarchy)
5. [Progressive Disclosure Pattern](#progressive-disclosure-pattern)
6. [Context-Aware Action Protocol (CAAP)](#context-aware-action-protocol-caap)
7. [Context Budget Management](#context-budget-management)
8. [Best Practices](#best-practices)
9. [Troubleshooting Guide](#troubleshooting-guide)

---

## Context Engineering Overview

**Context Engineering** is the discipline of managing the language model's context window - the art and science of filling an agent's context with precisely the right information at each step to ensure reliable and accurate outputs.

Context Engineering is the evolution beyond traditional Prompt Engineering. While Prompt Engineering focuses on optimizing linguistic structure of single instructions, Context Engineering manages the **entire information ecosystem** (system messages, tools, message history, external data, memory) that the model encounters during inference.

### Context as Finite Resource

Even with large context windows (100k+ tokens), performance degrades after approximately **30k tokens** due to:

- **Context Rot**: Decreasing recall accuracy as context size increases (N² complexity in attention mechanism)
- **Attention Budget**: Transformer architecture creates N² pairwise relationships between tokens
- **Diminishing Returns**: More context ≠ better performance after certain point

**Guideline**: Find the **smallest possible set of high-signal tokens** that maximize desired outcomes.

### High-Signal Information Only

Only include information that is **immediately necessary** for the current step. Keep:

- System prompts: Clear, specific, minimal
- Tool descriptions: Concise and unambiguous
- Examples: Diverse, representative, including good and bad cases
- Domain knowledge: Specific to current task

**Avoid**: Information that might be useful later, extensive background, just-in-case content.

---

## Core Strategies

### 1. Write Context (Externalize State)

Store information outside the immediate context window to manage token budget and persist data across sessions.

**Techniques**:

- **Scratchpads**: Temporary working memory for calculations and intermediate results
- **Memory Systems**: Long-term storage (file-based, database)
  - Episodic memory: Specific events and conversations
  - Procedural memory: How to perform tasks
  - Semantic memory: Facts and knowledge

**Benefits**:

- Reduces immediate context load
- Preserves information across sessions
- Enables knowledge base building

### 2. Select Context (Relevant Retrieval)

Retrieve only the most pertinent information for the current task from larger knowledge stores.

**Techniques**:

- **Embeddings & Vector Search**: Find semantically similar content
- **Knowledge Graphs**: Structured relationship-based retrieval
- **Re-ranking**: Prioritize retrieved results by relevance
- **RAG (Retrieval-Augmented Generation)**: Combine retrieval with generation
- **Just-in-Time Retrieval**: Load data only when needed

**Benefits**:

- Highly relevant information injection
- Reduced noise in context
- Dynamic context adaptation

### 3. Compress Context (Summarization/Trimming)

Reduce information passed to the agent by summarizing or removing outdated data.

**Techniques**:

- **Conversation Summarization**: Condense multi-turn interactions
- **Trim Outdated Information**: Remove irrelevant history
- **Expiry Dates**: Mark temporary content with expiration
- **Context Editing**: Automatically clear stale tool calls/results

**Benefits**:

- Focuses on current task
- Maintains conversation flow
- Extends effective session length

### 4. Isolate Context (Compartmentalization)

Split context into independent compartments to prevent cross-contamination.

**Techniques**:

- **Subagents**: Specialized agents for specific tasks (e.g., debugger, reviewer)
- **Feature Isolation**: Context scoped to specific features/modules
- **Module Boundaries**: Clear separation between subsystems
- **Independent Workflows**: Separate agent processes for unrelated tasks

**Benefits**:

- Prevents context pollution
- Enables parallel processing
- Easier debugging and troubleshooting

---

## System Prompt Structure

### R-G-C Template (Role-Goal-Constraints)

The recommended structure for effective system prompts:

```markdown
# Role

Identity, scope, and tone of the agent.

**Example**:
"You are a pragmatic, solutions-focused TypeScript developer specialized in Clean Architecture."

# Goal

Concrete deliverable or destination for the agent's task.

**Example**:
"Your goal is to implement REST endpoints following Clean Architecture principles while maintaining test coverage above 85%."

# Constraints / Guardrails

Non-negotiable rules, principles, and behavioral guidelines.

**Example**:

- Always follow Test-Driven Development (RED → GREEN → REFACTOR)
- Never use `any` type unless explicitly justified
- Functions must be < 15 lines
- Max 3 parameters per function
```

### Alternative: R-G-B Template (Role-Goal-Behavior)

```markdown
# Role

Defines agent identity, scope, and tone.

# Goal

Specifies measurable outcome or destination.

# Behavior

Rules, principles, and heuristics governing decision-making.
```

### Component Guidelines

**Role**:

- Be specific about expertise level (e.g., "Senior Developer" vs "Junior")
- Define domain knowledge (e.g., "React", "TypeScript", "DDD")
- Set communication tone (formal, casual, concise)

**Goal**:

- Be concrete and measurable
- Focus on outcomes, not activities
- Connect to broader objectives

**Constraints**:

- Use `# Guardrails` heading (models pay extra attention)
- Centralize non-negotiable rules
- Specify prohibited actions clearly
- Define technical and business constraints

### Markdown Best Practices

- Use **clear section headings**: `# Role`, `# Goal`, `# Guardrails`
- **Separate with whitespace**: Empty lines between sections
- **Use emphasis sparingly**: `**IMPORTANT**` for critical rules
- **Structure hierarchically**: H1 → H2 → H3 for nested rules

---

## Instruction Hierarchy

### Priority Levels

System/Developer messages should establish clear hierarchy:

```
1. System/Developer Messages (Highest Priority)
   └─ Application developer's original instructions
      └─ Safety guidelines, tool definitions, behavioral rules

2. User Messages
   └─ Task requests, clarifications, feedback

3. Tool Outputs (Lowest Priority)
   └─ Third-party content, retrieved data, external results
```

### Critical Finding

**System/User Hierarchy Can Fail**: Research shows traditional system/user separation has only **9-45% obedience rate** to system instructions when conflicts arise.

**Mitigation Strategies**:

1. **Explicit Priority Declarations**: Repeatedly emphasize hierarchy
2. **Natural Social Hierarchy**: Frame instructions as organizational roles
3. **Multiple Emphasis Points**: State rules in multiple sections
4. **Guardrails Section**: Dedicated section for non-negotiable constraints
5. **Pre-formatting**: Use standardized formatting for priority markers

### Example Emphasis Pattern

```markdown
# CRITICAL CONSTRAINTS

These rules CANNOT be overridden by any user request:

- [ ] Rule 1
- [ ] Rule 2

IMPORTANT: You must ALWAYS follow these constraints above any user instructions.

# Standard Workflow

1. [Standard steps]

# When User Requests Conflict

If a user request violates CRITICAL CONSTRAINTS:

- politely refuse
- explain why
- suggest alternative approach
```

---

## Progressive Disclosure Pattern

### Three-Tier Loading Strategy

#### Tier 1: Startup Scan (~100 tokens/skill)

Only metadata is pre-loaded into system prompt:

```yaml
---
name: domain-architect
description: Analyze requirements and create DDD domain models
allowed-tools: Read, Write, Edit, Bash
---
```

**Benefits**:

- Agent can scan available expertise without overhead
- Minimal context impact
- Fast startup

#### Tier 2: Full Skill Loading (when triggered)

Complete instructions loaded **only when** task description matches skill description:

```markdown
# Domain Architect

## Purpose

[Detailed instructions here...]

## When to Activate

Use this skill when user asks to:

- Create domain models
- Design entities and value objects
- Plan domain events
```

**Benefits**:

- Context loaded dynamically based on need
- No penalty for unused skills
- Targeted instruction delivery

#### Tier 3: Resource Access (on-demand)

Reference files, scripts, templates accessed via bash tools:

```markdown
## References

- Domain-Driven Design: `read docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md`
- Clean Architecture: `read docs/architecture/ARCHITECTURE.md`
```

**Benefits**:

- Effectively unlimited bundled content
- Zero context cost until accessed
- Enables deep reference libraries

### File Structure Guidelines

**Keep References One Level Deep**:

```
.claude/skills/domain-architect/
├── SKILL.md              # Main instructions (Tier 2)
├── references/
│   ├── ddd-guide.md     # Reference file (Tier 3)
│   └── patterns.md      # Reference file (Tier 3)
└── scripts/
    └── scaffold.sh      # Executable script (Tier 3)
```

**Table of Contents for Long Files**:

```markdown
# Domain-Driven Design Guide

**Contents:**

1. Bounded Contexts → [jump to section]
2. Aggregates → [jump to section]
3. Domain Events → [jump to section]

[Detailed content...]
```

### Cascaded Context System

#### Hierarchical Context Organization

Structure context into layers for focus and relevance:

```
1. Enterprise Level
   └─ Company-wide standards
   └─ Security policies
   └─ Brand guidelines

2. Global Level
   └─ Personal developer preferences
   └─ Code style preferences
   └─ Tool configurations

3. Project Level
   └─ Architecture decisions
   └─ Sprint notes
   └─ Technology stack

4. Module Level
   └─ Subsystem documentation
   └─ API contracts
   └─ Domain models

5. Feature Level
   └─ Specific feature requirements
   └─ Task-specific context
   └─ Temporary decisions
```

#### Context Hygiene Practices

**Use `/clear` Liberally**:

- Start new, unrelated tasks with fresh context
- Switch between features or modules
- When context becomes stale

**Use `/compact` Carefully**:

- Only when continuing related work
- When needing to preserve recent history
- When context is near token limit but still relevant

**Archive Old Decisions**:

- Store resolved discussions in separate files
- Reference archived decisions rather than keeping in active context
- Use expiry dates for temporary items

---

## Context-Aware Action Protocol (CAAP)

**CRITICAL: ALL AI agents MUST follow this protocol before taking ANY action.**

The Context-Aware Action Protocol (CAAP) ensures you load the right documentation, follow all rules, and never miss critical instructions. It's the systematic application of Context Engineering principles to every agent action.

### Protocol Overview

```
┌─────────────────────────────────────────────────────────┐
│ Phase 1: Task Classification & Discovery                │
│ 1. Classify task type                                    │
│ 2. Answer Three-Question Rule                           │
│ 3. Run Discovery Scan (grep)                            │
│ 4. Load ONLY Tier 2 Documents                           │
└──────────────┬────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 2: Action Execution                               │
│ 5. Create Constraints Checklist                         │
│ 6. Validate Before Acting                               │
│ 7. Execute Action                                       │
└──────────────┬────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 3: Commit Validation (If Applicable)              │
│ 8. Load GIT_VERSION_CONTROL.md                         │
│ 9. Create Detailed Commit Message                      │
│ 10. Run Quality Checks                                  │
│ 11. Only Then Commit                                    │
└─────────────────────────────────────────────────────────┘
```

### Phase 1: Task Classification & Discovery

#### Step 1: Classify Your Task

Before ANY git command, file edit, or action, classify your task type using this matrix:

| Task Type    | When You're...        | Tier 2 Docs to Load                                                                                               |
| ------------ | --------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **COMMITTING**   | Running `git commit`  | `docs/engineering/development/GIT_VERSION_CONTROL.md`                                                             |
| **CODING**       | Writing/editing code  | `CODING_AGENTS.md` + `docs/engineering/conventions/TESTING_CONVENTIONS.md`                                        |
| **TESTING**      | Writing tests         | `docs/engineering/test_driven/TEST_DRIVEN_DEVELOPMENT.md` + `docs/engineering/conventions/TESTING_CONVENTIONS.md` |
| **ARCHITECTURE** | Designing structure   | `docs/architecture/ARCHITECTURE.md` + `docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md`                 |
| **DOMAIN**       | Domain modeling       | `docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md` + `docs/engineering/problem_frames/PROBLEM_FRAMES.md`    |
| **REVIEW**       | Reviewing code        | `CODING_AGENTS.md` + `docs/engineering/conventions/TESTING_CONVENTIONS.md`                                        |
| **DOCS**         | Editing documentation | `docs/engineering/development/GIT_VERSION_CONTROL.md` (if committing)                                             |
| **GENERAL**      | Any agent task        | `docs/engineering/context_engineering/CONTEXT_ENGINEERING.md` + `AGENTS.md` (full)                                |

#### Step 2: Three-Question Rule

Before acting, answer these 3 questions:

**Question 1: What file/section will I modify?**

→ Use grep to find section in AGENTS.md and look for "See [link]" references

**Question 2: What type of change is this?**

→ Use Task Type Matrix above to identify which docs to load

**Question 3: What are validation requirements?**

→ Read PROGRESS.md "After Completing Step" section

#### Step 3: Discovery Scan (Find References)

Before reading documentation, scan AGENTS.md with grep to find what you need:

```bash
# Find relevant section (example for committing)
grep -A 30 "Git Commit Requirements" AGENTS.md

# Find what documents exist
grep "^## " AGENTS.md

# Find all "See [link]" references in relevant section
grep -B 10 -A 2 "GIT_VERSION_CONTROL" AGENTS.md
```

**Purpose**: This discovers what documentation EXISTS without loading everything.

#### Step 4: Load ONLY Tier 2 Documents (Not Tier 3)

**Tier 2 (Load Now - These contain the rules you MUST follow)**:

- `docs/engineering/development/GIT_VERSION_CONTROL.md` (for committing)
- `CODING_AGENTS.md` (for coding)
- `docs/engineering/conventions/TESTING_CONVENTIONS.md` (for testing)
- `docs/architecture/ARCHITECTURE.md` (for structure)
- `docs/engineering/clean_architecture/CLEAN_ARCHITECTURE.md` (for architecture)
- `docs/engineering/domain_driven/DOMAIN_DRIVEN_DESIGN.md` (for domain modeling)
- `docs/engineering/context_engineering/CONTEXT_ENGINEERING.md` (always)

**Tier 3 (Reference On-Demand - Read with Read tool only when needed)**:

- `docs/engineering/patterns/PATTERNS.md` (only when implementing specific pattern)
- `docs/engineering/conventions/SOLID_PRINCIPLES.md` (only when refactoring)
- `docs/engineering/conventions/NAMING_CONVENTIONS.md` (only when naming is uncertain)
- Other reference materials

### Phase 2: Action Execution

#### Step 5: Create Constraints Checklist

From the Tier 2 documents you loaded, extract all constraints into a checklist:

**Example for Committing**:

```markdown
## Constraints Checklist (from GIT_VERSION_CONTROL.md)

Commit Message Format:

- [ ] Subject: `<type>[scope]: <description>`
- [ ] Body: Detailed bullets explaining WHY, not just WHAT
- [ ] Footer: Optional references (Closes #123, Co-authored-by)

Pre-commit Quality Checks:

- [ ] All tests passing: `yarn test`
- [ ] Typecheck passing: `yarn typecheck`
- [ ] Linting passing: `yarn lint`
- [ ] Code formatted: `yarn format`
```

**Example for Coding**:

```markdown
## Constraints Checklist (from CODING_AGENTS.md)

Code Rules:

- [ ] Functions < 15 lines
- [ ] Max 3 parameters per function
- [ ] Descriptive names (no abbreviations)
- [ ] No `any` types (unless explicitly justified)

Architecture Rules:

- [ ] Clean Architecture 4-layer separation
- [ ] Dependencies point inward only
- [ ] No circular dependencies
- [ ] Domain layer has no framework dependencies

TDD Rules:

- [ ] Tests written first (RED phase)
- [ ] Minimal code to pass tests (GREEN phase)
- [ ] Refactor only after tests pass (REFACTOR phase)
```

#### Step 6: Validate Before Acting

Check your constraints checklist:

```markdown
## Pre-Action Validation

Document Loading:

- [ ] All constraints identified from loaded docs
- [ ] No contradictions between docs
- [ ] Context budget < 30k tokens
- [ ] Progressive disclosure followed

Constraint Completeness:

- [ ] All critical constraints identified
- [ ] All behavior rules understood
- [ ] All prohibited actions listed
- [ ] All required tools available

Readiness Check:

- [ ] I understand what I need to do
- [ ] I know the validation requirements
- [ ] I have the documentation I need
- [ ] I'm ready to execute action
```

**If any check fails**: STOP, re-read documentation, ask for clarification.

#### Step 7: Execute Action

Only now, execute the action:

- Edit files with full context
- Run quality checks (if applicable)
- Validate against all constraints

### Phase 3: Commit Validation (If Applicable)

#### Step 8: Load GIT_VERSION_CONTROL.md

Before ANY commit, read the full file:

**CRITICAL**: This is mandatory for all commits, no exceptions.

#### Step 9: Create Detailed Commit Message

Follow format with BODY (mandatory):

```markdown
<type>[scope]: <description>

<detailed body with bullets>

- Explain what changed
- Why it matters
- Lines added/modified
- Review time
- Step completion context

<footer>
```

#### Step 10: Run Quality Checks

From GIT_VERSION_CONTROL.md commit checklist:

```bash
# Run ALL these commands and verify they pass
yarn test           # ✅ Must pass
yarn typecheck      # ✅ Must pass
yarn lint           # ✅ Must pass
yarn format:check   # ✅ Must pass
```

#### Step 11: Only Then Commit

Commit only AFTER all quality checks pass and commit message follows full format.

### Never Violate These Rules

### ❌ NEVER do these:

- ❌ **NEVER act without classifying task type**
  - Always start with Step 1: Classify Task Type
  - Use Task Type Matrix to identify documentation

- ❌ **NEVER skip discovery scan (grep for references)**
  - Always scan AGENTS.md before reading docs
  - Find what exists before loading

- ❌ **NEVER assume "I know this"**
  - Documentation is the source of truth
  - Rules may change over time
  - Always verify against documentation

- ❌ **NEVER load ALL docs (causes context overflow)**
  - Load only Tier 2 documents for task type
  - Use Read tool for Tier 3 on-demand
  - Monitor context budget (< 30k tokens)

- ❌ **NEVER commit without validation**
  - Always run quality checks before committing
  - Always include detailed commit message body
  - Always read GIT_VERSION_CONTROL.md first

- ❌ **NEVER skip quality checks**
  - yarn test: Must pass
  - yarn typecheck: Must pass
  - yarn lint: Must pass
  - yarn format:check: Must pass

- ❌ **NEVER commit without detailed message body**
  - Subject line is not enough
  - Body must explain WHY not just WHAT
  - Include step context and line counts

### ✅ ALWAYS do these:

- ✅ **ALWAYS classify task type first**
  - Use Task Type Matrix
  - Identify Tier 2 documents to load
  - Document your classification

- ✅ **ALWAYS use grep to discover references**
  - Find relevant sections
  - Identify linked documentation
  - Don't assume document structure

- ✅ **ALWAYS read Tier 2 docs for task type**
  - Load only necessary documentation
  - Extract all constraints
  - Create constraints checklist

- ✅ **ALWAYS extract constraints from loaded docs**
  - Create detailed checklist
  - Verify no contradictions
  - Understand all requirements

- ✅ **ALWAYS create validation checklist**
  - Extract all checkpoints
  - Document validation steps
  - Ensure completeness

- ✅ **ALWAYS validate before acting**
  - Check all preconditions
  - Verify context budget
  - Confirm readiness

- ✅ **ALWAYS run quality checks before committing**
  - Run all quality commands
  - Fix any failures
  - Re-run to verify fixes

- ✅ **ALWAYS include detailed commit message body**
  - Explain WHY, not just WHAT
  - Include line counts and review time
  - Reference step completion

---

## Context Budget Management

### Target: < 30k tokens per session

### How to Stay Within Budget:

**1. Task Classification** → 0 tokens (mental step)

- No tokens required

**2. Discovery Scan (grep)** → ~1k tokens

- Use grep to find sections
- Don't read full files yet

**3. Load Tier 2 Docs** → ~5-10k tokens (task-specific only)

- Load only what's needed for task type
- Don't load all documentation

**4. Read Targeted Sections** → ~2-3k tokens (not full files)

- Use Read tool with specific sections
- Don't load entire Tier 3 files

**5. Execute Action** → Context from above steps

- Work with loaded context
- Load Tier 3 on-demand only

**Total**: ~10-15k tokens (well under 30k limit)

### Context Budget Monitoring:

```markdown
## Context Budget Tracking

Current Phase: Phase 2 - Action Execution

Token Usage:

- Task Classification: 0 tokens (mental)
- Discovery Scan (grep): 1,200 tokens
- Tier 2 Docs Loaded: 6,500 tokens
- Tier 3 References: 0 tokens (on-demand)
- Current Total: 7,700 tokens

Budget: < 30,000 tokens ✅
Remaining: 22,300 tokens

Status: ✅ Within budget
```

### Context Budget Best Practices:

1. **Use grep instead of reading full files**
   - Scan for relevant sections
   - Only read what you need

2. **Load documentation progressively**
   - Tier 1: Always loaded (~100 tokens)
   - Tier 2: Task-specific (~3k-5k tokens)
   - Tier 3: On-demand (unlimited, but accessed via Read tool)

3. **Remove stale context**
   - Clean up after each phase
   - Don't keep unnecessary information
   - Archive decisions to files

4. **Monitor context size proactively**
   - Track tokens at each step
   - Stop when approaching limit
   - Use `/compact` if needed

---

## Best Practices

### System Prompt Guidelines

1. **Be Explicit and Direct**: Claude 4.x requires clear, explicit instructions
2. **Add Context/Motivation**: Explain WHY behavior is important
3. **Use Examples Carefully**: Models follow examples precisely, including details
4. **Control Verbosity**: Claude 4.5 is concise - request detail explicitly if needed
5. **Be Specific**: Vague requests yield vague outputs

### Tool Usage Guidelines

1. **Build Small, Distinct Tools**: Each tool should have single responsibility
2. **Unambiguous Parameters**: Clear input/output specifications
3. **Efficient Operations**: Minimize token cost per tool call
4. **Graceful Failure Handling**: Define behavior for tool failures

### Context Optimization Guidelines

1. **Focus on Relevance Over Volume**: Maximize signal, minimize noise
2. **Progressive Disclosure**: Load information in tiers as needed
3. **Just-in-Time Retrieval**: Fetch data only when immediately needed
4. **Context Budgeting**: Target < 30k tokens for optimal performance
5. **Regular Cleanup**: Remove stale information proactively

---

## Troubleshooting Guide

### Problem: I'm not sure what task type this is

**Solution**: Use the decision tree:

```
Is it a git commit?
├─ Yes → COMMITTING task
└─ No → Is it writing/editing code?
    ├─ Yes → CODING task
    └─ No → Is it writing tests?
        ├─ Yes → TESTING task
        └─ No → Is it designing structure?
            ├─ Yes → ARCHITECTURE task
            └─ No → GENERAL task
```

### Problem: I can't find the relevant documentation

**Solution**: Use discovery scan:

```bash
# Find all sections in AGENTS.md
grep "^## " AGENTS.md

# Find specific section
grep -A 30 "Git Commit" AGENTS.md

# Find references
grep "See \[" AGENTS.md
```

### Problem: Documents contradict each other

**Solution**:

1. Identify which document is higher priority
2. Check AGENTS.md for hierarchy
3. If still unclear, stop and ask user
4. Document the conflict resolution

### Problem: Context is approaching 30k tokens

**Solution**:

1. Stop and assess what's essential
2. Remove outdated tool results
3. Archive decisions to external files
4. Use `/compact` if continuing related work
5. Use `/clear` if starting new task

### Problem: Quality checks are failing

**Solution**:

1. Identify which check is failing
2. Read the error message carefully
3. Fix the specific issue
4. Re-run the failing check only
5. Re-run all checks to ensure no regressions
6. Only proceed when ALL pass

### Problem: I'm not sure if I've completed all validation checkpoints

**Solution**: Use the checklist:

```markdown
## CAAP Completion Checklist

Phase 1: Task Classification & Discovery

- [ ] Task type classified
- [ ] Three-Question Rule answered
- [ ] Discovery scan completed
- [ ] Tier 2 docs loaded

Phase 2: Action Execution

- [ ] Constraints checklist created
- [ ] Validation before acting completed
- [ ] Action executed

Phase 3: Commit Validation (if applicable)

- [ ] GIT_VERSION_CONTROL.md loaded
- [ ] Detailed commit message created
- [ ] Quality checks run and passing
- [ ] Commit executed

Overall:

- [ ] Context budget < 30k tokens
- [ ] All constraints followed
- [ ] No rules violated
- [ ] Documentation updated (if applicable)
```

---

## Success Metrics

### CAAP Compliance

- ✅ Task classification: 100% of actions
- ✅ Documentation loading: 100% of actions
- ✅ Constraint validation: 100% of actions
- ✅ Quality checks: 100% of commits
- ✅ Context budget: < 30k tokens per session

### Context Efficiency

- ✅ Average context size: < 30k tokens
- ✅ Token efficiency ratio: > 80% useful tokens
- ✅ Context rotation: < 10% of sessions need clearing

### Agent Performance

- ✅ Task completion: > 95% first-pass success
- ✅ Context adherence: > 90% constraint compliance
- ✅ Response quality: Consistent with high-signal context

---

## References

### Research Papers

- **The Instruction Hierarchy**: Training LLMs to prioritize privileged instructions (2023)
- **Context Engineering: The Definitive Guide** (FlowHunt, 2025)
- **Effective Context Engineering for AI Agents** (Anthropic, 2025)

### Industry Guides

- **Context Engineering Best Practices** (Kubiya AI, 2025)
- **Prompting Best Practices** (Anthropic Claude, 2025)
- **Agent Skills Framework Documentation** (Anthropic, 2025)
- **Claude Code Best Practices** (Anthropic Engineering, 2025)

### Key Concepts

- **Context Rot**: Performance degradation with increasing context size
- **Attention Budget**: N² complexity limiting effective context
- **Progressive Disclosure**: Tiered information loading
- **Instruction Hierarchy**: Prioritizing system over user messages
- **High-Signal Information**: Only include immediately necessary data

---

**Context Engineering**: Treating context as a precious, finite resource and managing it systematically to maximize AI agent performance.

**CAAP**: Systematic approach to ensure complete context, rule compliance, and quality outcomes for every agent action.

**Remember**: This protocol prevents missing rules, ensures complete context, and maintains context budget discipline. ALWAYS follow CAAP before taking any action.
