# OpenCode Analysis

## Overview

OpenCode is an open-source AI coding agent developed by SST, featuring a provider-agnostic architecture, built-in LSP support, and a TUI-first design philosophy. The project demonstrates sophisticated content engineering patterns and agent system architecture.

**Repository**: https://github.com/sst/opencode

**Key Differentiators**:
- 100% open source
- Provider-agnostic (Claude, OpenAI, Google, local models)
- Built-in LSP support
- TUI-focused with remote control capability
- Client/server architecture

## Architecture

### Monorepo Structure

```
opencode/
├── .opencode/           # Configuration directory
├── packages/
│   ├── opencode/        # Core business logic & server
│   ├── console/         # TUI frontend
│   ├── desktop/         # Desktop application
│   ├── docs/            # User documentation
│   ├── enterprise/      # Enterprise features
│   ├── extensions/      # VS Code extension
│   ├── function/        # Serverless functions
│   ├── identity/        # Authentication
│   ├── plugin/          # Plugin system
│   ├── sdk/             # JavaScript SDK
│   ├── slack/           # Slack integration
│   ├── ui/              # UI components
│   └── util/            # Utilities
├── github/              # GitHub Actions
├── infra/               # Infrastructure (SST)
├── nix/                 # Nix packages
├── script/              # Build scripts
├── sdks/                # SDK definitions
└── specs/               # OpenAPI specs
```

### Core Packages

| Package | Purpose |
|---------|---------|
| `packages/opencode` | Core business logic, agent system, server |
| `packages/sdk/js` | JavaScript SDK for external integrations |
| `packages/console` | Terminal UI built with SolidJS |
| `packages/docs` | Documentation (MDX-based) |

## Content Engineering

### Agent System

OpenCode implements a sophisticated multi-agent system with the following components:

#### Agent Types

```typescript
// Agent modes from agent.ts
const result: Record<string, Info> = {
  build: {
    name: "build",
    mode: "primary",
    native: true,
  },
  plan: {
    name: "plan",
    mode: "primary",
    native: true,
    permission: planPermission, // Read-only by default
  },
  general: {
    name: "general",
    mode: "subagent",
    native: true,
    hidden: true,
  },
  explore: {
    name: "explore",
    mode: "subagent",
    native: true,
    prompt: PROMPT_EXPLORE,
  },
  compaction: {
    name: "compaction",
    mode: "primary",
    native: true,
    hidden: true,
    prompt: PROMPT_COMPACTION,
  },
  title: {
    name: "title",
    mode: "primary",
    native: true,
    hidden: true,
    prompt: PROMPT_TITLE,
  },
  summary: {
    name: "summary",
    mode: "primary",
    native: true,
    hidden: true,
    prompt: PROMPT_SUMMARY,
  },
}
```

#### Agent Configuration Schema

```typescript
const Info = z.object({
  name: z.string(),
  description: z.string().optional(),
  mode: z.enum(["subagent", "primary", "all"]),
  native: z.boolean().optional(),
  hidden: z.boolean().optional(),
  default: z.boolean().optional(),
  topP: z.number().optional(),
  temperature: z.number().optional(),
  color: z.string().optional(),
  permission: z.object({
    edit: Config.Permission,
    bash: z.record(z.string(), Config.Permission),
    skill: z.record(z.string(), Config.Permission),
    webfetch: Config.Permission.optional(),
    doom_loop: Config.Permission.optional(),
    external_directory: Config.Permission.optional(),
  }),
  model: z.object({
    modelID: z.string(),
    providerID: z.string(),
  }).optional(),
  prompt: z.string().optional(),
  tools: z.record(z.string(), z.boolean()),
  options: z.record(z.string(), z.any()),
  maxSteps: z.number().int().positive().optional(),
})
```

### Prompt System

#### Prompt Files

OpenCode uses text-based prompt files loaded as modules:

```typescript
import PROMPT_GENERATE from "./generate.txt"
import PROMPT_COMPACTION from "./prompt/compaction.txt"
import PROMPT_EXPLORE from "./prompt/explore.txt"
import PROMPT_SUMMARY from "./prompt/summary.txt"
import PROMPT_TITLE from "./prompt/title.txt"
```

#### Prompt Templates

**Explore Prompt** (`packages/opencode/src/agent/prompt/explore.txt`):
Fast agent for code exploration with configurable thoroughness levels (quick, medium, very thorough).

**Compaction Prompt** (`packages/opencode/src/agent/prompt/compaction.txt`):
Handles context compression and summarization for long conversations.

**Summary Prompt** (`packages/opencode/src/agent/prompt/summary.txt`):
Generates conversation summaries.

**Title Prompt** (`packages/opencode/src/agent/prompt/title.txt`):
Generates descriptive titles for conversations.

#### System Prompt Header

```typescript
// packages/opencode/src/session/system.ts
const system = SystemPrompt.header(defaultModel.providerID)
system.push(PROMPT_GENERATE)
```

The system prompt header is provider-specific, allowing different instructions for Claude, OpenAI, Google, etc.

### Skills System

Skills are reusable agent capabilities defined in markdown files:

```bash
.opencode/skill/
└── test-skill/
    └── SKILL.md
```

**SKILL.md Structure**:
```markdown
# Skill Name

## Description
Brief description of what the skill does

## When to Use
When to invoke this skill

## Parameters
Input parameters for the skill

## Implementation
How the skill is executed
```

Skills can be invoked by the agent and have configurable permissions:

```typescript
permission: {
  skill: {
    "*": "allow",  // or "ask" / "deny"
  }
}
```

### Configuration File

**`.opencode/opencode.jsonc`**:
```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": ["STYLE_GUIDE.md"],
  "provider": {
    "opencode": {
      "options": {},
    },
  },
  "mcp": {},
  "tools": {
    "github-triage": false,
  },
}
```

Configuration allows:
- Agent definitions and customizations
- Provider configuration
- MCP server connections
- Tool enable/disable
- Permission policies

## Tool System

### Tool Registry

OpenCode implements a comprehensive tool registry at `packages/opencode/src/tool/`:

| Tool | Purpose |
|------|---------|
| `read.ts` | File reading with context awareness |
| `write.ts` | File writing |
| `edit.ts` | In-place file editing |
| `glob.ts` | File pattern matching |
| `grep.ts` | Content search |
| `bash.ts` | Shell command execution |
| `lsp.ts` | Language server protocol |
| `webfetch.ts` | Web content fetching |
| `websearch.ts` | Web search |
| `codesearch.ts` | Code search (Exa API) |
| `task.ts` | Task management |
| `todowrite.ts` | Todo list operations |
| `skill.ts` | Skill invocation |
| `batch.ts` | Batch operations |
| `multiedit.ts` | Multiple edits |
| `patch.ts` | Patch management |

### Tool Implementation Pattern

Each tool follows a consistent pattern:

```typescript
// Example from tool.ts
export interface Tool {
  name: string
  description: string
  parameters: z.ZodType
  execute: (args: any) => Promise<any>
  permission?: string
}

// Tool registry pattern
export const registry = {
  read: { ... },
  write: { ... },
  edit: { ... },
  // ...
}
```

### Parallel Tool Execution

OpenCode emphasizes parallel tool execution:

```markdown
# Tool Calling

- ALWAYS USE PARALLEL TOOLS WHEN APPLICABLE.
```

This is a core principle from the `AGENTS.md` file, enabling efficient batch operations.

## SDK Architecture

### JavaScript SDK

The SDK provides programmatic access to OpenCode:

```
packages/sdk/js/
├── src/
│   ├── client.ts      # Client factory
│   ├── server.ts      # Server implementation
│   ├── index.ts       # Main exports
│   └── v2/            # V2 API
├── openapi.json       # OpenAPI specification
└── gen/               # Generated client
```

#### Client Usage

```typescript
import { createOpencodeClient } from "@opencode-ai/sdk"

const client = createOpencodeClient({
  fetch: customFetch,
  directory: "/path/to/project",
  headers: {
    "x-custom-header": "value",
  },
})

// Use the client to interact with OpenCode
const result = await client.someMethod()
```

### Plugin System

**`packages/plugin`** provides integration capabilities:

```typescript
// Plugin interface
export interface Plugin {
  name: string
  version: string
  initialize: (context: PluginContext) => void
}
```

### OpenAPI Specification

The SDK is generated from OpenAPI specs at `packages/sdk/openapi.json`, enabling:
- Type-safe client generation
- API documentation
- Multi-language SDK generation

## MCP Integration

OpenCode supports Model Context Protocol (MCP) integration:

```jsonc
{
  "mcp": {
    "servers": {
      "github": {
        "command": "uvx",
        "args": ["mcp-server-github"],
        "env": {}
      }
    }
  }
}
```

MCP allows:
- Third-party tool integration
- Extended capabilities
- External service connections

## Agent Prompts

### Build Agent Prompt

The default agent uses a comprehensive system prompt covering:
- Code style guidelines
- Tool usage patterns
- Best practices for development

### Plan Agent Prompt

The plan agent is read-only with restricted permissions:
- File edits denied by default
- Bash commands require permission
- Ideal for exploration and analysis

### Subagent System

Subagents handle specific task types:

- **general** (@general): Multi-step tasks, parallel execution
- **explore**: Codebase exploration with configurable depth
- **compaction**: Context compression
- **title**: Title generation
- **summary**: Summary generation

## Development Workflow

### Running OpenCode

```bash
# Development mode
bun install
bun dev

# Against specific directory
bun dev <directory>

# Build standalone
./packages/opencode/script/build.ts --single
```

### Debugging

```bash
# With inspector
bun run --inspect=<url> dev ...

# Spawn mode (for breakpoint debugging)
bun dev spawn
```

### SDK Generation

When API changes occur, regenerate SDK:

```bash
./script/generate.ts
```

## Content Engineering Patterns

### 1. Text-Based Prompts

Prompts are stored as `.txt` files, loaded as modules, enabling:
- Version control friendly
- Easy editing without recompilation
- Separation of code and content

### 2. Prompt Compaction

Long conversations are compressed using specialized agents:
- Maintains context while reducing tokens
- Preserves important information
- Configurable compression levels

### 3. Dynamic Agent Generation

Agents can be generated on-demand:

```typescript
export async function generate(input: { description: string; model?: {...} }) {
  const result = await generateObject({
    messages: [...],
    schema: z.object({
      identifier: z.string(),
      whenToUse: z.string(),
      systemPrompt: z.string(),
    }),
  })
  return result.object
}
```

### 4. Permission System

Fine-grained permission control:
```typescript
permission: {
  edit: "allow" | "ask" | "deny",
  bash: {
    "git *": "allow",
    "rm *": "deny",
    "*": "ask",
  },
  skill: {
    "*": "allow",
  },
  webfetch: "allow",
  doom_loop: "ask",
  external_directory: "ask",
}
```

### 5. Skill Composition

Skills can be combined:
- Each skill is self-contained
- Skills have metadata for discovery
- Permission system controls skill usage

## Style Guide

OpenCode follows specific coding conventions:

```markdown
- Keep things in one function unless composable or reusable
- DO NOT do unnecessary destructuring
- DO NOT use `else` statements
- DO NOT use `try`/`catch` if avoidable
- AVOID `any` type
- AVOID `let` statements
- PREFER single word variable names
- Use Bun APIs where possible
```

## Integration Points

### GitHub Integration

```bash
.github/
├── actions/          # CI/CD workflows
├── script/           # GitHub Actions scripts
└── action.yml        # Action definition
```

### Infrastructure

```bash
infra/
├── app.ts            # SST application
├── console.ts        # Console configuration
├── enterprise.ts     # Enterprise setup
├── secret.ts         # Secret management
└── stage.ts          # Stage configuration
```

### Nix Support

```bash
nix/
├── bundle.ts         # Bundle configuration
├── hashes.json       # Package hashes
├── node-modules.nix  # Node modules
└── opencode.nix      # OpenCode package
```

## Strengths

1. **Provider Agnosticism**: Architecture supports multiple LLM providers
2. **Content Separation**: Prompts as files enables easy iteration
3. **Tool System**: Comprehensive, parallel tool execution
4. **SDK & Plugins**: Extensible architecture
5. **TUI Focus**: Terminal-first design philosophy
6. **Open Source**: 100% transparent development

## Areas for Enhancement

1. More documentation on content engineering patterns
2. Additional skill templates
3. Better prompt version management
4. Expanded MCP server support

## References

- **Repository**: https://github.com/sst/opencode
- **Documentation**: https://opencode.ai/docs
- **Discord**: https://opencode.ai/discord
- **Style Guide**: `ref/opencode/STYLE_GUIDE.md`
- **Contributing**: `ref/opencode/CONTRIBUTING.md`
