# AGENTS.md

This folder contains reference documentation including guidelines, constitutions, and structural documentation for AI-assisted development.

## Files

- **ai_agent_development_guidelines.md** - Comprehensive guidelines for AI agent development
- **CONSTITUTION.md** - Foundational constitution for AI agent behavior and ethics
- **FOLDER_STRUCTURE.md** - Documentation of the repository folder structure
- **ai-coding-exercise-analysis.md** - Analysis of the AI Coding Exercise template repository for AI-assisted DDD development

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
