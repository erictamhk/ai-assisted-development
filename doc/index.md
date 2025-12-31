# Software Engineering Concepts and Methodologies

A comprehensive documentation of modern software engineering concepts, methodologies, and practices relevant to AI-assisted development.

## Documentation Index

### Requirements & Analysis
- [Requirements and Specification](requirements-and-specification.md) - Understanding software requirements and specification documents
- [Specification Driven Development](specification-driven-development.md) - Development approach centered around formal specifications
- [Problem Frames Approach](problem-frames-approach.md) - Michael Jackson's framework for requirements analysis

### Domain-Driven Design
- [Domain-Driven Design](domain-driven-design-megred.md) - Strategic and tactical DDD patterns
- [Domain Event Storming](domain-event-storming.md) - Collaborative event modeling technique
- [Domain Event Storytelling](domain-event-storytelling.md) - Narrative approach to domain events

### Architecture & Code Quality
- [Layered Architecture](layered-architecture.md) - Ports and Adapters / Clean Architecture principles
- [Clean Architecture Refactoring Journey](clean-architecture-refactoring-journey.md) - 14-step proven refactoring journey from monolith to Clean Architecture
- [Clean Code](clean-code.md) - Writing maintainable and expressive code (enhanced with AI coding anti-patterns)
- [Vertical Slice Architecture](vertical-slice-architecture-megred.md) - Feature-based project organization

### Specification & Testing
- [Specification by Example](specification-by-example.md) - Requirements through concrete examples
- [Design by Contract](design-by-contract.md) - Formal verification through contracts
- [Executable Specifications](executable-specifications.md) - Living documentation as tests
- [Living Documentation](living-documentation.md) - Self-updating project documentation

### Testing Methodologies
- [Test-Driven Development](test-driven-development.md) - TDD principles and practices
- [Behavior-Driven Development](behavior-driven-development.md) - BDD with Gherkin and Cucumber

### Advanced Patterns
- [Event Sourcing](event-sourcing.md) - Pattern for storing state changes
- [CQRS](cqrs.md) - Command Query Responsibility Segregation
- [Design Patterns](design-patterns.md) - Gang of Four patterns catalog (enhanced with Clean Architecture patterns)
- [Pattern Language](pattern-language.md) - Christopher Alexander's pattern language concept

### Collaborative Planning
- [Impact Mapping](impact-mapping.md) - Gojko Adzic's strategic planning technique
- [Example Mapping](example-mapping.md) - BDD example discovery technique

### Coding Standards
- [Coding Conventions](coding-conventions.md) - Universal naming, organization, and style guidelines (enhanced with AI-specific conventions)

### AI-Assisted Development
- [AI Coding Patterns](ai-coding-patterns.md) - Pattern language for AI-assisted development with anti-patterns and specification-driven development
- [AGENTS.md and CLI AI Agent Tools](agents-md-cli-ai-agent-tools.md) - The AGENTS.md standard and supporting tools
- [Prompt Engineering for AI Agents](prompt-engineering-ai-agents.md) - Techniques for effective agent instructions
- [AI Agent Limitations](ai-agent-limitations.md) - Why agents skip instructions and fundamental constraints
- [On-Demand Rule Loading](on-demand-rule-loading.md) - Techniques to load rules only when needed

## Quick Links

### For AI Coding Assistants
All documentation is optimized for AI-assisted development, including:
- Practical implementation guidance
- Code examples and patterns
- Tool recommendations
- Best practices for each methodology
- Anti-patterns to avoid

### How to Use This Documentation
1. **New Projects**: Start with Requirements and Specification, then explore Architecture patterns
2. **Refactoring**: Reference Layered Architecture and Clean Code for guidance
3. **Team Collaboration**: Use Impact Mapping and Example Mapping for planning
4. **Quality Assurance**: Implement TDD, BDD, and Executable Specifications
5. **AI-Assisted Development**: Use AGENTS.md and Prompt Engineering for effective agent collaboration

### AI Agent Development
When working with AI agents:
1. Read relevant AGENTS.md files before coding
2. Reference pattern documents for implementation guidance
3. Use the Domain-Driven Design document for domain modeling
4. Follow Coding Conventions for consistent code
5. Use On-Demand Rule Loading to manage complex rule sets

## Documentation Structure

| Category | Topics Covered | Best For |
|----------|---------------|----------|
| **Requirements** | Problem Frames, Spec-by-Example | Understanding what to build |
| **Architecture** | Layered Architecture, Vertical Slices | Structuring the system |
| **DDD** | Strategic/Tactical Patterns, Entity/VO/Aggregate | Complex domain modeling |
| **Testing** | TDD, BDD, Executable Specs | Quality assurance |
| **Patterns** | Design Patterns, Pattern Languages | Reusable solutions |
| **Collaboration** | Impact Mapping, Example Mapping | Team alignment |
| **AI Development** | AGENTS.md, Prompt Engineering, On-Demand Loading | Effective AI collaboration |

## Contributing

These documents are maintained as living documentation. Each file includes:
- Theoretical foundations
- Practical implementation guidance
- Code examples
- Common pitfalls and best practices
- References to authoritative sources

### Adding New Documentation
When adding new documentation files:
1. Follow the established structure and format
2. Include TypeScript/JavaScript code examples
3. Add AI-assisted development guidance sections
4. Document best practices and anti-patterns
5. Include references to authoritative sources

### Maintenance Guidelines
- Review and update documentation with each major release
- Verify code examples compile and work correctly
- Keep references and links current
- Incorporate feedback from AI-assisted development usage
