# AGENTS.md and CLI AI Agent Tools

## Core Concept

AGENTS.md is an open format for providing persistent project context and instructions to AI coding agents. Think of it as a "README for agents" - a dedicated file that gives AI assistants the context, conventions, and guidelines they need to work effectively on your codebase without requiring repetitive explanations in every conversation.

The fundamental insight behind AGENTS.md is that large language models are stateless functions. They have no inherent knowledge of your project structure, coding standards, or team conventions. AGENTS.md solves this by providing a single, predictable location where agents can automatically load project-specific instructions at the start of every session.

Unlike traditional documentation aimed at human developers, AGENTS.md focuses on information that helps AI agents understand how to work within your project: build commands, test procedures, code style preferences, architectural patterns, and operational guidelines.

---

## The AGENTS.md Standard

### What AGENTS.md Is

AGENTS.md is a simple Markdown file placed at the root of a repository (or in subdirectories for monorepos) that AI coding agents automatically discover and incorporate into their context. The format is intentionally lightweight - just standard Markdown with whatever sections make sense for your project.

The standard emerged from collaborative efforts across the AI software development ecosystem, including OpenAI Codex, Google (Jules), Cursor, Factory, and others. It is now stewarded by the Agentic AI Foundation under the Linux Foundation, ensuring it remains vendor-neutral and community-governed.

What makes AGENTS.md powerful is its cross-platform compatibility. A single AGENTS.md file works across multiple AI coding tools, eliminating the need to maintain separate configuration files for each agent. This standardization reduces maintenance overhead while ensuring consistent agent behavior regardless of which tool your team uses.

### Why AGENTS.md Matters

Without AGENTS.md, each conversation with an AI coding agent requires re-establishing project context. Developers repeatedly explain tech stacks, project structures, coding conventions, and operational procedures. This repetition wastes time and creates inconsistent results as context gets lost or miscommunicated.

AGENTS.md automates this context establishment. By reading the file once, agents gain persistent knowledge about your project that persists across sessions. This leads to more accurate responses, fewer clarification questions, and code that better matches your existing standards from the first interaction.

The file also serves as living documentation that evolves with your project. As coding standards change or new conventions emerge, updating AGENTS.md updates all agents simultaneously. This creates a single source of truth for agent behavior rather than fragmented, potentially contradictory instructions scattered across conversations.

---

## AGENTS.md Format Specification

### File Location and Discovery

AGENTS.md files follow a hierarchical discovery pattern. The primary file resides at the repository root, but subdirectories can contain their own AGENTS.md files with more specific instructions. Agents automatically discover the nearest file in the directory tree, with closer files taking precedence over ones higher in the hierarchy.

For large monorepos, this nested approach allows different teams or modules to have tailored instructions while still inheriting from root-level conventions. The main OpenAI repository, for example, contains 88 AGENTS.md files to provide specialized guidance for different parts of their codebase.

The discovery process walks up from the current working directory to the repository root, checking each directory for AGENTS.md or AGENTS.override.md files. When multiple files exist, they are concatenated with later files overriding earlier ones, allowing for progressive refinement of instructions.

### Recommended Sections

While AGENTS.md has no required fields, effective files typically include several core sections. Project Overview provides the agent with a high-level understanding of what the project does, its main technologies, and its architecture. This section answers the fundamental question of "what is this project?"

Setup Commands document the commands needed to install dependencies, start development servers, and run tests. Including these commands prevents agents from guessing incorrect commands or getting stuck on setup issues. Code Style Guidelines specify your team's conventions: language preferences, formatting rules, naming patterns, and architectural principles.

Testing Instructions describe how to run tests, what test frameworks are used, and any testing conventions. This ensures agents write tests that match your existing patterns and can verify their changes work correctly.

### Example AGENTS.md File

```markdown
# Project Name
A brief description of what this project does and its main purpose.

## Tech Stack
- Language: TypeScript 5.x
- Framework: Node.js 20+
- Package Manager: pnpm
- Testing: Vitest
- Linting: ESLint with TypeScript support

## Project Structure
- `src/` - Main source code
- `tests/` - Test files (mirrors src structure)
- `docs/` - Documentation
- `scripts/` - Build and deployment scripts

## Code Standards
- Use TypeScript strict mode
- Prefer functional patterns over classes where possible
- Use absolute imports with `@/` alias for src/ imports
- Write unit tests for all new functions
- Follow the existing file organization patterns

## Development Commands
- Install dependencies: `pnpm install`
- Run development server: `pnpm dev`
- Run tests: `pnpm test`
- Run linter: `pnpm lint`
- Type check: `pnpm typecheck`

## Testing Requirements
- Write tests alongside implementation (test-first encouraged)
- Aim for 80% code coverage on new code
- Use descriptive test names following pattern: should_[expected behavior]_when_[condition]
- Integration tests go in `tests/integration/`

## Common Patterns
- Error handling: Use Result type pattern for fallible operations
- Configuration: Environment variables with .env.example
- Logging: Use structured logging with context
- APIs: RESTful endpoints with OpenAPI documentation

## Security Guidelines
- Never commit secrets or API keys
- Validate all inputs on the server side
- Use parameterized queries for database operations
- Follow OWASP security practices
```

---

## CLI AI Agent Tools Supporting AGENTS.md

### OpenAI Codex

Codex CLI is OpenAI's command-line AI coding agent that reads AGENTS.md files with sophisticated discovery logic. When Codex starts, it builds an instruction chain by checking multiple locations in a specific precedence order.

The global scope check occurs first in the Codex home directory (default `~/.codex`). Codex reads AGENTS.override.md if it exists, otherwise AGENTS.md. Only the first non-empty file at this level is used, ensuring global defaults don't conflict.

Project scope follows, with Codex walking from the repository root to the current working directory. In each directory, it checks for AGENTS.override.md, then AGENTS.md, then any fallback filenames configured in the settings. Codex includes at most one file per directory, concatenating all found files from root down.

Codex stops adding files once the combined size reaches `project_doc_max_bytes` (32 KiB by default). This prevents context windows from overflowing while still providing comprehensive guidance. Raise this limit or split instructions across nested directories when you need more space.

Configuration fallback names allow Codex to recognize existing documentation files as instruction sources. Add names like `TEAM_GUIDE.md` or `.agents.md` to `project_doc_fallback_filenames` in your configuration, and Codex treats them like AGENTS.md files.

### OpenCode

OpenCode reads AGENTS.md files from both project and global locations. Project-specific rules apply when working in a directory or its subdirectories, while global rules in `~/.config/opencode/AGENTS.md` apply across all sessions.

OpenCode can also specify custom instruction files in `opencode.json` using the `instructions` field. This allows referencing multiple files that get combined with AGENTS.md:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": ["CONTRIBUTING.md", "docs/guidelines.md", ".cursor/rules/*.md"]
}
```

The `/init` command in OpenCode can scan your project and generate an initial AGENTS.md file, helping you get started quickly. This generated file includes project structure, detected technologies, and common conventions.

### Gemini CLI

Gemini CLI uses GEMINI.md files with a similar hierarchical system. It loads context files from multiple locations, concatenates their contents, and sends them to the model with every prompt. The loading order follows a clear precedence:

Global context resides at `~/.gemini/GEMINI.md` and provides default instructions for all projects. Project root and ancestor context files are discovered by walking up from the current directory, with closer files taking precedence.

Configuration through `settings.json` allows customizing the context filename:

```json
{
  "contextFileName": "AGENTS.md"
}
```

This flexibility means Gemini CLI can read AGENTS.md files directly, maintaining compatibility with the broader ecosystem while using its own configuration approach.

### Aider

Aider reads convention files specified in `.aider.conf.yml`. To use AGENTS.md:

```yaml
read: AGENTS.md
```

Aider's convention system allows defining additional files that provide context and instructions. Combine AGENTS.md with other convention files for comprehensive project guidance.

### Zed Editor

Zed's AI features support AGENTS.md files through their rules system. Place AGENTS.md in your project root, and Zed's AI assistant automatically incorporates it into conversations about your code.

### Warp Terminal

Warp's AI feature supports project-scoped rules that can reference AGENTS.md. Configure this in Warp's settings to provide context for AI-powered commands.

---

## Configuration Examples

### Codex Configuration

```toml
# ~/.codex/config.toml

# Global AGENTS.md location
codex_home = "~/.codex"

# Fallback filenames to recognize
project_doc_fallback_filenames = ["TEAM_GUIDE.md", ".agents.md"]

# Maximum combined size of instruction files
project_doc_max_bytes = 65536

# Enable session logging for debugging
session_logging = true
```

### OpenCode Configuration

```json
// ~/.config/opencode/opencode.json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "AGENTS.md",
    "CONTRIBUTING.md",
    "docs/style-guide.md",
    "packages/*/AGENTS.md"
  ],
  "rules": {
    "maxContextSize": 128000,
    "enableAutoScan": true
  }
}
```

### Gemini CLI Configuration

```json
// ~/.gemini/settings.json
{
  "contextFileName": "AGENTS.md",
  "model": "gemini-2.5-pro",
  "maxOutputTokens": 8192,
  "temperature": 0.2,
  "tools": {
    "enabled": true,
    "allowedTools": ["bash", "read", "grep", "edit"]
  }
}
```

---

## Best Practices for AGENTS.md

### Keep It Concise Yet Complete

AGENTS.md should contain everything an agent needs to work effectively, but avoid unnecessary verbosity. Agents have limited context windows, so prioritize the most important information. Focus on conventions that, if violated, would require significant rework or would produce code that doesn't match your standards.

Include the "why" behind conventions when it helps understanding. Saying "use functional patterns because our domain is inherently stateless and mutation causes subtle bugs" gives agents better guidance than just stating the rule.

### Update It Regularly

Treat AGENTS.md as living documentation that evolves with your project. When you establish a new convention, add it to AGENTS.md. When existing patterns change, update the file. This keeps agents aligned with current practices rather than outdated guidance.

Consider adding a "Last Updated" note or version information so developers know whether the file reflects current practice. Some teams link AGENTS.md to their changelog or commit history for traceability.

### Use Nested Files for Large Projects

For monorepos or large projects, create AGENTS.md files in individual packages or subdirectories. This keeps instructions focused and prevents root-level files from becoming unwieldy. The closest AGENTS.md to the edited file takes precedence, allowing specialized guidance where needed.

For example, a monorepo might have:

```
/AGENTS.md              # Project-wide conventions
/packages/
  backend/
    AGENTS.md           # Backend-specific rules
    src/
  frontend/
    AGENTS.md           # Frontend-specific rules
    src/
```

### Document Build and Test Commands Explicitly

Include the exact commands for common operations rather than expecting agents to discover them. Even if your project uses standard tools like `npm test`, specifying the command prevents agents from trying alternative invocations that might fail or produce different results.

For projects with complex setups, document prerequisite steps and expected outputs. This helps agents diagnose problems and understand whether operations succeeded.

### Include Security and Operational Guidelines

If your project has security requirements, operational constraints, or deployment procedures, document them in AGENTS.md. This ensures agents don't accidentally violate constraints or suggest approaches that wouldn't work in your environment.

For example, if production deployments require specific approval processes or if certain operations are restricted in production environments, include these constraints so agents can work within them.

---

## Anti-Patterns to Avoid

### Stale Documentation

Outdated AGENTS.md files are worse than no file at all because they provide wrong guidance. Set up reminders to review the file periodically, especially after major architectural changes or when onboarding new team members who might have different assumptions.

### Overly Vague Instructions

Instructions like "write clean code" or "follow best practices" provide no actionable guidance. Be specific about what "clean" means in your context: specific patterns, exact formatting rules, concrete examples of what you consider good code.

### Including Human-Only Documentation

AGENTS.md should focus on information relevant to AI agents. Don't duplicate your README or contribution guide. Link to those documents if agents need to reference them, but don't copy content that doesn't help agents understand how to work with your code.

### Ignoring Size Limits

Context windows are finite. If your AGENTS.md approaches the default 32 KiB limit, consider splitting it into focused files or moving detailed documentation elsewhere. Include the most critical information in AGENTS.md and reference additional files for depth.

---

## AGENTS.md in the AI Agent Ecosystem

### Growing Tool Support

The AGENTS.md standard has been adopted by a growing list of AI coding tools. Support spans CLI agents (Codex, Gemini CLI, OpenCode, Aider), IDE extensions (VS Code, Zed, Cursor), and commercial products (Devin, Copilot Coding Agent). This broad adoption means an investment in AGENTS.md works across your entire toolchain.

The standard continues to evolve under the Agentic AI Foundation's stewardship, with input from multiple organizations building AI development tools. This governance model ensures the standard serves the community rather than any single vendor.

### Relation to Other Formats

While AGENTS.md is the most widely adopted format, some tools use their own variants. Claude Code uses CLAUDE.md with similar functionality. These formats are interoperable - a CLAUDE.md file works in most AGENTS.md-compatible tools, and vice versa.

Some projects maintain both files to ensure maximum compatibility. The files can be identical or contain tool-specific additions. The AGENTS.md website provides guidance on migrating between formats when needed.

### Future Directions

The AGENTS.md standard continues to evolve with input from the AI agent community. Future enhancements may include structured metadata for automatic tool selection, version schemas for compatibility checking, and integration with documentation generation systems.

The Agentic AI Foundation welcomes contributions from projects using AGENTS.md. The governance model ensures that changes reflect real-world usage rather than theoretical preferences.

---

## References and Further Reading

1. "AGENTS.md Official Website" - https://agents.md
2. "OpenAI Codex AGENTS.md Documentation" - https://developers.openai.com/codex/guides/agents-md
3. "OpenCode Rules Documentation" - https://opencode.ai/docs/rules/
4. "Gemini CLI Configuration" - https://google-gemini.github.io/gemini-cli/docs/get-started/configuration.html
5. "Claude Code CLAUDE.md Guide" - https://claude.com/blog/using-claude-md-files
6. "Agentic AI Foundation" - https://aaif.io
