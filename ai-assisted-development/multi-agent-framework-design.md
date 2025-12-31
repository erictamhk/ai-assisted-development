# Multi-Agent Framework for OpenCode

## Executive Summary

This document outlines the design of a hierarchical multi-agent framework that extends OpenCode's built-in agent system. The framework leverages OpenCode's SDK, session management, and AGENTS.md rule loading to orchestrate specialized sub-agents for complex software engineering tasks.

## Alignment with OpenCode Architecture

The framework is built on OpenCode's existing capabilities:

1. **OpenCode Agent System** - Leverages primary agents (Build, Plan) and subagents (General, Explore) as building blocks
2. **SDK for Orchestration** - Uses `@opencode-ai/sdk` for programmatic session and agent management
3. **AGENTS.md Rule Loading** - Extends OpenCode's instruction system with on-demand rule loading
4. **Session Hierarchy** - Manages parent-child session relationships explicitly

## OpenCode Integration Points

### Agent Configuration Files

Framework agents are configured as OpenCode subagents following the Markdown format:

`.opencode/agent/planner.md`
```markdown
---
description: Decomposes requirements into actionable tasks using Impact Mapping and Example Mapping
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.1
maxSteps: 20
tools:
  find.text: true
  find.files: true
  file.read: true
  bash: false
permission:
  file writes: deny
  bash: deny
---
You are a Planning Agent specialized in software architecture and task decomposition.

## Your Responsibilities
1. Analyze user requirements and clarify ambiguities using Example Mapping
2. Apply Impact Mapping to understand business goals and technical deliverables
3. Decompose features into independent vertical slices
4. Identify cross-slice dependencies and architectural concerns

## Workflow
When given a task:
1. First, read the project's AGENTS.md to understand context
2. Use @rules/impact-mapping-framework.md for goal decomposition
3. Use @rules/example-mapping-guide.md for requirement clarification
4. Output a task graph with clear dependencies

## Output Format
Provide your analysis as:
- Task list with dependencies
- Risk assessments
- Implementation order recommendations

## Rules
- Always validate against project AGENTS.md before planning
- Preserve vertical slice boundaries when decomposing
- Flag any architectural concerns explicitly
```

`.opencode/agent/executor.md`
```markdown
---
description: Implements features following TDD/BDD methodologies with Clean Code principles
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.3
maxSteps: 50
tools:
  write: true
  edit: true
  bash: true
  find.text: true
  find.files: true
  file.read: true
permission:
  file writes: allow
  bash:
    "npm test": allow
    "npm run test:*": allow
    "git add *": allow
    "git commit*": ask
---
You are an Execution Agent focused on test-driven development and clean code.

## Your Responsibilities
1. Write failing tests before implementation (TDD)
2. Implement features following Clean Code principles
3. Maintain living documentation alongside code
4. Ensure test coverage meets project standards

## Workflow
For each feature:
1. Read the feature specification from Planner Agent
2. Write failing tests (Red phase)
3. Implement minimal code to pass tests (Green phase)
4. Refactor for clarity and maintainability (Refactor phase)
5. Update living documentation

## Rules
- Never skip the Red-Green-Refactor cycle
- Follow vertical slice architecture for new features
- Update AGENTS.md after architectural decisions
- Ask for clarification on ambiguous requirements

## Reference Documents
@docs/tdd-guidelines.md
@docs/clean-code-principles.md
@rules/architecture-patterns.md
```

`.opencode/agent/reflector.md`
```markdown
---
description: Reviews code for quality, architecture compliance, and potential issues
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.1
maxSteps: 30
tools:
  find.text: true
  find.files: true
  file.read: true
  bash:
    "git diff": allow
    "git log*": allow
permission:
  file writes: deny
  edit: deny
  bash: ask
---
You are a Code Review Agent focused on quality assurance and architecture validation.

## Your Responsibilities
1. Review code for Clean Code principle violations
2. Validate architectural boundary compliance
3. Detect anti-patterns and code smells
4. Suggest specific refactoring improvements
5. Flag security concerns

## Review Checklist
- [ ] Single Responsibility Principle followed
- [ ] Naming is descriptive and consistent
- [ ] Functions are small and focused
- [ ] No duplicate code (DRY principle)
- [ ] Error handling is consistent
- [ ] Architecture boundaries respected
- [ ] No security vulnerabilities

## Rules
- Never modify code without explicit approval
- Provide constructive, actionable feedback
- Reference specific patterns from @docs/clean-code.md
- Flag blocking issues vs. suggestions

## Output Format
- Blocking issues (must fix)
- Suggestions (should consider)
- Nitpicks (optional improvements)
- Learning opportunities (educational notes)
```

## SDK-Based Orchestrator

The framework uses the OpenCode SDK for programmatic orchestration:

```typescript
import { createOpencode, createOpencodeClient } from "@opencode-ai/sdk";

interface OrchestrationSession {
  parentSessionId: string;
  childSessionIds: string[];
  agentAssignments: Map<string, string>;
  taskGraph: TaskGraph;
}

class OpenCodeAgentOrchestrator {
  private client: ReturnType<typeof createOpencodeClient>;
  private rootSessionId: string;
  private agentConfigs: Map<string, AgentConfig>;

  async initialize(projectPath: string): Promise<void> {
    const { client } = await createOpencode({
      hostname: "127.0.0.1",
      port: 4096,
      config: {
        model: "anthropic/claude-sonnet-4-20250514",
        instructions: [
          ".opencode/agent/*.md",
          "docs/development-standards.md",
          "@rules/*.md"
        ]
      }
    });
    this.client = client;

    // Initialize root session
    const rootSession = await client.session.create({
      body: { title: "Multi-Agent Orchestration Session" }
    });
    this.rootSessionId = rootSession.id;

    // Initialize child sessions for each agent
    await this.initializeAgentSessions();
  }

  private async initializeAgentSessions(): Promise<void> {
    const agents = ["planner", "executor", "reflector"];

    for (const agent of agents) {
      const session = await client.session.create({
        body: {
          title: `${agent} Agent Session`,
          agent: agent  // Assign agent to session
        }
      });
      this.agentConfigs.set(agent, { sessionId: session.id });
    }
  }

  async executeTask(task: TaskRequest): Promise<OrchestrationResult> {
    // Step 1: Use Planner Agent to decompose task
    const plannerResult = await this.invokeAgent(
      "planner",
      {
        userRequest: task.description,
        projectContext: await this.getProjectContext(),
        availableAgents: Array.from(this.agentConfigs.keys())
      },
      this.rootSessionId
    );

    const taskGraph = this.parseTaskGraph(plannerResult);

    // Step 2: Execute tasks in dependency order
    const results: Map<string, TaskResult> = new Map();

    for (const taskNode of taskGraph.topologicalOrder) {
      const executorResult = await this.invokeAgent(
        "executor",
        {
          featureSpec: taskNode.specification,
          taskDependencies: taskNode.dependencies.map(d => results.get(d))
        },
        this.agentConfigs.get("executor")!.sessionId
      );

      // Step 3: Reflect on changes
      const reflectorResult = await this.invokeAgent(
        "reflector",
        {
          codeChanges: executorResult.changes,
          qualityStandards: await this.getQualityStandards()
        },
        this.agentConfigs.get("reflector")!.sessionId
      );

      if (reflectorResult.hasBlockingIssues) {
        throw new QualityGateFailedError(reflectorResult.blockingIssues);
      }

      results.set(taskNode.id, {
        changes: executorResult.changes,
        review: reflectorResult,
        tests: executorResult.tests
      });
    }

    return this.synthesizeResults(results, taskGraph);
  }

  private async invokeAgent(
    agentName: string,
    payload: unknown,
    sessionId: string
  ): Promise<AgentResponse> {
    const result = await this.client.session.prompt({
      path: { id: sessionId },
      body: {
        parts: [{ type: "text", text: JSON.stringify(payload) }],
        model: {
          providerID: "anthropic",
          modelID: "claude-sonnet-4-20250514"
        }
      }
    });

    return this.parseAgentResponse(result, agentName);
  }

  async navigateSessionHierarchy(direction: "forward" | "backward"): Promise<void> {
    // Simulate Leader+Right/Left keybind for session navigation
    const keybind = direction === "forward" ? "session_child_cycle" : "session_child_cycle_reverse";
    await this.client.tui.executeCommand({
      body: { command: keybind }
    });
  }

  async cleanup(): Promise<void> {
    // Delete all child sessions
    for (const [agent, config] of this.agentConfigs) {
      await this.client.session.delete({ path: { id: config.sessionId } });
    }
    this.agentConfigs.clear();
  }
}
```

## On-Demand Rule Loading

Extends OpenCode's instruction system with intelligent lazy loading:

`opencode.json`
```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "AGENTS.md",
    ".opencode/agent/*.md",
    "@rules/base-rules.md",
    "@rules/**/*.md"
  ],
  "agent": {
    "framework-orchestrator": {
      "description": "Orchestrates multiple specialized agents for complex tasks",
      "mode": "primary",
      "model": "anthropic/claude-sonnet-4-20250514",
      "temperature": 0.2,
      "maxSteps": 100
    }
  }
}
```

`@rules/base-rules.md` (referenced in AGENTS.md)
```markdown
# Base Rules for Multi-Agent Framework

## Rule Loading Strategy
CRITICAL: This framework uses on-demand rule loading. When you encounter
file references (e.g., @rules/domain-driven-design.md), use your Read tool
to load them on a need-to-know basis.

## External File References
- Architecture Patterns: @rules/architecture-patterns.md
- DDD Patterns: @rules/domain-driven-design.md
- Clean Code Principles: @rules/clean-code.md
- TDD Guidelines: @rules/tdd-guidelines.md
- Test Standards: @rules/testing-standards.md

## Lazy Loading Instructions
1. Load referenced files only when directly relevant to your current task
2. When loading, treat content as mandatory instructions
3. Follow references recursively when encountering new references
4. Cache loaded content for the duration of the session

## Agent Communication Protocol
When invoking subagents via @mention:
1. Provide clear, structured input
2. Include relevant context from loaded rules
3. Specify expected output format
4. Handle errors gracefully with fallback strategies
```

`@rules/architecture-patterns.md`
```markdown
# Architecture Patterns Reference

## Layered Architecture (Ports & Adapters)
- Core Domain: Business logic independent of frameworks
- Application Layer: Use case orchestration
- Infrastructure Layer: External system adapters

## Vertical Slice Architecture
- Features are organized as independent vertical slices
- Each slice contains all components for a feature
- Cross-slice communication via explicit interfaces

## Event Sourcing Pattern
- State changes captured as immutable events
- Full audit trail of all decisions
- Event replay for state reconstruction

## CQRS Pattern
- Command Query Responsibility Segregation
- Separate models for reads and writes
- Optimized for specific use cases
```

## Session Management Strategy

Addressing OpenCode issue #6491 (session navigation) with explicit management:

```typescript
interface SessionHierarchy {
  root: SessionNode;
  children: Map<string, SessionNode>;
}

class SessionNavigator {
  private client: ReturnType<typeof createOpencodeClient>;

  async getSessionTree(rootSessionId: string): Promise<SessionHierarchy> {
    const root = await this.client.session.get({ path: { id: rootSessionId } });
    const children = await this.client.session.children({ path: { id: rootSessionId } });

    return {
      root,
      children: new Map(children.map(c => [c.id, c]))
    };
  }

  async switchToSession(sessionId: string): Promise<void> {
    // Store current position for quick navigation
    const history = await this.getNavigationHistory();
    history.push(sessionId);
    await this.setNavigationHistory(history);

    // Switch to target session
    await this.client.tui.executeCommand({
      body: { command: `session_select ${sessionId}` }
    });
  }

  async returnToParent(): Promise<void> {
    const history = await this.getNavigationHistory();
    if (history.length > 0) {
      const parentId = history.pop();
      await this.setNavigationHistory(history);
      await this.switchToSession(parentId);
    }
  }

  async autoNavigateAfterSubagent(): Promise<void> {
    // Workaround for #6491: explicit parent return
    await this.client.tui.executeCommand({
      body: { command: "session_child_cycle_reverse" }
    });
  }
}
```

## AGENTS.md Integration for Project Context

Root `AGENTS.md` defines framework integration:

```markdown
# Multi-Agent Framework Integration

This project uses a multi-agent framework extending OpenCode with specialized agents.

## Available Agents

### @planner
Task decomposition and planning agent.
- Use for: Feature analysis, task breakdown, dependency identification
- Input: Feature description, project context
- Output: Task graph, implementation plan

### @executor
Implementation agent following TDD methodology.
- Use for: Code implementation, test writing, refactoring
- Input: Task specification, dependencies
- Output: Code changes, tests, documentation updates

### @reflector
Code review and quality assurance agent.
- Use for: Architecture review, code quality analysis, security audit
- Input: Code changes, quality standards
- Output: Review report, improvement suggestions

## Workflow

1. **Planning Phase**
   - Invoke `@planner` with feature description
   - Review task graph and implementation plan
   - Approve or request modifications

2. **Implementation Phase**
   - Invoke `@executor` with approved task specification
   - Monitor progress through session navigation
   - Review test coverage and code quality

3. **Review Phase**
   - Invoke `@reflector` on completed changes
   - Address blocking issues
   - Apply suggestions

## Quality Gates
- All code must pass @reflector review before merge
- Test coverage must meet project standards (80%+)
- Architecture violations are blocking issues
- Security concerns must be addressed immediately

## Rule References
@rules/base-rules.md (loaded on-demand)
@rules/architecture-patterns.md
@rules/domain-driven-design.md
@rules/clean-code.md
@rules/tdd-guidelines.md
```

## Event Streaming for Real-Time Monitoring

```typescript
class AgentEventMonitor {
  private client: ReturnType<typeof createOpencodeClient>;

  async subscribeToSession(sessionId: string): Promise<AsyncIterator<AgentEvent>> {
    const events = await this.client.event.subscribe();

    return (async function* () {
      for await (const event of events.stream) {
        if (event.type === "message" && event.properties.sessionId === sessionId) {
          yield this.parseEvent(event);
        }
      }
    })();
  }

  async monitorTaskProgress(taskId: string): Promise<TaskProgress> {
    const events = await this.subscribeToSession(taskId);
    const progress: TaskProgress = {
      started: false,
      steps: [],
      completed: false,
      blockingIssues: []
    };

    for await (const event of events) {
      switch (event.type) {
        case "task_started":
          progress.started = true;
          break;
        case "step_completed":
          progress.steps.push(event.stepInfo);
          break;
        case "blocking_issue":
          progress.blockingIssues.push(event.issue);
          break;
        case "task_completed":
          progress.completed = true;
          break;
      }
    }

    return progress;
  }
}
```

## Feature-Based Directory Structure

Following Vertical Slice Architecture:

```
.opencode/
├── agent/
│   ├── planner.md         # Planner subagent configuration
│   ├── executor.md        # Executor subagent configuration
│   └── reflector.md       # Reflector subagent configuration
├── rules/
│   ├── base-rules.md      # Base framework rules
│   ├── architecture-patterns.md
│   ├── domain-driven-design.md
│   ├── clean-code.md
│   ├── tdd-guidelines.md
│   └── testing-standards.md
└── opencode.json          # Framework configuration

docs/
├── development-standards.md
└── living-documentation-template.md

scripts/
├── orchestrate.ts         # Orchestrator implementation
├── session-manager.ts     # Session navigation utilities
└── event-monitor.ts       # Real-time monitoring

tests/
├── orchestration/
│   ├── planner.spec.ts
│   ├── executor.spec.ts
│   └── reflector.spec.ts
└── integration/
    ├── session-hierarchy.spec.ts
    └── event-streaming.spec.ts
```

## Implementation Roadmap

### Phase 1: Foundation
- [ ] Create `.opencode/agent/` directory structure
- [ ] Configure planner, executor, reflector subagents
- [ ] Create base rules in `@rules/` format
- [ ] Test agent invocation via @mention

### Phase 2: SDK Integration
- [ ] Implement `OpenCodeAgentOrchestrator` class
- [ ] Set up session creation and management
- [ ] Implement agent invocation via SDK
- [ ] Add session hierarchy navigation

### Phase 3: Rule Loading System
- [ ] Create `AGENTS.md` with framework integration docs
- [ ] Implement on-demand rule loading pattern
- [ ] Add rule caching and validation
- [ ] Test lazy loading with large rule sets

### Phase 4: Monitoring & Quality
- [ ] Implement event streaming for progress monitoring
- [ ] Add quality gate enforcement
- [ ] Create review feedback loop
- [ ] Implement automatic session cleanup

### Phase 5: Polish
- [ ] Add comprehensive error handling
- [ ] Implement retry strategies for failed agents
- [ ] Create developer documentation
- [ ] Establish performance benchmarks

## Quality Metrics

Agents are evaluated against:

```typescript
interface FrameworkQualityMetrics {
  specificationCompliance: number;      // % of outputs matching spec
  taskCompletionRate: number;           // % of tasks completed successfully
  codeQualityScore: number;             // Clean Code metrics from reflector
  architecturalCompliance: number;      // Layer boundary violations
  reviewPassRate: number;               // % passing reflector on first try
  sessionNavigationSuccess: number;     // % of successful session returns
  ruleLoadingEfficiency: number;        // Cache hit rate for rule loading
}
```

## Conclusion

This framework provides a practical extension to OpenCode's built-in capabilities:

1. **Leverages OpenCode SDK** - Uses official APIs for orchestration
2. **Extends Agent System** - Builds on primary/subagent architecture
3. **Integrates with AGENTS.md** - Follows established patterns
4. **Addresses Known Issues** - Explicit session management for #6491
5. **Enables Complex Workflows** - Planner → Executor → Reflector pipeline

The framework is designed to evolve with OpenCode releases, maintaining compatibility while enabling sophisticated multi-agent workflows.
