# OpenCode Usage Guide

A comprehensive guide to using OpenCode as an AI-powered coding assistant.

## Table of Contents

1. [Installation](#installation)
2. [Basic Usage](#basic-usage)
3. [Agent System](#agent-system)
4. [Tools Reference](#tools-reference)
5. [Configuration](#configuration)
6. [Skills System](#skills-system)
7. [Commands](#commands)
8. [Best Practices](#best-practices)
9. [Advanced Usage](#advanced-usage)

---

## Installation

### Quick Install

```bash
# Using curl (YOLO)
curl -fsSL https://opencode.ai/install | bash

# Using npm
npm i -g opencode-ai@latest

# Using Homebrew (macOS/Linux)
brew install opencode

# Using mise
mise use -g opencode
```

### Desktop App

Download from [opencode.ai/download](https://opencode.ai/download) or:

```bash
brew install --cask opencode-desktop  # macOS
```

### Development Installation

```bash
git clone https://github.com/sst/opencode.git
cd opencode
bun install
bun dev
```

---

## Basic Usage

### Starting OpenCode

```bash
# Run in current directory
opencode

# Run in specific directory
opencode /path/to/project

# Attach to running server
opencode attach http://localhost:4096

# Start as server
opencode serve --port 4096
```

### First Conversation

1. Type your request in natural language
2. Press `Enter` to submit
3. OpenCode will:
   - Read relevant files
   - Make edits
   - Run commands as needed
   - Ask for permission when required

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Tab` | Switch agent |
| `Ctrl+C` | Interrupt response |
| `Ctrl+P` | List commands |
| `<leader>a` | List agents |
| `<leader>m` | List models |
| `<leader>s` | View status |
| `<leader>x` | Export session |

---

## Agent System

### Built-in Agents

#### Build Agent (Default)

Full-access agent for development work:
- Can edit files
- Can run any bash command
- Use for coding, refactoring, debugging

```bash
# Default mode - full access enabled
```

#### Plan Agent

Read-only agent for analysis:
- File edits denied by default
- Bash commands require permission
- Use for:
  - Code exploration
  - Architecture analysis
  - Planning changes

```bash
# Press Tab to switch to plan mode
```

#### Subagents

Specialized agents for specific tasks:

| Subagent | Purpose |
|----------|---------|
| `@general` | Multi-step tasks with parallel execution |
| `@explore` | Fast codebase exploration |
| `@compaction` | Context compression |
| `@title` | Generate conversation titles |
| `@summary` | Generate summaries |

```markdown
# Usage example
@explore Find all API endpoints in this project

# Parallel execution
@general Research React patterns and implement a component
```

### Creating Custom Agents

#### Interactive Creation

```bash
opencode agent create
```

Follow the prompts to:
1. Choose location (project/global)
2. Enter description
3. Select tools
4. Choose mode (all/primary/subagent)

#### Programmatic Creation

```bash
opencode agent create \
  --path ./.opencode \
  --description "React component expert" \
  --mode primary \
  --tools "read,write,edit,glob"
```

#### Agent File Format

Agents are defined in `.opencode/agent/*.md`:

```markdown
---
mode: primary
description: Expert at React component development
model: anthropic/claude-sonnet-4
tools:
  read: true
  write: true
  edit: true
  glob: true
  bash: true
---

You are a React expert. Help users create clean, performant React components.

## Guidelines

- Use functional components with hooks
- Prefer TypeScript
- Follow accessibility best practices
```

### Agent Configuration Options

```markdown
---
mode: subagent              # all | primary | subagent
description: When to use this agent
model: provider/model       # Specific model
temperature: 0.3           # Creativity (0-1)
top_p: 0.9                 # Nucleus sampling
maxSteps: 50               # Max iterations
color: "#FF5733"           # UI color
tools:
  read: true
  write: false
permission:
  edit: ask                # ask | allow | deny
  bash:
    "git *": allow
    "rm *": deny
    "*": ask
---
```

---

## Tools Reference

### File Operations

#### Read

```typescript
// Read file with line numbers
await read({
  filePath: "src/index.ts",
  offset: 0,      // Start line (0-based)
  limit: 100      // Number of lines
})
```

**Features:**
- Line numbering
- Binary file detection
- Image preview support
- Suggestion for typos

#### Write

```typescript
// Create or overwrite file
await write({
  filePath: "src/new-file.ts",
  content: "..."
})
```

#### Edit

```typescript
// In-place editing
await edit({
  filePath: "src/file.ts",
  oldString: "const old = 'value'",
  newString: "const new = 'new value'"
})
```

#### Glob

```typescript
// Find files by pattern
await glob({
  pattern: "**/*.test.ts"
})

// Multiple patterns
await glob({
  pattern: "{src,lib}/**/*.ts"
})
```

#### Grep

```typescript
// Search file contents
await grep({
  pattern: "function\\s+\\w+",
  include: "*.ts"
})
```

### Shell Commands

#### Bash

```typescript
await bash({
  command: "npm run build",
  description: "Build the project",
  timeout: 120000  // 2 minutes
})
```

**Permission Levels:**
- `allow` - Runs without asking
- `ask` - Prompts for permission
- `deny` - Never runs

### Web Tools

#### Web Fetch

```typescript
await webfetch({
  url: "https://api.example.com/docs"
})
```

#### Web Search

```typescript
await websearch({
  query: "OpenCode AI coding agent features",
  numResults: 5
})
```

#### Code Search

```typescript
await codesearch({
  query: "React useState hook examples",
  tokensNum: 5000
})
```

### Development Tools

#### LSP

Language Server Protocol integration for:
- Code completion
- Diagnostics
- Go to definition
- Find references

```typescript
await lsp({
  command: "typescript-language-server",
  args: ["--stdio"]
})
```

#### Task Management

```typescript
await todowrite({
  todos: [
    { id: "1", content: "Implement feature", status: "in_progress" },
    { id: "2", content: "Write tests", status: "pending" }
  ]
})
```

---

## Configuration

### Configuration File Location

1. Project: `.opencode/opencode.jsonc`
2. Global: `~/.config/opencode/opencode.jsonc`

### Basic Configuration

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4",
  "default_agent": "build",
  
  "instructions": [
    "STYLE_GUIDE.md",
    "coding-standards.md"
  ],
  
  "agent": {
    "build": {
      "temperature": 0.3
    }
  },
  
  "provider": {
    "anthropic": {
      "options": {
        "apiKey": "{env:ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

### Provider Configuration

```jsonc
{
  "provider": {
    "anthropic": {
      "options": {
        "apiKey": "{env:ANTHROPIC_API_KEY}",
        "baseURL": "https://api.anthropic.com"
      }
    },
    "openai": {
      "options": {
        "apiKey": "{env:OPENAI_API_KEY}"
      }
    },
    "google-vertex": {
      "options": {
        "project": "{env:GOOGLE_CLOUD_PROJECT}",
        "location": "us-east5"
      }
    }
  }
}
```

### Permission Configuration

```jsonc
{
  "permission": {
    "edit": "ask",
    "bash": {
      "git *": "allow",
      "npm *": "allow",
      "rm *": "deny",
      "*": "ask"
    },
    "skill": {
      "*": "allow"
    },
    "webfetch": "allow",
    "external_directory": "ask"
  }
}
```

### MCP Configuration

```jsonc
{
  "mcp": {
    "github": {
      "type": "local",
      "command": ["uvx", "mcp-server-github"],
      "enabled": true,
      "timeout": 5000
    },
    "filesystem": {
      "type": "remote",
      "url": "https://mcp.example.com/filesystem",
      "headers": {
        "Authorization": "Bearer {env:MCP_TOKEN}"
      }
    }
  }
}
```

### LSP Configuration

```jsonc
{
  "lsp": {
    "typescript": {
      "command": ["typescript-language-server", "--stdio"],
      "extensions": [".ts", ".tsx"]
    },
    "python": {
      "command": ["pyright", "--stdio"],
      "extensions": [".py"]
    }
  }
}
```

### Compaction Settings

```jsonc
{
  "compaction": {
    "auto": true,    // Auto-compact when context is full
    "prune": true    // Prune old tool outputs
  }
}
```

---

## Skills System

Skills are reusable agent capabilities.

### Built-in Skills

Skills are automatically discovered from:
- `.opencode/skill/*/SKILL.md`
- `.claude/skills/*/SKILL.md`
- `~/.claude/skills/`

### Skill File Format

```markdown
---
name: test-skill
description: Use this when asked to test skill
---

This skill tests the skill system.

## When to Use

When someone asks you to demonstrate skills.

## Parameters

No parameters required.

## Example

```
@skill test-skill
```
```

### Creating a Custom Skill

1. Create directory: `.opencode/skill/my-skill/`
2. Add `SKILL.md`:

```markdown
---
name: my-skill
description: Expert at database migrations
---

You are a database migration expert.

## Guidelines

- Always use transactions
- Test migrations locally
- Include rollback scripts
- Document schema changes

## Example

```sql
-- Safe migration pattern
BEGIN TRANSACTION;

-- Your migration here

COMMIT;
```
```

### Using Skills

```markdown
# Invoke by name
@skill my-skill

# Or describe what you need
Help me create a database migration for adding users table
```

---

## Commands

### Built-in Commands

Commands are reusable prompts defined in `.opencode/command/*.md`.

#### Command File Format

```markdown
---
agent: build
model: anthropic/claude-sonnet-4
subtask: true
---

Create a comprehensive test suite for the following code:

{{file}}
```

#### Creating a Command

1. Create `.opencode/command/test-suite.md`:

```markdown
---
agent: build
description: Generate unit tests
---

Generate unit tests for the following code. Use the testing framework that's already in the project.

Code to test:
{{file}}
```

2. Use it:

```markdown
/opencode test-suite
```

### Available Commands

```bash
# List available commands
Ctrl+P

# Run a command
/opencode <command-name>
```

### Managing Agents

```bash
# List all agents
opencode agent list

# Create a new agent
opencode agent create

# Options
opencode agent create \
  --path ./.opencode \
  --description "My custom agent" \
  --mode primary \
  --tools "read,write,edit"
```

### Managing Models

```bash
# List available models
opencode models

# Show model details
opencode models --model anthropic/claude-sonnet-4
```

### Managing MCP Servers

```bash
# List MCP servers
opencode mcp list

# Add MCP server
opencode mcp add <name>

# Remove MCP server
opencode mcp remove <name>
```

---

## Best Practices

### 1. Use the Right Agent

| Task | Agent |
|------|-------|
| Writing code | build |
| Debugging | build |
| Planning changes | plan |
| Exploring codebase | @explore |
| Multi-step tasks | @general |

### 2. Leverage Permissions

```jsonc
{
  "permission": {
    "bash": {
      "git status": "allow",
      "git diff": "allow",
      "npm test": "allow",
      "npm run build": "allow",
      "rm *": "deny",
      "*": "ask"
    }
  }
}
```

### 3. Organize Project Knowledge

```
project/
├── .opencode/
│   ├── agent/          # Custom agents
│   ├── command/        # Custom commands
│   ├── skill/          # Custom skills
│   ├── opencode.jsonc  # Configuration
│   └── themes/         # UI themes
├── STYLE_GUIDE.md      # Code style
├── README.md           # Project docs
└── docs/              # Additional docs
```

### 4. Use Parallel Tools

Always use parallel tool execution when possible:

```typescript
// Instead of sequential:
const file1 = await read({ filePath: "file1.ts" })
const file2 = await read({ filePath: "file2.ts" })

// Use parallel:
const [file1, file2] = await Promise.all([
  read({ filePath: "file1.ts" }),
  read({ filePath: "file2.ts" })
])
```

### 5. Manage Context

- Use `@summary` for long conversations
- Enable compaction in config
- Break large tasks into smaller steps

### 6. Security Best Practices

```jsonc
{
  "permission": {
    "edit": "ask",
    "bash": {
      "rm *": "deny",
      "chmod *": "deny",
      "*": "ask"
    },
    "external_directory": "deny"
  }
}
```

---

## Advanced Usage

### Remote Development

```bash
# Start server
opencode serve --port 4096 --hostname 0.0.0.0

# Connect from anywhere
opencode attach http://your-server:4096
```

### Debugging OpenCode

```bash
# With inspector
bun run --inspect=ws://localhost:6499/ dev

# Spawn mode for breakpoint debugging
bun dev spawn
```

### Custom Formatters

```jsonc
{
  "formatter": {
    "prettier": {
      "command": ["prettier", "--write"],
      "extensions": [".ts", ".js", ".json"]
    }
  }
}
```

### Environment Variables

```bash
# API Keys
export ANTHROPIC_API_KEY="sk-..."
export OPENAI_API_KEY="sk-..."
export GOOGLE_CLOUD_PROJECT="my-project"

# Configuration
export OPENCODE_CONFIG="/path/to/config"
export OPENCODE_CONFIG_DIR="/path/to/config-dir"
```

### Plugin Development

```typescript
// packages/plugin/index.ts
export default {
  name: "my-plugin",
  version: "1.0.0",
  
  async onLoad() {
    // Initialize plugin
  },
  
  tools: {
    myTool: async () => ({
      execute: async (params) => {
        // Tool implementation
      }
    })
  }
}
```

### Batch Operations

```typescript
await batch({
  operations: [
    { type: "read", filePath: "file1.ts" },
    { type: "read", filePath: "file2.ts" },
    { type: "edit", filePath: "file3.ts", ... }
  ]
})
```

### Session Management

```bash
# List sessions
opencode session list

# Export session
opencode session export <session-id>

# Fork session
opencode session fork <session-id>
```

---

## Troubleshooting

### Common Issues

#### Model Not Found

```bash
# Check available models
opencode models

# Set default model
opencode config set model anthropic/claude-sonnet-4
```

#### Permission Denied

```bash
# Check permission settings
opencode config get permission

# Update permissions
opencode config set permission.bash."*" allow
```

#### MCP Server Issues

```bash
# List MCP servers
opencode mcp list

# Check server status
opencode mcp list --verbose

# Restart server
opencode mcp remove <name>
opencode mcp add <name>
```

### Getting Help

```bash
# View help
opencode --help

# View command help
opencode agent --help
opencode mcp --help

# Check logs
opencode --print-logs --log-level DEBUG
```

---

## File Reference

### Configuration Schema

```typescript
interface Config {
  $schema?: string
  theme?: string
  model?: string
  small_model?: string
  default_agent?: string
  instructions?: string[]
  agent?: Record<string, AgentConfig>
  provider?: Record<string, ProviderConfig>
  mcp?: Record<string, McpConfig>
  lsp?: Record<string, LspConfig>
  permission?: PermissionConfig
  compaction?: CompactionConfig
  experimental?: ExperimentalConfig
}
```

### Agent Schema

```typescript
interface AgentConfig {
  mode?: "subagent" | "primary" | "all"
  description?: string
  model?: string
  temperature?: number
  top_p?: number
  maxSteps?: number
  color?: string
  tools?: Record<string, boolean>
  permission?: PermissionConfig
  prompt?: string
}
```

---

## Resources

- **Documentation**: https://opencode.ai/docs
- **GitHub**: https://github.com/sst/opencode
- **Discord**: https://opencode.ai/discord
- **Issues**: https://github.com/sst/opencode/issues

---

## Quick Reference Card

### Essential Commands

```bash
opencode                    # Start in current directory
opencode /path              # Start in specific directory
opencode attach <url>       # Attach to server
opencode agent create       # Create new agent
opencode agent list         # List agents
opencode models             # List models
opencode mcp list           # List MCP servers
```

### Agent Switching

| Key | Action |
|-----|--------|
| `Tab` | Switch agent |
| `@general` | Call general subagent |
| `@explore` | Call explore subagent |
| `@skill <name>` | Call specific skill |

### Tool Names

| Tool | Purpose |
|------|---------|
| `read` | Read files |
| `write` | Write files |
| `edit` | Edit files |
| `glob` | Find files |
| `grep` | Search content |
| `bash` | Run commands |
| `webfetch` | Fetch URLs |
| `websearch` | Web search |
| `codesearch` | Code search |
| `lsp` | Language server |
| `task` | Task management |
| `todowrite` | Todo lists |

---

*Last updated: December 2025*
