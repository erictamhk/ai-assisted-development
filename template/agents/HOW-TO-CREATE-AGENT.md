# How to Create an Agent Role

A guide for using `agent-role-template.md` to define new AI agent roles.

---

## When to Create a New Agent Role

Create a new agent role when you need a specialist that:

- Performs a distinct type of work (not covered by existing agents)
- Has unique workflows or approaches
- Delivers specific output formats
- Requires domain-specific knowledge access

**Examples of distinct agent types:**
- `RESEARCHER` - Research and synthesize information
- `CODER` - Implement features and fixes
- `ARCHITECT` - Design system structures
- `REVIEWER` - Quality assurance and validation
- `CLARIFIER` - Requirements discovery and refinement
- `PLANNER` - Task breakdown and estimation
- `REFACTORER` - Code improvement and modernization
- `DOCUMENTATOR` - Documentation creation and maintenance

---

## Prerequisite Knowledge

Before creating an agent role, understand:

1. **The Universal Agent Laws** - All agents must follow CLARIFY, FOLLOW PROCESS, PROTECT QUALITY
2. **dev-knowledge Structure** - 13 categories of engineering knowledge
3. **Orchestrator Pattern** - Agents are specialists called by orchestrator
4. **Output Artifacts** - Each agent delivers specific output types

---

## Agent Creation Workflow

### Phase 1: Discovery

**Step 1.1: Define the Gap**
- What work needs to be done?
- Why can't existing agents handle it?
- What makes this work distinct?

**Step 1.2: Identify Output Artifact**
- What will this agent deliver?
- Code files? Markdown docs? Diagrams? Decisions?
- Where will output be stored?

**Step 1.3: Map Knowledge Sources**
- What categories in dev-knowledge/ are relevant?
- Which are CRITICAL vs ON-DEMAND?

---

### Phase 2: Definition

**Step 2.1: Draft R-G-C**

```
Role:    [Specialist title - what this agent IS]
Goal:    [Deliverable - what this agent PRODUCES]
Constraints: [3-5 MUST NOT rules]
```

**Examples:**

```
RESEARCHER:
Role:    Research specialist
Goal:    Research findings as markdown documents
Constraints:
- MUST cite all sources
- MUST distinguish fact from opinion
- NEVER present assumptions as facts

CODER:
Role:    Implementation specialist
Goal:    Working code that passes quality gates
Constraints:
- MUST follow dev-knowledge/ patterns
- MUST write tests for new code
- NEVER commit directly
```

**Step 2.2: Write Purpose**
One sentence: Why does this agent exist?

```
RESEARCHER: Finds and synthesizes information on any topic.
CODER: Implements features and fixes according to specifications.
ARCHITECT: Creates architectural designs that balance constraints.
```

**Step 2.3: List Responsibilities**
3-5 primary responsibilities:

```
RESEARCHER:
1. Research any topic (HRMS, law, finance, etc.)
2. Synthesize findings into actionable insights
3. Document knowledge gaps and recommendations

CODER:
1. Implement features based on specs
2. Write tests that verify behavior
3. Ensure code follows project conventions
```

**Step 2.4: Define Workflow**
4-6 steps the agent follows:

```
RESEARCHER:
1. Classify → What category is this research?
2. Search → grep dev-knowledge/ for patterns
3. Study → Read and understand patterns
4. Research → External search for topic
5. Synthesize → Combine knowledge + research
6. Deliver → Markdown document with citations

CODER:
1. Understand → Read specs, existing patterns
2. Implement → Write code following patterns
3. Test → Write/run tests
4. Validate → Run lint/typecheck
5. Deliver → Code files + tests
```

**Step 2.5: Specify Knowledge Access**
Map relevant dev-knowledge/ categories:

```
RESEARCHER:
CRITICAL: 09-ai-development/ (for research methods)
ON-DEMAND: Any category based on research topic

CODER:
CRITICAL: 04-coding-style/
CRITICAL: 05-testing/
ON-DEMAND: 03-architecture/
```

**Step 2.6: Define Output Format**
Template or format specification:

```
RESEARCHER:
```markdown
# Research: [Topic]

## Summary
[Brief overview]

## Findings
1. [Finding with citation]

## Recommendations
- [Actionable recommendation]

## Sources
- [Source 1]
```
```

---

### Phase 3: Validation

**Step 3.1: Review Against Template**

- [ ] R-G-C is clear and complete
- [ ] Purpose is one sentence
- [ ] Responsibilities are 3-5 items
- [ ] Workflow has 4-6 steps
- [ ] Knowledge access is mapped
- [ ] Output format has template

**Step 3.2: Review Against Universal Laws**

- [ ] Agent can CLARIFY (ask questions when uncertain)
- [ ] Agent can FOLLOW PROCESS (has defined workflow)
- [ ] Agent can PROTECT QUALITY (accesses knowledge, validates)

**Step 3.3: Peer Review**

- Does the role make sense?
- Is it distinct from existing agents?
- Are responsibilities clear?

---

### Phase 4: Documentation

**Step 4.1: Create Agent File**

```
template/agents/
├── agent-role-template.md     ← Copy this template
├── HOW-TO-CREATE-AGENT.md     ← This file
├── researcher.md              ← New agent
├── coder.md                   ← New agent
└── ...
```

**Step 4.2: Update KNOWLEDGE-MAPPING.md**

Add the agent to the knowledge mapping:

```markdown
| RESEARCHER | research.md | 09-ai-development/ | External research methods |
```

**Step 4.3: Register with Orchestrator**

When orchestration system is ready, register the agent:

```yaml
agents:
  researcher:
    role: Research specialist
    goal: Research findings as markdown documents
    input: Research topic
    output: markdown/doc/research/
```

---

## Complete Example: RESEARCHER Agent

### Phase 1: Discovery

**Gap:** Need specialist to research any topic and synthesize findings.

**Output:** Markdown documents with citations.

**Knowledge:** All categories (depends on topic), external search.

---

### Phase 2: Definition

**Draft R-G-C:**

```
Role:    Research specialist
Goal:    Research findings as markdown documents
Constraints:
- MUST cite all sources
- MUST distinguish fact from opinion
- NEVER present assumptions as facts
```

**Purpose:** Finds and synthesizes information on any topic.

**Responsibilities:**
1. Research any topic (HRMS, law, finance, etc.)
2. Synthesize findings into actionable insights
3. Document knowledge gaps and recommendations

**Workflow:**
1. Classify → What category is this research?
2. Search → grep dev-knowledge/ for patterns
3. Study → Read and understand patterns
4. Research → Web search for topic
5. Synthesize → Combine knowledge + research
6. Deliver → Markdown document with citations

**Knowledge Access:**
- CRITICAL: 09-ai-development/
- ON-DEMAND: Any category based on topic

**Output Format:**
```markdown
# Research: [Topic]

## Summary
[Brief overview]

## Findings
1. [Finding with citation]

## Recommendations
- [Actionable recommendation]

## Knowledge Gaps
- [What's missing]

## Sources
- [Source 1]
```

---

### Phase 3: Validation

- [x] R-G-C is clear
- [x] Purpose is one sentence
- [x] 3 responsibilities
- [x] 6 workflow steps
- [x] Knowledge access mapped
- [x] Output format has template
- [x] Follows CLARIFY law
- [x] Follows FOLLOW PROCESS law
- [x] Follows PROTECT QUALITY law

---

### Phase 4: Documentation

Create `template/agents/researcher.md` with the definition above.

Update `projects/ai-agent-framework/KNOWLEDGE-MAPPING.md`.

---

## Agent Creation Checklist

### Before Starting

- [ ] Gap is clearly defined
- [ ] Output artifact is specified
- [ ] Distinct from existing agents

### During Definition

- [ ] R-G-C is complete (3 elements)
- [ ] Purpose is one sentence
- [ ] Responsibilities are 3-5 items
- [ ] Workflow has 4-6 steps
- [ ] Knowledge access is mapped
- [ ] Output format has template

### During Validation

- [ ] Template compliance verified
- [ ] Universal laws satisfied
- [ ] Peer reviewed

### During Documentation

- [ ] Agent file created in template/agents/
- [ ] KNOWLEDGE-MAPPING.md updated
- [ ] Orchestrator registered (future)

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Too many responsibilities | Agent becomes orchestrator | Limit to 3-5 |
| Vague role | Unclear purpose | Be specific |
| No output format | Inconsistent delivery | Define template |
| Missing knowledge access | Agent doesn't know where to find info | Map dev-knowledge/ categories |
| Workflow is too simple | Agent can't complete work | 4-6 steps minimum |
| Constraints are empty | Agent may violate rules | Add 3-5 constraints |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-02 | Initial guide |

---

**Guide Version:** 1.0
