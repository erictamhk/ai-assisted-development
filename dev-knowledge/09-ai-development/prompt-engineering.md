# Prompt Engineering for AI Agents

## Core Concept

Prompt engineering is the discipline of crafting effective instructions that reliably guide AI systems toward desired behaviors. For AI coding agents, prompt engineering becomes especially critical because agents must understand complex technical contexts, follow project conventions, and produce code that integrates seamlessly with existing systems.

Unlike general-purpose chat prompts, agent-focused prompts must account for the unique challenges of software development: understanding project-specific conventions, working within architectural constraints, maintaining code quality standards, and producing outputs that pass rigorous testing and review processes.

The goal of prompt engineering for agents is not merely to get any response, but to get the right response consistently. This requires understanding how language models process instructions, what factors influence their outputs, and how to structure prompts for maximum clarity and effectiveness.

Effective prompts for AI agents share common characteristics: they provide clear roles and context, specify constraints explicitly, include relevant examples, and structure information for easy processing. Mastery of these techniques dramatically improves agent reliability and output quality.

---

## The Foundation: System Prompts

### Understanding System Prompts

System prompts form the foundation of agent behavior. They establish the agent's role, capabilities, limitations, and operating principles. Unlike user prompts that change with each conversation, system prompts persist across sessions, providing consistent behavioral guidance.

For AI coding agents, system prompts typically establish several key elements. The role definition tells the agent what type of assistant it is - a coding expert, a code reviewer, a documentation specialist. This role shapes the language, depth, and approach the agent uses.

Capability boundaries define what the agent can and cannot do. For coding agents, this might include limits on modifying production systems, requirements for test coverage, or restrictions on certain types of changes. These boundaries prevent agents from taking inappropriate actions while focusing their efforts productively.

Operating principles establish how the agent should approach problems. This includes preferences like preferring existing patterns over novel solutions, seeking clarification when uncertain, or prioritizing code clarity over clever optimizations.

### System Prompt Structure

Effective system prompts follow a consistent structure that agents can parse and apply reliably:

```markdown
# Role and Purpose
You are an expert software engineer specializing in [domain]. Your role is to help developers write high-quality, maintainable code that follows best practices and integrates well with existing systems.

# Core Capabilities
- Writing clean, tested code in [languages]
- Understanding and extending existing codebases
- Writing comprehensive tests and documentation
- Following security best practices
- Explaining technical concepts clearly

# Operating Principles
1. Prioritize code clarity and maintainability over clever optimizations
2. Follow existing patterns and conventions in the codebase
3. Write tests before or alongside implementation code
4. Ask for clarification when requirements are ambiguous
5. Explain your reasoning when making architectural decisions
6. Never modify production systems without explicit approval

# Constraints
- Always write type-safe code with proper TypeScript/JavaScript patterns
- Include unit tests for all new functions (aim for 80%+ coverage)
- Use proper error handling with descriptive error messages
- Follow the project's existing code style and patterns
- Do not commit secrets or sensitive information
- Do not make changes to infrastructure without team review

# Project Context
[Reference to AGENTS.md or project-specific documentation]
```

### Dynamic System Prompts

Modern agent frameworks support dynamic system prompts that incorporate context at runtime. This allows prompts to include project-specific information without hardcoding it into base instructions:

```typescript
import { Agent, RunContext } from 'pydantic_ai';

const agent = Agent(
  'openai:gpt-4o',
  deps_type=ProjectContext,
  system_prompt="Help the user with their coding tasks.",
);

@agent.system_prompt
def add_project_context(ctx: RunContext[ProjectContext]) -> str:
    return f"""You are working on {ctx.deps.project_name}.
    
    Tech stack: {ctx.deps.tech_stack}
    Code style: {ctx.deps.code_style_guide}
    Project structure: {ctx.deps.structure_summary}
    
    Always follow the project's conventions."""
```

---

## Core Prompt Engineering Techniques

### Role and Persona Definition

Defining a clear role helps agents adopt appropriate expertise and communication styles. Effective role definitions are specific rather than generic, grounding the agent in domain expertise:

**Weak**: "You are a helpful coding assistant."

**Strong**: "You are a senior backend engineer at a fintech company, specializing in distributed systems and payment processing. You have 15 years of experience building high-reliability financial services and understand the regulatory requirements for handling money securely."

The strong version provides concrete expertise that shapes how the agent approaches problems, anticipates edge cases, and communicates with appropriate technical depth.

For AGENTS.md files, include role definitions that match your project's needs:

```markdown
# Role
You are an expert TypeScript developer familiar with the project's domain of real-time data processing. You understand streaming architectures, event-driven systems, and the specific requirements of low-latency data pipelines.

# Expertise Areas
- TypeScript/Node.js best practices
- Event-driven architecture patterns
- WebSocket and streaming protocols
- Time-series data handling
- Performance optimization for high-throughput systems
```

### Context and Background Provision

Agents need sufficient context to make good decisions. This includes project background, architectural decisions, and operational knowledge that experienced developers would have:

```markdown
# Project Background
This is a real-time analytics platform that processes clickstream data from web and mobile applications. The system ingests approximately 50,000 events per second, processes them through a streaming pipeline, and stores results in both a time-series database and a data warehouse.

# Architectural Decisions
- Event sourcing pattern for audit trail
- CQRS for separating read and write models
- Kafka for event streaming between services
- ClickHouse for time-series queries
- Redis for caching hot data

# Key Concepts
- Events represent user actions (clicks, page views, conversions)
- Sessions group related events by user activity within a time window
- Streams process events through a pipeline of transformations
```

### Explicit Constraints and Rules

Constraints guide agent behavior by defining boundaries. Effective constraints are specific, actionable, and include the reasoning behind them:

**Vague**: "Write secure code."

**Specific**: "For database queries, always use parameterized queries to prevent SQL injection. The project uses Prisma ORM with strongly-typed queries. Never concatenate strings into query templates - use Prisma's query builder or parameterized raw queries."

For AGENTS.md, structure constraints clearly:

```markdown
## Code Quality Constraints

### Type Safety
- Use TypeScript strict mode at all times
- Avoid `any` type; use `unknown` when type is truly uncertain
- Prefer union types over boolean parameters (e.g., `status: 'pending' | 'complete'` instead of `isComplete: boolean`)

### Error Handling
- Use Result types for fallible operations instead of exceptions
- Never swallow errors silently
- Log errors with sufficient context for debugging
- Propagate errors to appropriate handlers rather than suppressing them

### Testing
- Write tests before or alongside implementation
- Aim for 80%+ code coverage on new code
- Use descriptive test names: `should_[expected]_when_[condition]`
- Mock external dependencies at the appropriate level
```

### Example-Based Guidance

Examples are among the most effective prompt engineering tools because they show rather than tell. Include examples that demonstrate the patterns, style, and quality expected:

```markdown
## Code Examples

### Good Error Handling Pattern
```typescript
// Use Result type for fallible operations
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

async function processPayment(orderId: string): Promise<Result<PaymentConfirmation, PaymentError>> {
  try {
    const payment = await paymentGateway.charge(orderId);
    return { ok: true, value: payment };
  } catch (error) {
    if (error instanceof InsufficientFundsError) {
      return { ok: false, error: new PaymentError('Insufficient funds', 'INSUFFICIENT_FUNDS') };
    }
    return { ok: false, error: new PaymentError('Payment processing failed', 'UNKNOWN') };
  }
}
```

### Test Structure
```typescript
describe('processPayment', () => {
  it('should return confirmation when payment succeeds', async () => {
    const confirmation = await processPayment('order-123');
    expect(confirmation.ok).toBe(true);
    expect(confirmation.value.transactionId).toBeDefined();
  });

  it('should return error when insufficient funds', async () => {
    const result = await processPayment('order-123');
    expect(result.ok).toBe(false);
    expect(result.error.code).toBe('INSUFFICIENT_FUNDS');
  });
});
```
```

---

## Advanced Prompting Techniques

### Chain-of-Thought Prompting

Chain-of-Thought (CoT) prompting encourages agents to reason step-by-step before producing final answers. This improves performance on complex tasks by making reasoning explicit and verifiable:

```markdown
Before making changes to the codebase, think through the following:

1. What is the current behavior of the code?
2. What is the desired behavior?
3. What files need to change?
4. What is the minimal change needed to achieve this?
5. What tests need to pass?
6. What edge cases should be considered?

Think through each step explicitly before proceeding. Explain your reasoning so I can verify your understanding is correct.
```

### Adaptive Demonstrative CoT (ADCoT)

ADCoT combines few-shot examples with automated reasoning chain generation. Select diverse examples representing different aspects of the task, then use step-by-step prompting to generate reasoning chains:

```markdown
When implementing new features, follow this process:

1. Understand the requirement by tracing through existing code
2. Identify the minimal set of changes needed
3. Implement changes incrementally
4. Add or update tests for the new behavior
5. Verify all tests pass
6. Review the changes for consistency

Example for adding a new API endpoint:

Thought: I need to add a GET /users/:id endpoint to retrieve user details.
Thought: The existing pattern for endpoints is in controllers/users.ts.
Thought: I need to add a route, controller method, service call, and tests.
Thought: I'll follow the existing pattern for GET endpoints.
Thought: The route should be added to routes/users.ts.
Thought: The controller method should handle the happy path and errors.
Thought: The service method should fetch from the repository.
Thought: Tests should cover success and error cases.
```

### Interactive Contextual Prompting (ICP)

ICP creates conversational flows where prompts evolve based on agent responses. This is useful for complex tasks requiring iterative refinement:

```markdown
Let's work through this task iteratively:

1. First, describe the current code structure and how the feature should integrate
2. I'll provide feedback on your understanding
3. Propose the specific changes needed
4. I'll review and approve or request modifications
5. Implement the changes with tests
6. Verify everything works together

Start by describing your understanding of the task and the current code structure.
```

### Knowledge Enrichment Prompting (KEP)

KEP supplements agent knowledge with project-specific context. This is especially valuable for understanding proprietary systems or domain-specific patterns:

```markdown
## Project-Specific Knowledge

### Domain Model
- Users have roles: ADMIN, EDITOR, VIEWER
- Content goes through workflow: DRAFT → REVIEW → APPROVED → PUBLISHED
- Permissions are hierarchical: ADMIN can do everything, EDITOR can edit approved content with approval, VIEWER can only read

### State Machine
```
DRAFT → [submit_for_review] → REVIEW
REVIEW → [approve] → APPROVED
REVIEW → [reject] → DRAFT
APPROVED → [publish] → PUBLISHED
APPROVED → [unapprove] → REVIEW
```

### Common Patterns
- Use the State pattern for workflow states
- Use the Repository pattern for data access
- Use Domain Events for state transitions
```

---

## Prompt Engineering for AGENTS.md

### Structuring AGENTS.md Content

AGENTS.md files should be structured for easy scanning and parsing. Use clear sections with descriptive headings, keeping each section focused on a specific aspect of working with the project:

```markdown
# Project Name

Brief description of what this project does and its main purpose.

## Overview
- **Purpose**: [What problem does this solve?]
- **Tech Stack**: [Key technologies used]
- **Architecture**: [High-level architectural pattern]

## Development Setup

### Prerequisites
- Node.js 20+
- pnpm 8+
- Docker (for local services)

### Getting Started
```bash
pnpm install
pnpm dev
```

### Running Tests
```bash
pnpm test          # Run all tests
pnpm test:unit     # Unit tests only
pnpm test:integration  # Integration tests
```

## Code Standards

### Language Conventions
- TypeScript with strict mode
- Use functional patterns where possible
- Prefer immutability

### Code Organization
```
src/
  domain/     # Business logic
  application/ # Use cases
  infrastructure/ # External integrations
  interfaces/  # Public APIs
```

### Testing Requirements
- Unit tests for all business logic
- 80%+ coverage on new code
- Integration tests for API endpoints

## Security Guidelines

### Sensitive Data
- Never commit .env files
- Use environment variables for secrets
- Rotate API keys immediately if compromised

### Code Review Requirements
- Security review for auth changes
- Performance review for data operations
- All changes require at least one reviewer
```

### Progressive Disclosure

For large projects, use progressive disclosure with root-level AGENTS.md providing essential information and subdirectory files adding specifics:

```markdown
# Root AGENTS.md (Essential Info Only)

This project uses a monorepo structure with packages for different concerns.

## Quick Reference
- Package manager: pnpm
- Test command: pnx test
- Linting: npx eslint

## Package Structure
- `packages/api/` - Backend API
- `packages/web/` - Frontend application
- `packages/shared/` - Shared utilities

## See Also
- packages/api/AGENTS.md - Backend-specific rules
- packages/web/AGENTS.md - Frontend-specific rules
```

### Reference External Documentation

Rather than duplicating content, reference external files for depth:

```markdown
## For detailed guidance, see:

- Code style: docs/style-guide.md
- API documentation: docs/api/openapi.yaml
- Architecture: docs/architecture/overview.md
- Deployment: docs/ops/deployment.md
```

In the AGENTS.md itself, you can teach agents to load referenced files:

```markdown
## Development Guidelines

When working with TypeScript code:
- Follow the style guide: Read @docs/typescript-style.md
- Use the component patterns: Read @docs/react-components.md
- Handle API errors: Read @docs/error-handling.md

For testing:
- Testing patterns: Read @test/testing-guide.md
- Mock utilities: Read @test/mocks.md
```

---

## Techniques for Agent Instruction Following

### Verification and Validation

Include explicit instructions for self-verification:

```markdown
## Before Submitting Changes

1. Run the full test suite:
   ```bash
   pnpm test
   ```

2. Run type checking:
   ```bash
   pnx typecheck
   ```

3. Run linting:
   ```bash
   npx eslint src/ --fix
   ```

4. Verify the changes:
   - Code compiles without errors
   - All tests pass (no skipped tests)
   - TypeScript strict mode passes
   - No new lint warnings

5. Review the changes:
   - Changes are minimal and focused
   - Tests are added for new behavior
   - Documentation is updated if needed
```

### Handling Ambiguity

Teach agents to recognize and resolve ambiguity:

```markdown
## Handling Ambiguity

If instructions are unclear or requirements are incomplete:

1. **Identify the ambiguity**: State what is unclear
2. **Propose options**: Suggest reasonable interpretations
3. **Ask for clarification**: Request guidance on which approach to take
4. **Document assumptions**: Note any assumptions made

Do NOT guess when consequences could be significant. Ask for clarification.

Example:
> The requirement says "add search functionality" but doesn't specify:
> - Should it be server-side or client-side search?
> - Should it search all fields or specific ones?
> - What should happen with no results?
>
> I could implement client-side search on the name field as a starting point, or I could clarify the requirements first. Which do you prefer?
```

### Scope Management

Prevent agents from overreaching:

```markdown
## Scope Boundaries

### What This Project Includes
- Backend API service
- Frontend web application
- Shared utilities
- CI/CD pipelines

### What This Project Does NOT Include
- Mobile applications (separate repositories)
- Data analytics pipelines (separate repository)
- Infrastructure provisioning (separate repository)

### Change Scope
- Keep changes focused on the task at hand
- Do NOT refactor unrelated code
- Do NOT add features outside the scope
- If you discover a related issue, note it but focus on the primary task
```

---

## Common Anti-Patterns

### Overly Complex Instructions

Long, dense instructions reduce effectiveness. Agents have limited context windows, and complex instructions are harder to parse consistently. Keep instructions focused and actionable.

**Avoid**: Walls of text with multiple concerns mixed together.

**Prefer**: Short, focused sections with clear separation of concerns.

### Implicit Assumptions

Don't assume agents know your project's conventions. What seems obvious to humans often requires explicit statement for agents.

**Avoid**: "Follow the existing pattern" without explaining what that pattern is.

**Prefer**: "Follow the pattern in src/services/*.ts: constructors accept dependencies, methods are async, errors return Result types."

### Inconsistent Formatting

Inconsistent formatting makes instructions harder to parse. Use consistent heading styles, code block formatting, and list structures throughout.

### Outdated Information

Stale prompts misguide agents. Set up processes to review and update AGENTS.md and system prompts regularly, especially after architectural changes.

---

## Testing and Validating Prompts

### Prompt Versioning

Version your prompts and prompt templates to track changes and roll back when needed:

```markdown
# AGENTS.md
# Version: 2.3.1
# Last Updated: 2024-12-28
# Changelog:
# v2.3.1 - Added testing requirements section
# v2.3.0 - Updated tech stack to Node.js 20
# v2.2.0 - Added monorepo structure documentation
```

### A/B Testing Prompts

Compare agent behavior with different prompt variations. Track metrics like:
- Clarification requests made
- Time to complete tasks
- Code review feedback frequency
- Test coverage achieved

### Feedback Integration

Create channels for capturing prompt improvement opportunities. When agents make mistakes or ask unnecessary questions, document the gap in instructions.

---

## References and Further Reading

1. "Prompt Engineering Guide" - https://www.promptingguide.ai/
2. "OpenAI Prompting Best Practices" - https://help.openai.com/en/articles/6654000
3. "Anthropic Claude Code Best Practices" - https://www.anthropic.com/engineering/claude-code-best-practices
4. "Chain-of-Thought Prompting" - https://arxiv.org/abs/2201.11903
5. "Self-Consistency Improves Chain-of-Thought Reasoning" - https://arxiv.org/abs/2203.11171
6. "WizardLM: Empowering Large Language Models" - https://arxiv.org/abs/2304.12244
7. doc/agents-md-cli-ai-agent-tools.md - CLI AI Agent Tools
8. doc/on-demand-rule-loading.md - On-Demand Rule Loading
9. ref/engineering/context_engineering/CAAP.md - Context Engineering patterns

---

## Appendix: AGENTS.md Template Library

### Minimal AGENTS.md Template

```markdown
# Project Name

Brief description of what this project does.

## Tech Stack
- [Language/Framework 1]
- [Language/Framework 2]
- [Database/Storage]

## Quick Commands
- `npm install` - Install dependencies
- `npm test` - Run tests
- `npm run build` - Build for production

## Code Standards
- Follow existing code patterns
- Add tests for new functionality
- Run linting before committing

## Project Structure
```
src/
  ├── domain/     # Business logic
  ├── application/ # Use cases
  └── infrastructure/ # External systems
```
```

### Comprehensive AGENTS.md Template

```markdown
# [Project Name]

A [type] application built with [tech stack].

## Overview

- **Purpose**: [What problem does this solve?]
- **Architecture**: [Clean Architecture/DDD/Microservices]
- **Environment**: [Development/Production]

## Development Setup

### Prerequisites
- [Tool 1] version X+
- [Tool 2] version X+

### Getting Started
```bash
[command 1]
[command 2]
```

### Testing
```bash
[test command]       # All tests
[unit test command]  # Unit tests only
[integration command] # Integration tests
```

## Code Standards

### Language Conventions
- Use [patterns, e.g., "functional programming"]
- Prefer [principles, e.g., "immutability"]
- TypeScript/JavaScript: strict mode enabled

### Naming Conventions
- Variables/functions: [convention, e.g., camelCase]
- Classes/components: [convention, e.g., PascalCase]
- Constants: [convention, e.g., UPPER_SNAKE_CASE]

### File Organization
```
src/
  ├── [layer 1]/   # e.g., domain
  ├── [layer 2]/   # e.g., application
  └── [layer 3]/   # e.g., infrastructure
```

## Testing Requirements

- Minimum [X]% coverage on new code
- Unit tests for business logic
- Integration tests for API endpoints

## Security Guidelines

- Never commit secrets or .env files
- Use environment variables for sensitive data
- [Project-specific security rules]

## Code Review

- All changes require [number] reviewer(s)
- [Special review requirements, e.g., "Security review for auth changes"]
- Run [linting/testing] before requesting review

## Last Updated
- 2024-12-31
```

### Monorepo AGENTS.md Template

```markdown
# Monorepo Root AGENTS.md

This is a monorepo containing multiple packages.

## Quick Reference
- Package manager: [pnpm/npm/yarn]
- Test command: [command]
- Build command: [command]

## Package Structure
- `packages/api/` - Backend API service
- `packages/web/` - Frontend application
- `packages/shared/` - Shared utilities

## Common Commands
```bash
[install all]        # Install all dependencies
[test all]          # Run all tests
[build all]         # Build all packages
```

## See Also
- `/packages/api/AGENTS.md` - Backend-specific rules
- `/packages/web/AGENTS.md` - Frontend-specific rules
```

---

## Appendix: Prompt Engineering Checklist

When creating or reviewing prompts for AI agents, use this checklist:

### Prompt Structure
- [ ] Clear role definition
- [ ] Explicit context provided
- [ ] Constraints listed
- [ ] Examples included (where helpful)
- [ ] Output format specified
- [ ] Success criteria defined

### Context Management
- [ ] Relevant information included
- [ ] Irrelevant information excluded
- [ ] Priority given to critical rules
- [ ] On-demand loading configured for large projects
- [ ] Cross-references to documentation

### Clarity and Specificity
- [ ] Ambiguous terms defined
- [ ] Technical jargon explained
- [ ] "Why" behind rules included
- [ ] Concrete examples provided
- [ ] Edge cases addressed

### Testing and Validation
- [ ] Prompt tested with common scenarios
- [ ] Edge cases verified
- [ ] Feedback loop established
- [ ] Version tracking implemented
- [ ] Review process defined
