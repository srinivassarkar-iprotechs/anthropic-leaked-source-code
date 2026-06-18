# Claude Code - Complete Working Architecture

## Overview
This document outlines the complete working architecture of Claude Code, Anthropic's AI-powered coding agent. It's a sophisticated multi-layered system designed for autonomous code analysis, execution, and generation.

---

## Tech Stack

### Core Runtime
- **Language**: TypeScript/JavaScript (100%)
- **Runtime**: Bun (modern, fast JavaScript runtime)
- **Package Manager**: Bun

### API & AI Layer
- **AI SDK**: `@anthropic-ai/sdk` (Anthropic's official API client)
- **Model Context Protocol (MCP)**: `@modelcontextprotocol/sdk` - enables AI to interact with external tools and resources
- **API Style**: REST with streaming support (SSE, WebSocket)

### UI & Rendering
- **Framework**: React (interactive components)
- **CLI Rendering**: Ink (React for terminal UI)
- **Terminal Utilities**: 
  - `strip-ansi` (ANSI color code stripping)
  - Terminal-based spinner and progress indicators

### Data & Validation
- **Schema Validation**: Zod (TypeScript-first schema validation)
- **Utilities**: `lodash-es` (utility functions)

### File System & Process
- **File Operations**: Native Node.js fs module
- **Process Management**: Child process execution (Bash, Shell)
- **Path Handling**: Native Node.js path module

---

## Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    USER INTERFACES                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   CLI/REPL   │  │  Desktop App  │  │  Web Socket  │      │
│  │   (Ink)      │  │  (SSE Stream) │  │   Bridge     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  QUERY ENGINE LAYER                          │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  QueryEngine: Core orchestration & session lifecycle  │  │
│  │  - Message processing                                │  │
│  │  - Permission management                             │  │
│  │  - Cost tracking                                      │  │
│  │  - File history & attribution                         │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              AGENT ORCHESTRATION LAYER                       │
│  ┌───────────────┐  ┌──────────────┐  ┌──────────────────┐ │
│  │  Coordinator  │  │  Task Queue  │  │  Agent Manager   │ │
│  │  (Multi-turn) │  │              │  │  (Subagents)     │ │
│  └───────────────┘  └──────────────┘  └──────────────────┘ │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Commands Framework: /slash commands & shortcuts      │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  CONTEXT & STATE LAYER                       │
│  ┌─────────────────┐  ┌──────────────┐  ┌────────────────┐ │
│  │  Context Setup  │  │  AppState    │  │  Permissions   │ │
│  │                 │  │  Management  │  │  & Sandboxing  │ │
│  └─────────────────┘  └──────────────┘  └────────────────┘ │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  System Prompt Construction & Session Caching         │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  TOOLS EXECUTION LAYER                       │
│  ┌──────────────┐  ┌─────────────┐  ┌─────────────────────┐│
│  │   Bash Tool  │  │  FileEdit   │  │  WebSearch/WebFetch ││
│  └──────────────┘  └─────────────┘  └─────────────────────┘│
│  ┌──────────────┐  ┌─────────────┐  ┌─────────────────────┐│
│  │  FileRead    │  │   Glob      │  │    Grep/Search      │││
│  └──────────────┘  └─────────────┘  └─────────────────────┘│
│  ┌──────────────┐  ┌─────────────┐  ┌─────────────────────┐│
│  │  Notebook    │  │  Agent Tool │  │    MCP Proxy        │││
│  └──────────────┘  └─────────────┘  └─────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              SERVICES & INFRASTRUCTURE                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   API Layer  │  │    Logging   │  │   Session        │  │
│  │  (Claude SDK)│  │   & Metrics  │  │   Storage        │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   MCP Server │  │   Hooks      │  │  Error Handling  │  │
│  │   Manager    │  │   System     │  │  & Retry Logic   │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  FILE SYSTEM LAYER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  File Cache  │  │  History &   │  │  Scratch Pad     │  │
│  │  Management  │  │  Snapshots   │  │  & Memory        │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. **QueryEngine** (`QueryEngine.ts`)
**Responsibility**: Main orchestration engine for a conversation lifecycle

**Key Features**:
- Manages message state and conversation history
- Handles permission tracking and sandboxing
- Coordinates tool execution and result processing
- Tracks usage (tokens, costs)
- Manages file history and attribution
- Supports multi-turn conversations
- Generates and refines system prompts dynamically

**Key Methods**:
- `submitMessage()`: Process user input and run query loop
- `interrupt()`: Abort current operation
- `getMessages()`: Retrieve conversation history
- `getReadFileState()`: Access file cache state
- `setModel()`: Switch between AI models

**Data Flow**:
```
User Input → processUserInput → QueryEngine.submitMessage()
            ↓
Query Processing → Tool Execution → Response Generation
            ↓
Message Storage → Permission Tracking → Result Output
```

### 2. **Tool System** (`Tool.ts`, `tools/` directory)
**Responsibility**: Unified interface for all executable actions

**Architecture**:
```typescript
Tool<Input, Output, Progress>
├── Metadata
│   ├── name, aliases, searchHint
│   └── strict mode, deferral flags
├── I/O Schema
│   ├── inputSchema (Zod)
│   ├── inputJSONSchema (optional, for MCP)
│   └── outputSchema
├── Permission & Validation
│   ├── checkPermissions()
│   ├── validateInput()
│   └── preparePermissionMatcher()
├── Execution
│   ├── call() → ToolResult
│   ├── isConcurrencySafe()
│   └── interruptBehavior()
├── UI Rendering
│   ├── renderToolUseMessage()
│   ├── renderToolResultMessage()
│   └── renderToolUseProgressMessage()
└── Classification
    ├── isSearchOrReadCommand()
    ├── toAutoClassifierInput()
    └── isOpenWorld()
```

**Built-in Tools**:
- **Bash**: Execute shell commands
- **FileRead/FileWrite/FileEdit**: Filesystem operations
- **Glob/Grep**: File searching and pattern matching
- **WebSearch/WebFetch**: Internet information retrieval
- **Notebook**: Jupyter-style code execution (Python, Node.js)
- **AgentTool**: Spawn subagents for parallel work
- **MCP Proxy**: Interface to Model Context Protocol servers
- **SkillTool**: Load and execute skills from dynamic directories
- **SyntheticOutputTool**: Enforce structured JSON output

### 3. **Command System** (`commands.ts`)
**Responsibility**: User-facing slash commands for interaction

**Built-in Commands**:
- `/help` - Show available commands
- `/think` - Enable thinking mode
- `/retry` - Retry last operation
- `/undo` - Revert last message
- `/model` - Switch between models
- `/clear` - Clear conversation history
- `/skills` - List available skills
- `/budget` - Check remaining budget

### 4. **Context Management** (`context.ts`, `context/`)
**Responsibility**: Maintain execution context across operations

**Manages**:
- User context (working directory, permissions)
- System context (model capabilities, constraints)
- Tool permission context
- MCP resource availability
- Session state

### 5. **Task Management** (`Task.ts`, `tasks/`)
**Responsibility**: Handle background and concurrent work

**Task Types**:
```typescript
type TaskType = 
  | 'local_bash'           // Shell command execution
  | 'local_agent'          // Spawned agent process
  | 'remote_agent'         // Remote execution
  | 'in_process_teammate'  // Co-located parallel agent
  | 'local_workflow'       // Workflow orchestration
  | 'monitor_mcp'          // MCP server monitoring
  | 'dream'                // Speculative/preview execution
```

**Task States**: pending → running → (completed|failed|killed)

### 6. **Permission System** (`permissions/`)
**Responsibility**: Enforce security and user control

**Features**:
- Permission modes: `default`, `auto`, `coordinator`, `bypass`
- Permission rules: `alwaysAllow`, `alwaysDeny`, `alwaysAsk`
- Dynamic working directory management
- Tool-specific permission matchers
- Denial tracking and escalation
- Scratchpad sandboxing

### 7. **Session Management** (`bootstrap/`, `state/AppState.ts`)
**Responsibility**: Persistent state across conversations

**Manages**:
- Session IDs and lifecycle
- Conversation history
- File state cache
- Cost tracking
- Attribution tracking
- Plugin and skill state
- Theme and configuration

### 8. **API & Claude Integration** (`services/api/`)
**Responsibility**: Communication with Claude API

**Features**:
- Message streaming (SSE, WebSocket)
- Token usage tracking
- Error categorization and retry logic
- Cost calculation by model
- Latency measurement
- Adaptive retry strategies

**Supported Models**:
- `claude-opus` (most capable)
- `claude-sonnet` (balanced)
- `claude-haiku` (fast, small)

### 9. **MCP Server Management** (`services/mcp/`)
**Responsibility**: Model Context Protocol integration

**Features**:
- Server discovery and connection
- Resource listing and access
- Tool registration from MCP servers
- Error handling and recovery
- URL elicitation support

### 10. **Prompt Construction** (`utils/queryContext.ts`, `utils/messages/`)
**Responsibility**: Dynamic system prompt generation

**Components**:
- Default system prompt with tool documentation
- User context injection (working directory, permissions)
- System context (model info, capabilities)
- Memory mechanics (if enabled)
- Thinking configuration
- Plugin/skill descriptions

### 11. **Cost Tracking** (`cost-tracker.ts`)
**Responsibility**: Monitor API spending

**Features**:
- Per-model cost calculation
- Token counting
- Total cost aggregation
- Budget enforcement
- Cost reporting per turn

### 12. **History Management** (`history.ts`, `utils/fileHistory.ts`)
**Responsibility**: Track and revert file changes

**Features**:
- File snapshots at message boundaries
- Diff generation
- Rollback capabilities
- Attribution tracking

---

## Data Flow Diagrams

### Message Processing Flow
```
┌─────────────────┐
│  User Input     │
└────────┬────────┘
         ↓
┌─────────────────────────────────────┐
│  processUserInput()                 │
│  - Parse slash commands             │
│  - Extract attachments              │
│  - Apply inline processors          │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  QueryEngine.submitMessage()        │
│  - Append to messages array         │
│  - Record to transcript             │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  query() - Main Loop                │
│  - Fetch system prompt              │
│  - Call Claude API                  │
│  - Stream response                  │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  Tool Use Detection                 │
│  - Parse tool_use blocks            │
│  - Validate against schema          │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  Permission Check                   │
│  - canUseTool() hook                │
│  - Apply rules & matchers           │
│  - Prompt user if needed            │
└────────┬────────────────────────────┘
         ↓
     ┌───┴───┐
     │       │
  ┌──▼──┐  ┌─▼──┐
  │Allow│  │Deny│
  └──┬──┘  └─┬──┘
     │       └────→ Return error
     ↓
┌─────────────────────────────────────┐
│  Tool Execution                     │
│  - Run tool.call()                  │
│  - Capture output & errors          │
│  - Track progress                   │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  Result Processing                  │
│  - Serialize tool result            │
│  - Record to messages               │
│  - Update file cache                │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  Continue Query Loop                │
│  - Next assistant turn              │
│  - Or: finish & return result       │
└─────────────────────────────────────┘
```

### Tool Execution Flow
```
Tool Use Block (from Claude API)
        ↓
┌─────────────────────────────────────┐
│  1. Parse & Validate                │
│     - Extract tool name, ID, input  │
│     - Validate against schema       │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  2. Find Tool                       │
│     - toolMatchesName()             │
│     - Support aliases               │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  3. Pre-execution Hooks             │
│     - PreToolUse lifecycle          │
│     - Input transformation          │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  4. Permission Check                │
│     - validateInput()               │
│     - checkPermissions()            │
│     - canUseTool() callback         │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  5. Execute                         │
│     - tool.call()                   │
│     - Emit progress events          │
│     - Handle errors                 │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  6. Format Result                   │
│     - mapToolResultToToolResultBlockParam()
│     - Serialize output              │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  7. Post-execution Hooks            │
│     - PostToolUse lifecycle         │
│     - Result modification           │
└────────┬────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│  8. Update State                    │
│     - Record to messages            │
│     - Update file cache             │
│     - Track attribution             │
└────────┬────────────────────────────┘
         ↓
    Tool Result to Claude
```

---

## Key Features

### 1. **Multi-Model Support**
- Dynamic model switching via `/model` command
- Fallback model for reliability
- Model-specific prompt caching

### 2. **Agentic Loop**
- Unbounded tool calling
- Turn counting with `maxTurns` limit
- Budget enforcement with `maxBudgetUsd`
- Graceful error handling

### 3. **Streaming & Real-time**
- Server-sent events (SSE) for CLI
- WebSocket for hybrid modes
- Ink-based terminal UI
- Progressive result rendering

### 4. **Advanced Prompting**
- Dynamic system prompt construction
- Memory mechanics (CLAUDE.md)
- Extended thinking support
- Context injection

### 5. **Safety & Sandboxing**
- Permission-based tool gating
- Working directory restrictions
- Scratchpad isolation
- Dangerous rule filtering

### 6. **Extensibility**
- MCP (Model Context Protocol) integration
- Plugin system with dynamic loading
- Skill discovery in project directories
- Custom tool registration

### 7. **Observability**
- Cost tracking per message
- Usage analytics (tokens, duration)
- Error logging with context
- Performance profiling

### 8. **Session Persistence**
- Transcript storage with resumption
- Compact boundary compression
- File history snapshots
- Attribution metadata

---

## Directory Structure

```
anthropic-leaked-source-code/
├── QueryEngine.ts              # Core orchestration
├── Tool.ts                      # Tool interface & types
├── Task.ts                      # Task management
├── commands.ts                  # Slash command framework
├── context.ts                   # Context management
├── cost-tracker.ts              # Cost calculation
├── history.ts                   # File history
├── query.ts                     # Main query loop
├── setup.ts                     # Initialization
│
├── assistant/                   # Assistant interfaces
├── bootstrap/                   # Session bootstrap & state
├── bridge/                      # Transport bridges (SSE, WebSocket)
├── buddy/                       # AI companionship features
├── cli/                         # CLI entry points
├── commands/                    # Additional commands
├── components/                  # React/Ink components
├── constants/                   # Configuration constants
├── coordinator/                 # Multi-turn coordinator
├── entrypoints/                 # SDK & entry points
├── hooks/                       # Tool lifecycle hooks
├── ink/                         # Ink UI components
├── keybindings/                 # Keyboard shortcuts
├── memdir/                      # Memory directory (CLAUDE.md)
├── migrations/                  # Data migrations
├── moreright/                   # Additional utilities
├── native-ts/                   # Native bindings
├── outputStyles/                # Terminal styling
├── plugins/                     # Plugin system
├── remote/                      # Remote execution
├── schemas/                     # Data schemas
├── screens/                     # Screen components
├── server/                      # Server infrastructure
├── services/
│   ├── api/                     # Claude API client
│   ├── compact/                 # Message compaction
│   └── mcp/                     # MCP management
├── skills/                      # Skill loading
├── state/                       # State management
├── tools/                       # Tool implementations
│   ├── BashTool/
│   ├── FileEditTool/
│   ├── FileReadTool/
│   ├── WebSearchTool/
│   ├── NotebookTool/
│   ├── AgentTool/
│   ├── MCPProxyTool/
│   └── ...more tools
├── types/                       # TypeScript types
├── utils/
│   ├── fileStateCache.ts        # File cache management
│   ├── fileHistory.ts           # File snapshots
│   ├── queryContext.ts          # System prompt building
│   ├── permissions/             # Permission enforcement
│   ├── model/                   # Model utilities
│   ├── sessionStorage.ts        # Session I/O
│   ├── theme.ts                 # Theme management
│   ├── thinking.ts              # Extended thinking
│   └── ...more utilities
├── vim/                         # Vim keybindings
├── voice/                       # Voice interface
└── upstreamproxy/               # Proxy layer

```

---

## Integration Points

### External APIs
- **Anthropic Claude API**: Main LLM backend
- **Model Context Protocol (MCP)**: Tool integration
- **Web APIs**: WebSearch, WebFetch tools

### Internal Subsystems
- **Session Storage**: File-based transcript persistence
- **Plugin System**: Dynamic extension loading
- **Permission System**: User-defined access controls
- **Cost Tracking**: Real-time budget management

---

## Performance Optimizations

1. **Prompt Caching**: Reuse system prompts across messages
2. **File State Caching**: LRU cache for file reads
3. **Message Compaction**: Snip boundaries to compress history
4. **Lazy Loading**: Deferred tool definitions with ToolSearch
5. **Streaming**: SSE/WebSocket for real-time feedback
6. **Fast Mode**: Reduced analysis for simple queries

---

## Error Handling

### Error Categories
- **Validation Errors**: Schema validation failures
- **Permission Errors**: User denied tool access
- **API Errors**: Claude API communication failures
- **Tool Execution Errors**: Tool-specific failures
- **State Errors**: Conversation state inconsistencies

### Retry Strategy
- Exponential backoff for transient errors
- Categorized retry logic per error type
- Budget-aware retry limiting
- Structured output retry limits

---

## Security Model

### Tool Sandboxing
- Working directory restrictions
- Permission-based access control
- Scratchpad isolation
- Dangerous operation filtering

### Permission Modes
- **Default**: Ask user for each tool
- **Auto**: Automatic approval with classifier
- **Coordinator**: Pre-approval with async checks
- **Bypass**: Full access (careful use)

### Data Protection
- Session data encryption (at rest)
- Transcript redaction options
- File history snapshots
- Attribution metadata preservation

---

## Future Extensions

Potential enhancement areas:
1. **Parallelization**: Enhanced concurrent tool execution
2. **Collaborative Agents**: Multi-agent coordination
3. **Custom Models**: Fine-tuned model support
4. **Persistent Memory**: Long-term context management
5. **Audit Logging**: Comprehensive security audit trails
6. **Advanced Analytics**: Usage patterns and optimization recommendations

---

## Dependencies Summary

| Category | Package | Purpose |
|----------|---------|---------|
| **AI** | `@anthropic-ai/sdk` | Claude API client |
| **MCP** | `@modelcontextprotocol/sdk` | Tool protocol |
| **Validation** | `zod` | Schema validation |
| **UI** | `react`, `ink` | Component rendering |
| **Utilities** | `lodash-es`, `strip-ansi` | Helper functions |
| **Runtime** | `bun` | JavaScript runtime |

---

## Conclusion

Claude Code represents a comprehensive, production-grade architecture for autonomous AI-powered coding assistance. Its layered design, extensive tool system, and careful permission management enable secure, powerful agent capabilities while maintaining user control and transparency.
