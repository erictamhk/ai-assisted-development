# AGENTS.md

This folder contains reference documentation including guidelines, constitutions, and structural documentation for AI-assisted development.

## Files

- **ai_agent_development_guidelines.md** - Comprehensive guidelines for AI agent development
- **CONSTITUTION.md** - Foundational constitution for AI agent behavior and ethics
- **FOLDER_STRUCTURE.md** - Documentation of the repository folder structure
- **ai-coding-exercise-analysis.md** - Analysis of the AI Coding Exercise template repository for AI-assisted DDD development

### opencode

An open-source AI coding agent with provider-agnostic architecture, built-in LSP support, and TUI-focused design.

**Location**: `ref/opencode/`

**Key Features**:
- Provider-agnostic architecture (Claude, OpenAI, Google, local models)
- Built-in LSP support for multiple languages
- TUI-first design (terminal user interface)
- Client/server architecture for remote control
- Agent system with build, plan, and subagent modes
- MCP (Model Context Protocol) integration
- Skills system for extending agent capabilities

**Agent Modes**:
- **build** - Default agent with full access for development work
- **plan** - Read-only agent for analysis and exploration
- **subagent** - General-purpose agent for complex multi-step tasks (@general)

**Content Engineering**:
- Skills system (`SKILL.md`) for defining reusable agent capabilities
- Agent prompts stored as text files with context-aware templates
- Prompt compaction, exploration, summary, and title generation
- System prompt header for provider-specific instructions
- Agent generation API for dynamic agent configuration

**Documentation Structure**:
- `.opencode/` - Configuration directory
  - `agent/` - Agent prompts and definitions
  - `command/` - Custom commands
  - `skill/` - Skills for extending agent behavior
  - `opencode.jsonc` - Main configuration file
- `packages/opencode/src/agent/` - Core agent implementation
- `packages/docs/` - User-facing documentation
- `packages/sdk/` - JavaScript SDK for external integrations

**Quick Start**:
```bash
# Run against current directory
bun dev

# Run against specific directory
bun dev <directory>

# Build standalone executable
./packages/opencode/script/build.ts --single
```

See `ref/opencode-analysis.md` for detailed analysis including:
- Content Engineering patterns
- Agent system architecture
- Tool calling and parallel execution
- SDK and plugin architecture
- MCP integration patterns

## Referenced Repositories

### ai-coding-exercise

A template project demonstrating AI-assisted Domain-Driven Design (DDD) development with Clean Architecture, CQRS, and Event Sourcing patterns.

**Location**: `ref/ai-coding-exercise/`

**Key Features**:
- Sub-agent Workflow System for different development tasks
- Event Sourcing with ezddd framework
- Profile-based testing (test-inmemory, test-outbox)
- Comprehensive code review checklists
- Task-driven development with JSON task files

**Documentation Structure**:
- `.ai/` - AI Coding framework prompts, workflows, and guides
- `.dev/` - Project-specific content including ADRs and lessons
- `frontend/` - React TypeScript frontend application

**Quick Start**:
```bash
# Execute a task
execute task-create-product.json

# Run tests
mvn test -q
```

See `ref/ai-coding-exercise-analysis.md` for detailed analysis including:
- AI Coding Pattern Language (9.5/10)
- Specification-Driven Development (9.5/10)
- DDD Implementation (9.0/10)
- Clean Architecture Patterns (9.5/10)
- Sub-Agent System Architecture (9.0/10)
- Testing Strategy (9.0/10)
- Code Review Framework (9.5/10)
- **Overall AI-Assisted Development Score: 92/100**

### refactor-to-ca

A real-world refactoring project demonstrating a step-by-step transition from a monolithic codebase to Clean Architecture with Domain-Driven Design principles.

**Location**: `ref/refactor-to-ca/`

**Branch**: `step14` (tracks origin/step14)

**Key Features**:
- 14-step incremental refactoring approach
- TDD-driven implementation at each step
- Full DDD tactical design patterns (Aggregates, Entities, Value Objects)
- Clean Architecture with clear layer separation (entity, usecase, adapter, io)
- Dual interface support (Console and Web controllers)
- Persistence via JPA with in-memory repository option
- Bridge pattern for repository abstraction

**Refactoring Steps Overview**:
| Step | Focus Area | Key Changes |
|------|------------|-------------|
| 1 | Initial Setup | JDK 21, JUnit5, test structure |
| 2 | DDD Foundation | Create 4 layers, Aggregate Root, Value Objects |
| 3 | Entity Extraction | Extract command/query classes from monolithic code |
| 4 | Use Case Formation | Implement Command and Query use cases |
| 5 | Controller Formation | Extract and inject use case dependencies |
| 6 | DI Framework | Introduce SpringBoot as DI container |
| 7 | Web Controllers | Create REST controllers for each feature |
| 8 | Persistence Layer | Add PO, Mappers, Repository pattern |
| 9 | JPA Integration | Switch to relational database via Spring Data JPA |
| 10-11 | Feature Expansion | Add Deadline, Today, Custom ID features |
| 12-13 | Console Refactor | Improve console controller, add View by Deadline |
| 14 | Controller Refactor | Extract to individual controllers with executor pattern |

**Documentation Structure**:
- `src/main/java/tw/teddysoft/tasks/` - Main source code
  - `entity/` - Domain model (Aggregate Root, Entities, Value Objects)
  - `usecase/` - Business logic (Command and Query use cases)
  - `adapter/` - Controllers and Presenters (in/out boundaries)
  - `io/` - Entry points (Spring Boot, Console apps)
- `src/test/java/tw/teddysoft/tasks/` - Comprehensive test suite

See `ref/refactor-to-ca-analysis.md` for detailed step-by-step analysis including:
- Complete commit history with refactoring patterns
- Architecture evolution at each step
- Design pattern applications (Strategy, Bridge, Factory, etc.)
- Dependency injection progression
- Testing strategy evolution
- **Overall Refactoring Approach Score: 95/100**
