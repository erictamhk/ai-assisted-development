# AI Development Guidelines

This folder contains comprehensive knowledge about AI-assisted software development, including guidelines, patterns, best practices, and limitations when working with AI coding assistants.

## Contents

### Getting Started

- **[agents-md-standard.md](agents-md-standard.md)**: AGENTS.md standard for documenting AI agent constraints, workflows, and project-specific guidelines
- **[ai-agent-development-guidelines.md](ai-agent-development-guidelines.md)**: Comprehensive guidelines for developing with AI coding assistants, including workflow patterns and best practices

### Core Practices

- **[ai-coding-patterns.md](ai-coding-patterns.md)**: Pattern language for AI-assisted development including anti-patterns to avoid and best practices to follow
- **[ai-limitations.md](ai-limitations.md)**: Understanding AI agent limitations, common failure modes, and strategies to work around them
- **[context-engineering.md](context-engineering.md)**: Managing AI context effectively with the Context-Aware Action Protocol (CAAP) for optimal AI assistance
- **[on-demand-loading.md](on-demand-loading.md)**: Techniques for loading rules and context dynamically based on task requirements

### Communication

- **[prompt-engineering.md](prompt-engineering.md)**: Effective prompt techniques for AI coding assistants to get better results

## Quick Reference

### AI Development Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI ASSISTED DEVELOPMENT                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. CONTEXT LOADING                                             │
│      ↓ Load relevant rules, patterns, and specifications        │
│   2. TASK DEFINITION                                             │
│      ↓ Define clear requirements and constraints                 │
│   3. AI GENERATION                                               │
│      ↓ AI generates code based on context and requirements       │
│   4. REVIEW & VALIDATION                                         │
│      ↓ Validate AI output against specifications                 │
│   5. ITERATION                                                   │
│      ↓ Refine and improve based on feedback                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Principles

| Principle | Description |
|-----------|-------------|
| **Clear Specifications** | Provide detailed, unambiguous requirements |
| **Incremental Development** | Build in small, verifiable steps |
| **Human Review** | Always validate AI-generated code |
| **Pattern Consistency** | Follow established patterns and conventions |
| **Context Management** | Provide relevant context, avoid overload |

### Common AI Anti-Patterns

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| **Over-Reliance** | Accepting all AI suggestions | Always review and validate |
| **Vague Prompts** | Unclear instructions | Be specific and detailed |
| **No Validation** | Skipping tests | Always write tests |
| **Missing Context** | Not providing enough info | Include relevant patterns |
| **Large Context** | Overwhelming the AI | Use focused, incremental prompts |

## AI Interaction Patterns

### Effective Prompt Structure

```markdown
## Context
[Describe the system and purpose]

## Requirements
1. [Specific requirement]
2. [Specific requirement]

## Constraints
- [Language/framework]
- [Architecture style]
- [Patterns to follow]

## Input
[Code or data to work with]

## Output
[Expected format and structure]
```

### Iterative Refinement

1. **Start Simple**: Begin with basic structure
2. **Add Details**: Incrementally add complexity
3. **Validate Early**: Test as you go
4. **Refine Based on Feedback**: Improve based on review

## Best Practices

### Before AI Interaction

- [ ] Define clear requirements
- [ ] Identify relevant patterns
- [ ] Gather necessary context
- [ ] Prepare test cases

### During AI Interaction

- [ ] Provide specific instructions
- [ ] Include examples
- [ ] Break down complex tasks
- [ ] Ask for explanations

### After AI Interaction

- [ ] Review generated code
- [ ] Run tests
- [ ] Validate against requirements
- [ ] Document changes

## Related Topics

- **Architecture**: [03-architecture/](../03-architecture/) - Clean Architecture patterns
- **DDD**: [02-ddd/](../02-ddd/) - Domain-Driven Design patterns
- **Clean Code**: [04-coding-style/clean-code.md](../04-coding-style/clean-code.md) - Code quality
- **Testing**: [05-testing/](../05-testing/) - Testing strategies

## References

- "Designing AI-Assisted Development Workflows"
- OpenAI API Documentation
- Anthropic API Documentation
- Pattern Language for AI Development
