# On-Demand Rule Loading for AI Agents

## Core Concept: Just-in-Time Rule Retrieval

The fundamental insight is that **agents don't need all rules at once** - they need **relevant rules at the right time**. This approach, sometimes called **Just-in-Time (JIT) Rule Compilation**, involves fetching and presenting rules dynamically based on the specific task, file, or context an agent is working with.

This document provides specific techniques, tools, and code implementations for making OpenCode, Codex, Claude Code, and other agents load rules on-demand rather than all at once.

---

## Technique 1: Semantic Rule References (OpenCode Native)

OpenCode supports a built-in mechanism for referencing external rule files that agents load on-demand:

### Implementation

```markdown
# AGENTS.md with On-Demand References

## Critical Rules (Always Load)
These rules are always included:
- **Security**: Never commit secrets or API keys
- **Testing**: All new functions must have unit tests
- **Type Safety**: Use TypeScript strict mode

## Task-Specific Rules (Load on Demand)

When working with TypeScript code, load:
- @docs/typescript-style.md
- @docs/error-handling.md
- @docs/type-patterns.md

When working with React components, load:
- @docs/react-components.md
- @docs/hooks-patterns.md
- @docs/state-management.md

When working with APIs, load:
- @docs/api-design.md
- @docs/authentication.md
- @docs/error-responses.md

When writing tests, load:
- @docs/testing-patterns.md
- @docs/test-organization.md
- @docs/mock-strategies.md

## Instructions for Rule Loading
- Do NOT load all @docs/*.md files at once
- Load only the files relevant to the CURRENT task
- When you start working on a file type, load its associated rules
- When you switch file types, load the new rules
- Keep previously loaded rules only if they're still relevant
```

### Agent Instruction for Using References

```markdown
# Rule Loading Protocol

## When Starting a New Task
1. Identify the file type(s) you'll be working with
2. Load the associated rule files using the @reference syntax
3. Keep only active rules in context

## Example Workflow
Task: "Add error handling to api/users.ts"

Step 1: Identify file type = TypeScript API code
Step 2: Load relevant rules:
  - @docs/typescript-style.md
  - @docs/error-handling.md  
  - @docs/api-design.md
Step 3: Work with these rules in context
Step 4: If switching to test file, load @docs/testing-patterns.md

## Important
- Always use your Read tool to load @reference files when starting tasks
- Don't preemptively load all references
- Focus on rules relevant to the SPECIFIC task at hand
```

---

## Technique 2: opencode.json Rule Module System

OpenCode's `opencode.json` supports modular rule loading with glob patterns:

### Configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "AGENTS.md",
    "docs/core-standards.md",
    {
      "type": "module",
      "patterns": [
        "packages/*/AGENTS.md",
        "packages/*/rules.md"
      ],
      "load_strategy": "on-demand",
      "default_rules": ["packages/core/AGENTS.md"],
      "conditional_rules": {
        "**/src/**/*.test.ts": ["packages/*/test-rules.md"],
        "**/src/**/*.api.ts": ["packages/*/api-rules.md"],
        "**/src/components/**/*": ["packages/*/component-rules.md"]
      }
    },
    "docs/security-rules.md",
    {
      "path": "rules",
      "load_strategy": "lazy",
      "glob": "*.md",
      "priority": ["critical.md", "important.md", "guidelines.md"]
    }
  ]
}
```

### Directory Structure

```
project/
├── opencode.json
├── AGENTS.md
├── docs/
│   ├── core-standards.md
│   ├── security-rules.md
│   ├── typescript-style.md
│   ├── error-handling.md
│   ├── api-design.md
│   ├── react-components.md
│   └── testing-patterns.md
├── rules/
│   ├── critical.md      # Always load
│   ├── important.md     # Load based on context
│   └── guidelines.md    # Load on demand
└── packages/
    ├── core/
    │   ├── AGENTS.md
    │   ├── rules.md
    │   └── src/
    ├── api/
    │   ├── AGENTS.md
    │   ├── api-rules.md
    │   └── src/
    └── web/
        ├── AGENTS.md
        ├── component-rules.md
        └── src/
```

---

## Technique 3: Rule Tags and Conditional Loading

Create rule files with metadata for conditional loading:

### Rule File with Tags

```markdown
---
title: TypeScript Error Handling Rules
tags: [typescript, error-handling, backend, api]
priority: high
applies_to:
  - "**/*.ts"
  - "**/src/**/*"
excludes:
  - "**/*.test.ts"
  - "**/*.d.ts"
---

# TypeScript Error Handling Rules

## Use Result Types for Fallible Operations

✅ **DO THIS:**
```typescript
type Result<T, E = Error> = 
  | { ok: true; value: T }
  | { ok: false; error: E };

async function fetchUser(id: string): Promise<Result<User>> {
  const user = await db.users.find(id);
  if (!user) {
    return { ok: false, error: new UserNotFoundError(id) };
  }
  return { ok: true, value: user };
}
```

❌ **DON'T DO THIS:**
```typescript
async function fetchUser(id: string): Promise<User | null> {
  try {
    const user = await db.users.find(id);
    return user ?? null;
  } catch (error) {
    console.error('Failed to fetch user:', error);
    return null;
  }
}
```

## Error Classification

All errors should be classified:

| Error Type | Handling Pattern | Example |
|------------|------------------|---------|
| Validation | Return Result with typed error | `InvalidInputError` |
| Not Found | Return Result with null | `UserNotFoundError` |
| Transient | Retry with backoff | `NetworkTimeoutError` |
| Fatal | Propagate exception | `DatabaseCorruptionError` |
```

### Rule Loader Script

```typescript
// scripts/rule-loader.ts

import * as fs from 'fs';
import * as path from 'path';
import matter from 'gray-matter';

interface RuleMetadata {
  title: string;
  tags: string[];
  priority: 'critical' | 'high' | 'medium' | 'low';
  applies_to: string[];
  excludes: string[];
}

interface RuleFile {
  path: string;
  metadata: RuleMetadata;
  content: string;
}

class RuleLoader {
  private ruleDirectory: string;
  private loadedRules: Map<string, RuleFile> = new Map();

  constructor(ruleDirectory: string) {
    this.ruleDirectory = ruleDirectory;
  }

  async loadRulesForContext(
    filePath: string,
    taskType: 'edit' | 'create' | 'review' | 'test'
  ): Promise<string[]> {
    const relevantRules: string[] = [];

    // Load critical rules first
    const criticalRules = await this.loadByTag('critical');
    relevantRules.push(...criticalRules);

    // Determine file tags
    const fileTags = this.inferTags(filePath);
    
    // Load file-specific rules
    const fileRules = await this.loadByFilePattern(filePath);
    relevantRules.push(...fileRules);

    // Load task-specific rules
    const taskRules = await this.loadByTaskType(taskType);
    relevantRules.push(...taskRules);

    // Load tag-based rules
    for (const tag of fileTags) {
      const taggedRules = await this.loadByTag(tag);
      relevantRules.push(...taggedRules);
    }

    return [...new Set(relevantRules)];
  }

  private async loadByTag(tag: string): Promise<string[]> {
    const rules: string[] = [];
    
    for (const [path, rule] of this.loadedRules) {
      if (rule.metadata.tags.includes(tag)) {
        rules.push(rule.content);
      }
    }
    
    return rules;
  }

  private async loadByFilePattern(filePath: string): Promise<string[]> {
    const rules: string[] = [];
    
    for (const [path, rule] of this.loadedRules) {
      for (const pattern of rule.metadata.applies_to) {
        if (this.matchPattern(pattern, filePath)) {
          // Check exclusions
          const isExcluded = rule.metadata.excludes.some(ex => 
            this.matchPattern(ex, filePath)
          );
          if (!isExcluded) {
            rules.push(rule.content);
          }
        }
      }
    }
    
    return rules;
  }

  private async loadByTaskType(taskType: string): Promise<string[]> {
    const taskRules: Record<string, string[]> = {
      'edit': ['task-edit.md'],
      'create': ['task-create.md'],
      'review': ['task-review.md'],
      'test': ['task-test.md', 'testing-patterns.md']
    };
    
    const rules: string[] = [];
    for (const ruleFile of taskRules[taskType] || []) {
      const rule = this.loadedRules.get(ruleFile);
      if (rule) {
        rules.push(rule.content);
      }
    }
    return rules;
  }

  private inferTags(filePath: string): string[] {
    const tags: string[] = [];
    
    const extensionMap: Record<string, string[]> = {
      '.ts': ['typescript', 'backend'],
      '.tsx': ['typescript', 'react', 'frontend'],
      '.js': ['javascript', 'frontend'],
      '.test.ts': ['typescript', 'testing'],
      '.api.ts': ['typescript', 'api', 'backend'],
      '.component.tsx': ['typescript', 'react', 'component']
    };
    
    const ext = path.extname(filePath);
    if (extensionMap[ext]) {
      tags.push(...extensionMap[ext]);
    }
    
    // Check directory structure
    if (filePath.includes('/api/') || filePath.includes('/routes/')) {
      tags.push('api');
    }
    if (filePath.includes('/components/')) {
      tags.push('component');
    }
    if (filePath.includes('/test/') || filePath.includes('/__tests__/')) {
      tags.push('testing');
    }
    
    return tags;
  }

  private matchPattern(pattern: string, filePath: string): boolean {
    // Simple glob matching
    const regex = new RegExp(
      '^' + pattern
        .replace(/\*\*/g, '.*')
        .replace(/\*/g, '[^/]*')
        .replace(/\?/g, '.')
      + '$'
    );
    return regex.test(filePath);
  }

  async loadRuleFiles(): Promise<void> {
    const files = await this.findRuleFiles(this.ruleDirectory);
    
    for (const file of files) {
      const content = fs.readFileSync(file, 'utf-8');
      const { data, content: ruleContent } = matter(content);
      
      this.loadedRules.set(path.relative(this.ruleDirectory, file), {
        path: file,
        metadata: data as RuleMetadata,
        content: ruleContent
      });
    }
  }

  private async findRuleFiles(dir: string): Promise<string[]> {
    const files: string[] = [];
    const entries = await fs.promises.readdir(dir, { withFileTypes: true });
    
    for (const entry of entries) {
      const fullPath = path.join(dir, entry.name);
      if (entry.isDirectory()) {
        files.push(...await this.findRuleFiles(fullPath));
      } else if (entry.name.endsWith('.md') && entry.name.startsWith('rule-')) {
        files.push(fullPath);
      }
    }
    
    return files;
  }
}

// Usage
const loader = new RuleLoader('./rules');
await loader.loadRuleFiles();

const relevantRules = await loader.loadRulesForContext(
  'packages/api/src/users.api.ts',
  'create'
);

console.log('Relevant rules for this task:', relevantRules.length);
```

---

## Technique 4: Semantic Rule Search with Vector Database

For advanced rule retrieval, use semantic search to find relevant rules based on task description:

```typescript
// semantic-rule-search.ts

import { OpenAIEmbeddings } from '@langchain/openai';
import { Chroma } from '@langchain/community/vectorstores/chroma';

interface RuleChunk {
  id: string;
  content: string;
  file: string;
  tags: string[];
  priority: number;
}

class SemanticRuleSearch {
  private vectorStore: Chroma;
  private embeddings: OpenAIEmbeddings;
  private ruleChunks: Map<string, RuleChunk> = new Map();

  constructor() {
    this.embeddings = new OpenAIEmbeddings();
    this.vectorStore = new Chroma(
      this.embeddings,
      { collectionName: 'project-rules' }
    );
  }

  async indexRules(rules: { path: string; content: string }[]): Promise<void> {
    const chunks: RuleChunk[] = [];

    for (const rule of rules) {
      // Split rules into semantic chunks
      const ruleChunks = this.chunkRuleContent(rule.content, rule.path);
      chunks.push(...ruleChunks);
    }

    // Create embeddings and store
    const texts = chunks.map(c => c.content);
    const metadatas = chunks.map(c => ({
      id: c.id,
      file: c.file,
      tags: c.tags,
      priority: c.priority
    }));

    await this.vectorStore.addTexts(texts, metadatas);
  }

  async searchRelevantRules(
    taskDescription: string,
    currentFile: string,
    options: {
      maxResults?: number;
      minScore?: number;
      tagFilter?: string[];
    } = {}
  ): Promise<RuleChunk[]> {
    const {
      maxResults = 5,
      minScore = 0.7,
      tagFilter = []
    } = options;

    // Search by semantic similarity
    const results = await this.vectorStore.similaritySearchWithScore(
      taskDescription,
      maxResults * 2
    );

    // Filter and rank results
    const filtered = results
      .filter(([doc, score]) => score >= minScore)
      .map(([doc, score]) => ({
        ...this.ruleChunks.get(doc.metadata.id)!,
        score
      }))
      .filter(chunk => 
        tagFilter.length === 0 || 
        chunk.tags.some(t => tagFilter.includes(t))
      )
      .sort((a, b) => b.priority - a.priority)
      .slice(0, maxResults);

    return filtered;
  }

  private chunkRuleContent(
    content: string,
    file: string,
    chunkSize: number = 1000
  ): RuleChunk[] {
    const chunks: RuleChunk[] = [];
    const lines = content.split('\n');
    
    let currentChunk = '';
    let currentTags: string[] = [];
    let priority = 1;
    let chunkId = 0;

    for (const line of lines) {
      // Detect tags and priority from comments
      const tagMatch = line.match(/^#+\s*@(\w+):\s*(.+)/);
      const priorityMatch = line.match(/@priority:\s*(\d+)/);
      const sectionMatch = line.match(/^##+\s+(.+)/);

      if (tagMatch) {
        currentTags.push(tagMatch[1]);
      }
      if (priorityMatch) {
        priority = parseInt(priorityMatch[1]);
      }
      if (sectionMatch && currentChunk.length > 500) {
        // Save current chunk and start new one
        chunks.push({
          id: `${file}-${chunkId++}`,
          content: currentChunk,
          file,
          tags: [...currentTags],
          priority
        });
        currentChunk = '';
      }

      currentChunk += line + '\n';

      if (currentChunk.length >= chunkSize) {
        chunks.push({
          id: `${file}-${chunkId++}`,
          content: currentChunk,
          file,
          tags: [...currentTags],
          priority
        });
        currentChunk = '';
      }
    }

    // Add final chunk
    if (currentChunk.trim()) {
      chunks.push({
        id: `${file}-${chunkId}`,
        content: currentChunk,
        file,
        tags: [...currentTags],
        priority
      });
    }

    // Store chunks for later retrieval
    chunks.forEach(c => this.ruleChunks.set(c.id, c));

    return chunks;
  }
}

// Usage
const ruleSearch = new SemanticRuleSearch();

await ruleSearch.indexRules([
  { path: 'docs/typescript-style.md', content: tsStyleContent },
  { path: 'docs/error-handling.md', content: errorHandlingContent },
  { path: 'docs/api-design.md', content: apiDesignContent },
]);

// When starting a task
const relevantRules = await ruleSearch.searchRelevantRules(
  'Add error handling to the user API endpoint that validates input and returns proper error responses',
  'packages/api/src/users.api.ts',
  { maxResults: 5, tagFilter: ['typescript', 'error-handling'] }
);

console.log('Top relevant rules:', relevantRules.map(r => r.content));
```

---

## Technique 5: Codex AGENTS.md Override System

Codex supports `AGENTS.override.md` for task-specific overrides:

```markdown
# Command Pattern for Task-Specific Rules

For complex projects, use command patterns to specify rule sets:

## Available Rule Sets

Run Codex with specific rule configurations:

```bash
# Use only critical rules
codex --agents-override critical-only "Implement feature X"

# Use API-focused rules
codex --agents-override api-rules "Work on API endpoints"

# Use frontend-focused rules  
codex --agents-override frontend-rules "Work on React components"

# Use full rules (default)
codex --agents-override full "Comprehensive review"
```

## Rule Set Definitions

### critical-only
- Security rules only
- Type safety rules only
- No style/linting rules

### api-rules
- API design patterns
- Error handling for APIs
- Authentication/authorization
- REST/GraphQL conventions

### frontend-rules
- React patterns
- Component structure
- State management
- Performance optimization
```

### Codex Configuration for Rule Sets

```toml
# ~/.codex/config.toml

[rule-sets.critical-only]
priority = ["security", "type-safety"]
files = ["AGENTS.md"]
exclude = ["docs/style.md", "docs/linting.md"]

[rule-sets.api-rules]
priority = ["api-design", "error-handling", "auth"]
files = ["AGENTS.md", "docs/api-rules.md", "docs/auth-rules.md"]

[rule-sets.frontend-rules]
priority = ["react", "components", "state"]
files = ["AGENTS.md", "docs/react-rules.md", "docs/component-rules.md"]

[rule-sets.full]
priority = ["all"]
files = ["AGENTS.md", "docs/**/*.md"]
```

---

## Technique 6: Claude Code Context Management

Claude Code supports `.claude/` directory for structured context:

```
.claude/
├── CLAUDE.md              # Main context (keep < 20K tokens)
├── rules/
│   ├── critical/          # Critical rules only
│   │   ├── security.md
│   │   ├── type-safety.md
│   │   └── testing.md
│   ├── important/         # Important rules
│   │   ├── error-handling.md
│   │   ├── documentation.md
│   │   └── api-design.md
│   └── guidelines/        # Style guidelines
│       ├── typescript.md
│       ├── react.md
│       └── general.md
├── prompts/
│   ├── coding.md
│   ├── review.md
│   ├── testing.md
│   └── debugging.md
└── context/
    └── project-summary.json
```

### Context Loading Script

```typescript
// .claude/context-loader.ts

interface TaskContext {
  taskType: 'create' | 'edit' | 'review' | 'debug' | 'test';
  fileTypes: string[];
  scope: 'small' | 'medium' | 'large';
}

class ClaudeContextLoader {
  async loadContext(task: TaskContext): Promise<string> {
    const parts: string[] = [];

    // Always load critical rules
    parts.push(await this.loadFile('.claude/rules/critical/*.md'));

    // Load task-specific rules
    const taskRules = await this.loadFile(`.claude/prompts/${task.taskType}.md`);
    parts.push(taskRules);

    // Load file-type specific rules
    for (const fileType of task.fileTypes) {
      const typeRules = await this.loadFile(`.claude/rules/**/*.md`, 
        { tag: fileType }
      );
      if (typeRules) parts.push(typeRules);
    }

    // Load scope-appropriate rules
    if (task.scope === 'large') {
      parts.push(await this.loadFile('.claude/rules/important/*.md'));
    }

    // Combine and truncate if needed
    const combined = parts.join('\n\n---\n\n');
    return this.truncateToContextLimit(combined, 100000); // 100K tokens max
  }

  private async loadFile(pattern: string, filters?: { tag?: string }): Promise<string> {
    // Implementation for loading and filtering rule files
    // Returns only relevant content
  }

  private truncateToContextLimit(content: string, maxTokens: number): string {
    // Truncate to fit in context window while preserving structure
  }
}
```

---

## Technique 7: Gemini CLI Rule Configuration

Gemini CLI uses `settings.json` for context management:

```json
// ~/.gemini/settings.json
{
  "contextFileName": "AGENTS.md",
  "contextManagement": {
    "strategy": "semantic",
    "maxContextSize": 150000,
    "ruleCategories": {
      "critical": {
        "files": ["AGENTS.md", "security-rules.md"],
        "alwaysInclude": true
      },
      "important": {
        "files": ["docs/*.md"],
        "includeBasedOn": "semantic-relevance"
      },
      "guidance": {
        "files": ["guides/*.md"],
        "includeBasedOn": "explicit-request"
      }
    },
    "semanticSearch": {
      "enabled": true,
      "model": "gemini-1.5-pro",
      "topK": 5,
      "threshold": 0.75
    }
  },
  "tools": {
    "read": {
      "enabled": true,
      "autoLoadRules": false,
      "rulePatterns": ["@rules/**", "@docs/**"]
    }
  }
}
```

---

## Technique 8: MCP Rule Server

Use Model Context Protocol (MCP) to create a dedicated rule server:

```typescript
// mcp-rules-server/src/index.ts

import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';

const server = new Server({
  name: 'rules-server',
  version: '1.0.0',
});

interface RuleDatabase {
  rules: Map<string, { content: string; tags: string[]; priority: number }>;
  embeddings: Map<string, number[]>;
}

const ruleDB: RuleDatabase = {
  rules: new Map(),
  embeddings: new Map()
};

// Tool: search_rules
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === 'search_rules') {
    const { query, fileType, taskType, maxResults = 5 } = args as {
      query: string;
      fileType?: string;
      taskType?: string;
      maxResults?: number;
    };

    // Semantic search for relevant rules
    const relevantRules = await semanticSearch(query, {
      fileType,
      taskType,
      maxResults
    });

    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify({
            rules: relevantRules,
            totalFound: relevantRules.length
          }, null, 2)
        }
      ]
    };
  }

  if (name === 'get_rules_for_file') {
    const { filePath, task } = args as { filePath: string; task: string };
    
    const rules = await getRulesForFileAndTask(filePath, task);
    
    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify({ rules }, null, 2)
        }
      ]
    };
  }
});

server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: 'search_rules',
        description: 'Search for relevant rules based on a query',
        inputSchema: {
          type: 'object',
          properties: {
            query: { type: 'string', description: 'Search query' },
            fileType: { type: 'string', description: 'File type (typescript, react, etc.)' },
            taskType: { type: 'string', description: 'Task type (create, edit, review)' },
            maxResults: { type: 'number', description: 'Maximum results to return' }
          },
          required: ['query']
        }
      },
      {
        name: 'get_rules_for_file',
        description: 'Get rules applicable to a specific file and task',
        inputSchema: {
          type: 'object',
          properties: {
            filePath: { type: 'string', description: 'Path to file being worked on' },
            task: { type: 'string', description: 'Current task type' }
          },
          required: ['filePath', 'task']
        }
      }
    ]
  };
});

async function semanticSearch(
  query: string,
  options: { fileType?: string; taskType?: string; maxResults?: number }
): Promise<any[]> {
  // Implementation of semantic search over rules
  // Returns top-k most relevant rules
}

async function getRulesForFileAndTask(filePath: string, task: string): Promise<any[]> {
  // Determine rules based on file extension and task type
  // Return only applicable rules
}

// Start server
const transport = new StdioServerTransport();
server.connect(transport);
```

### Using MCP Rule Server from Claude Code

```
# Add MCP rule server to Claude Code
claude mcp add rules-server --command "node mcp-rules-server/dist/index.js"

# Use in conversation:
# You: "Add error handling to api/users.ts"
# Claude: (calls search_rules with query="error handling", fileType="typescript", taskType="edit")
# Returns relevant rules without loading all documentation
```

---

## Technique 9: RAG-Based Rule Retrieval

Implement RAG (Retrieval-Augmented Generation) for rules:

```typescript
// rag-rule-retriever.ts

interface RuleDocument {
  id: string;
  content: string;
  metadata: {
    title: string;
    category: string;
    tags: string[];
    filePattern: string[];
    priority: number;
  };
}

class RAGRuleRetriever {
  private rules: RuleDocument[] = [];
  private embeddingModel: any;
  private vectorIndex: any;

  constructor() {
    this.embeddingModel = new OpenAIEmbeddings();
    this.vectorIndex = new HNSWLib();
  }

  async addRules(rules: RuleDocument[]): Promise<void> {
    this.rules.push(...rules);
    
    const texts = rules.map(r => r.content);
    const embeddings = await this.embeddingModel.embedDocuments(texts);
    
    for (let i = 0; i < rules.length; i++) {
      await this.vectorIndex.addPoint(embeddings[i], rules[i].id);
    }
  }

  async retrieve(
    query: string,
    options: {
      fileContext?: string;
      taskType?: string;
      maxResults?: number;
      minRelevance?: number;
    } = {}
  ): Promise<RuleDocument[]> {
    const {
      fileContext,
      taskType,
      maxResults = 5,
      minRelevance = 0.7
    } = options;

    // Generate query embedding
    const queryEmbedding = await this.embeddingModel.embedQuery(query);

    // Search vector index
    const results = await this.vectorIndex.search(queryEmbedding, maxResults * 2);

    // Filter and rank
    let filtered = results
      .filter(r => r.score >= minRelevance)
      .map(r => this.rules.find(rule => rule.id === r.id))
      .filter((r): r is RuleDocument => r !== undefined);

    // Apply context filters
    if (fileContext) {
      filtered = filtered.filter(r =>
        r.metadata.filePattern.some(pattern =>
          this.matchesPattern(pattern, fileContext)
        )
      );
    }

    if (taskType) {
      filtered = filtered.filter(r =>
        r.metadata.tags.includes(taskType) ||
        r.metadata.category === taskType
      );
    }

    // Re-rank by priority
    filtered.sort((a, b) => b.metadata.priority - a.metadata.priority);

    return filtered.slice(0, maxResults);
  }

  private matchesPattern(pattern: string, text: string): boolean {
    const regex = new RegExp(
      '^' + pattern.replace(/\*\*/g, '.*').replace(/\*/g, '[^/]*') + '$'
    );
    return regex.test(text);
  }
}

// Usage
const retriever = new RAGRuleRetriever();

// Index all rules
await retriever.addRules([
  {
    id: 'ts-error-handling',
    content: tsErrorHandlingRules,
    metadata: {
      title: 'TypeScript Error Handling',
      category: 'error-handling',
      tags: ['typescript', 'error', 'backend'],
      filePattern: ['**/*.ts', '**/*.tsx'],
      priority: 10
    }
  },
  // ... more rules
]);

// Retrieve relevant rules for a task
const relevantRules = await retriever.retrieve(
  'Add error handling to the user API with proper Result types',
  {
    fileContext: 'packages/api/src/users.ts',
    taskType: 'edit',
    maxResults: 5
  }
);

console.log('Relevant rules:', relevantRules.map(r => r.metadata.title));
```

---

## Comparison: On-Demand Loading Techniques

| Technique | Tool Support | Pros | Cons |
|-----------|-------------|------|------|
| **Semantic References** | OpenCode | Simple, native | Manual rule identification |
| **opencode.json Modules** | OpenCode | Organized, glob patterns | Configuration complexity |
| **Tag-Based Filtering** | Custom | Flexible filtering | Requires metadata in rules |
| **Semantic Search** | Custom | AI-powered relevance | Setup complexity |
| **MCP Rule Server** | All MCP clients | Centralized, shareable | Server setup required |
| **RAG Retrieval** | Custom | High-quality retrieval | Vector DB required |
| **Codex Override** | Codex | Task-specific focus | Limited customization |

---

## Recommended Implementation Strategy

### Phase 1: Basic On-Demand Loading (Week 1)

1. Restructure AGENTS.md with `@reference` syntax
2. Tag rule files with metadata
3. Create task-specific rule files

### Phase 2: Context-Aware Loading (Week 2)

1. Implement file-type detection
2. Create rule loader based on file patterns
3. Add priority-based rule filtering

### Phase 3: Semantic Retrieval (Week 3)

1. Set up vector database for rules
2. Implement semantic search
3. Add RAG-based rule retrieval

### Phase 4: Production Optimization (Week 4)

1. Add MCP server for shared access
2. Implement caching layer
3. Add usage analytics

---

## Quick Start: Minimal Viable Solution

```markdown
# AGENTS.md (Minimal Viable On-Demand)

## Critical Rules (Always Apply)
- Security: Never commit secrets
- Type Safety: Use TypeScript strict mode
- Testing: All new code must have tests

## Load Rules on Demand

When working on TypeScript files, load:
→ @docs/typescript-style.md
→ @docs/error-handling.md

When working on React files, load:
→ @docs/react-patterns.md
→ @docs/hooks-rules.md

When working on API endpoints, load:
→ @docs/api-design.md
→ @docs/auth-rules.md

When writing tests, load:
→ @docs/testing-patterns.md
→ @docs/test-organization.md

## Rule Loading Protocol
1. Identify file type from extension
2. Load relevant @reference files using Read tool
3. Keep only active rules in context
4. Load new rules when switching file types

## Examples
Task: "Add validation to user form"
- Load @docs/react-patterns.md
- Load @docs/validation-rules.md
- Implement with loaded rules
```

---

## References and Further Reading

1. OpenCode Rules Documentation: https://opencode.ai/docs/rules/
2. MCP Protocol: https://modelcontextprotocol.io/docs/getting-started/intro
3. LangChain Knowledge Base: https://docs.langchain.com/oss/python/langchain/knowledge-base
4. "Improve Rule Retrieval and Reasoning" - arXiv:2505.10870
5. "General Agentic Memory (GAM)" - arXiv:2511.18423
