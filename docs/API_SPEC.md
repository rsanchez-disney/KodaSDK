# API Specification

## KodaSDK

Main entry point. Instantiated once per app (in Electron main process).

```typescript
interface KodaSDKOptions {
  appName: string;                    // Identifier sent in ACP clientInfo
  workspace?: string;                 // Workspace ID (auto-detected if omitted)
  agent?: string;                     // Default agent (default: 'orchestrator')
  autoConnect?: boolean;              // Start kiro-cli immediately (default: true)
  mcpServers?: Record<string, any>;   // Override MCP config (default: reads from ~/.kiro/)
  trustAllTools?: boolean;            // Auto-approve tool calls (default: true)
}

class KodaSDK {
  constructor(options: KodaSDKOptions);

  // --- Chat ---
  chat(prompt: string, options?: ChatOptions): AsyncIterable<StreamEvent>;

  // --- Agents ---
  agent(id: string): AgentHandle;
  agents: AgentManager;

  // --- Tools ---
  tools: ToolRegistry;

  // --- Sessions ---
  sessions: SessionManager;

  // --- Context ---
  context: ContextManager;

  // --- Scoring ---
  score(prompt: string): Promise<ScoreResult>;

  // --- Lifecycle ---
  connect(): Promise<void>;
  disconnect(): void;
  isConnected(): boolean;

  // --- Events ---
  on(event: 'connected' | 'disconnected' | 'error', handler: Function): void;
  on(event: 'stream', handler: (event: StreamEvent) => void): void;
}
```

---

## Chat

```typescript
interface ChatOptions {
  sessionId?: string;      // Resume existing session
  agent?: string;          // Override default agent for this call
  context?: ContextEntry[];// Additional context to inject
}

// Usage:
const stream = koda.chat('Analyze sprint health', { agent: 'sprint_manager_agent' });
for await (const event of stream) {
  switch (event.type) {
    case 'text':       // Streaming text chunk
    case 'toolCall':   // Agent invoked a tool
    case 'delegation': // Sub-agent spawned
    case 'metadata':   // Context usage update
    case 'complete':   // Generation finished
    case 'error':      // Error occurred
  }
}
```

---

## StreamEvent

```typescript
type StreamEvent =
  | { type: 'text'; content: string }
  | { type: 'toolCall'; name: string; params: unknown; status: 'in_progress' | 'complete' }
  | { type: 'delegation'; subagents: { name: string; status: 'running' | 'complete' }[] }
  | { type: 'metadata'; contextUsage: number }
  | { type: 'complete'; stopReason: string; fullContent: string }
  | { type: 'error'; message: string };
```

---

## Agents

```typescript
interface AgentHandle {
  run(prompt: string, options?: ChatOptions): AsyncIterable<StreamEvent>;
  id: string;
}

interface AgentManager {
  list(): Promise<AgentInfo[]>;
  switch(id: string): Promise<void>;
  current(): string;
}

interface AgentInfo {
  id: string;
  name: string;
  description: string;
  tools: string[];
}
```

---

## Tools

Register app-side functions that agents can invoke via MCP.

```typescript
interface ToolDefinition {
  name: string;
  description: string;
  parameters?: Record<string, { type: string; description?: string }>;
  handler: (params: any) => Promise<unknown>;
}

interface ToolRegistry {
  register(tool: ToolDefinition): void;
  unregister(name: string): void;
  list(): ToolDefinition[];
}
```

When an agent calls a registered tool, the SDK intercepts the `session/request_permission` notification, invokes the handler, and returns the result to kiro-cli.

---

## Sessions

```typescript
interface Session {
  id: string;
  name: string;
  agentId: string;
  workspaceId: string;
  createdAt: string;
  updatedAt: string;
  messageCount: number;
}

interface SessionManager {
  create(options?: { name?: string; agentId?: string }): Promise<Session>;
  list(): Promise<Session[]>;
  get(id: string): Promise<Session | null>;
  messages(id: string): Promise<Message[]>;
  resume(id: string): Promise<void>;
  delete(id: string): Promise<void>;
  export(id: string, format: 'markdown' | 'json'): Promise<string>;
}

interface Message {
  id: string;
  sessionId: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  createdAt: string;
}
```

---

## Context

```typescript
interface ContextEntry {
  type: 'text' | 'file' | 'data';
  content: string;
  label?: string;
}

interface ContextManager {
  add(entry: ContextEntry): void;
  clear(): void;
  setWorkspace(id: string): Promise<void>;
  getWorkspace(): string | null;
}
```

---

## Scoring

```typescript
interface ScoreResult {
  total: number;              // 0-100 composite score
  estimatedTokens: number;
  dimensions: Record<string, { score: number; reasons: string[] }>;
}

// Usage:
const result = await koda.score('Explain the velocity trend');
console.log(result.total, result.estimatedTokens);
```

---

## Electron Bridge

Optional package for exposing SDK to renderer via contextBridge.

```typescript
// In preload.ts:
import { createKodaBridge } from '@koda/sdk-electron';
createKodaBridge(); // exposes window.koda

// In renderer:
const stream = window.koda.chat('Hello');
stream.on('text', (chunk) => { /* render */ });
stream.on('complete', ({ fullContent }) => { /* done */ });
```

---

## Error Handling

```typescript
class KodaError extends Error {
  code: 'NOT_CONNECTED' | 'CLI_NOT_FOUND' | 'SESSION_FAILED' | 'TIMEOUT' | 'PROCESS_CRASHED';
}

// SDK never throws unhandled. Errors surface as:
// 1. StreamEvent { type: 'error' } during chat
// 2. Promise rejection on direct calls (connect, sessions.create, etc.)
// 3. Event emitter: koda.on('error', handler)
```

---

## Configuration Resolution

The SDK resolves configuration in this order (first wins):

| Setting | Sources (priority order) |
|---------|--------------------------|
| kiro-cli path | `options.cliPath` → known paths → PATH |
| MCP servers | `options.mcpServers` → `~/.kiro/settings/mcp.json` |
| Agent | `options.agent` → workspace `default_agent` → `'orchestrator'` |
| Workspace | `options.workspace` → `~/.kiro/settings/kite.json` active → auto-detect from cwd |
| Scoring URL | `options.scoringUrl` → `http://localhost:8000/score` |

---

## Tokens & Credentials (Secure)

Secure credential management. Tokens are never exposed to renderer or stored in plaintext outside the system keychain.

```typescript
interface TokenManager {
  // Get a token by key (resolved from keychain → tokens.env → env var)
  get(key: string): Promise<string | null>;

  // Store a token securely (system keychain on macOS/Windows, encrypted file on Linux)
  set(key: string, value: string): Promise<void>;

  // Delete a stored token
  delete(key: string): Promise<void>;

  // List available token keys (not values)
  list(): Promise<string[]>;

  // Check if a token exists without reading it
  has(key: string): Promise<boolean>;
}

// Usage:
const pat = await koda.tokens.get('JIRA_PAT');
await koda.tokens.set('GITHUB_TOKEN', 'ghp_...');
const keys = await koda.tokens.list(); // ['JIRA_PAT', 'GITHUB_TOKEN', ...]
```

### Resolution order
1. System keychain (macOS Keychain, Windows Credential Manager)
2. `~/.kiro/tokens.env` (fallback, existing Koda convention)
3. Process environment variables

### Security guarantees
- Tokens never cross the IPC bridge to renderer
- Renderer can check `has(key)` but never read values
- All token access is logged (auditable)
- SDK passes tokens to MCP servers via env injection, not via app code

---

## MCP Servers

Discover and manage MCP server configurations.

```typescript
interface McpServer {
  name: string;
  command: string;
  args: string[];
  env: Record<string, string>;
  disabled: boolean;
  source: 'global' | 'workspace' | 'fork';
}

interface McpManager {
  // List all configured MCP servers (merged: global + workspace + fork)
  list(): Promise<McpServer[]>;

  // List servers available for current workspace
  available(): Promise<McpServer[]>;

  // Get tools exposed by a specific server
  tools(serverName: string): Promise<McpTool[]>;

  // Check health of a server
  health(serverName: string): Promise<{ status: 'running' | 'stopped' | 'error'; error?: string }>;
}

interface McpTool {
  name: string;
  description: string;
  server: string;
  inputSchema?: Record<string, unknown>;
}

// Usage:
const servers = await koda.mcp.list();
const jiraTools = await koda.mcp.tools('jira-jira');
```

---

## Discovery APIs

### Workspaces

```typescript
interface WorkspaceInfo {
  id: string;
  name: string;
  team: string;
  description: string;
  jiraPrefix: string;
  profiles: string[];
  defaultAgent: string;
  projects: { name: string; repo?: string }[];
  installed: boolean;
}

interface WorkspaceManager {
  list(): Promise<WorkspaceInfo[]>;
  active(): Promise<WorkspaceInfo | null>;
  switch(id: string): Promise<void>;
  projects(id: string): Promise<{ name: string; repo?: string; host?: string }[]>;
}

// Usage:
const workspaces = await koda.workspaces.list();
const active = await koda.workspaces.active();
await koda.workspaces.switch('payments-vertical');
```

### Agents

```typescript
interface AgentInfo {
  id: string;
  name: string;
  description: string;
  tools: string[];
  profile: string;
  welcomeMessage?: string;
}

interface AgentManager {
  list(): Promise<AgentInfo[]>;
  listByProfile(profile: string): Promise<AgentInfo[]>;
  get(id: string): Promise<AgentInfo | null>;
  switch(id: string): Promise<void>;
  current(): string;
}

// Usage:
const agents = await koda.agents.list();
const qaAgents = await koda.agents.listByProfile('qa');
await koda.agents.switch('defect_analyst_agent');
```

### Skills / Powers

Powers are high-level capabilities composed from agents + tools + context. They represent what the SDK can *do* for an app.

```typescript
interface Power {
  id: string;
  name: string;
  description: string;
  agent: string;            // Agent that executes this power
  requiredTools: string[];  // MCP tools needed
  requiredTokens: string[]; // Credentials needed (e.g., 'JIRA_PAT')
  parameters: PowerParam[];
  available: boolean;       // All requirements met?
}

interface PowerParam {
  name: string;
  type: 'string' | 'number' | 'boolean' | 'object';
  description: string;
  required: boolean;
}

interface PowerResult {
  content: string;          // Final text response
  data?: unknown;           // Structured data (if power returns JSON)
  toolCalls: { name: string; result: unknown }[];
  duration: number;
}

interface PowerManager {
  // List all registered powers
  list(): Promise<Power[]>;

  // List available powers (all requirements met)
  available(): Promise<Power[]>;

  // Invoke a power with parameters — returns streaming
  invoke(id: string, params?: Record<string, unknown>): AsyncIterable<StreamEvent>;

  // Invoke and wait for complete result
  run(id: string, params?: Record<string, unknown>): Promise<PowerResult>;

  // Register a custom power
  register(power: PowerDefinition): void;
}

interface PowerDefinition {
  id: string;
  name: string;
  description: string;
  agent: string;
  requiredTools?: string[];
  requiredTokens?: string[];
  parameters?: PowerParam[];
  promptTemplate: string;   // Template with {{param}} placeholders
}

// Usage:
// List what the SDK can do
const powers = await koda.powers.available();
// → [{ id: 'analyze-sprint', name: 'Analyze Sprint', available: true, ... }]

// Invoke a power
const result = await koda.powers.run('analyze-sprint', { teamId: 'lodging-sales' });
console.log(result.content); // "Sprint is on track..."

// Stream a power
for await (const event of koda.powers.invoke('explain-burndown', { teamId: 'bolt' })) {
  if (event.type === 'text') render(event.content);
}

// Register app-specific power
koda.powers.register({
  id: 'dcc-health-check',
  name: 'Team Health Check',
  description: 'Analyze sprint health for a team',
  agent: 'sprint_manager_agent',
  requiredTools: ['jira_search_issues'],
  requiredTokens: ['JIRA_PAT'],
  parameters: [{ name: 'teamId', type: 'string', description: 'Team ID', required: true }],
  promptTemplate: 'Analyze the current sprint health for team {{teamId}}. Check velocity, burndown, and blockers.',
});
```

---

## UI Components (`@koda/sdk-react`)

Pre-built React components for common AI interaction patterns. Themed to match the host app via CSS variables.

```typescript
import { ChatPanel, PromptBar, ResponseBubble, PowerButton, AgentPicker } from '@koda/sdk-react';
```

### ChatPanel

Full chat interface — prompt input, streaming responses, tool indicators.

```tsx
<ChatPanel
  sdk={koda}
  sessionId="sprint-review"
  agent="orchestrator"
  placeholder="Ask about your sprint..."
  maxHeight={400}
  showToolCalls={true}
  onMessage={(msg) => console.log(msg)}
/>
```

### PromptBar

Standalone prompt input with send button, slash commands, and streaming state.

```tsx
<PromptBar
  sdk={koda}
  onSubmit={(prompt) => { /* handle */ }}
  placeholder="Ask a question..."
  enableSlashCommands={true}
  enableAtMentions={true}
/>
```

### ResponseBubble

Render a single AI response with markdown, code blocks, and copy button.

```tsx
<ResponseBubble content={message.content} streaming={isStreaming} />
```

### PowerButton

One-click button that invokes a registered power and shows result inline.

```tsx
<PowerButton
  sdk={koda}
  powerId="analyze-sprint"
  params={{ teamId: 'lodging-sales' }}
  label="Analyze Sprint"
  variant="compact"  // 'compact' | 'full' | 'icon'
/>
```

### AgentPicker

Dropdown/modal for switching agents.

```tsx
<AgentPicker
  sdk={koda}
  current={currentAgent}
  onSwitch={(id) => setAgent(id)}
  filter={{ profile: 'dev' }}
/>
```

### Theming

Components inherit styles via CSS variables. Host app sets the theme:

```css
:root {
  --koda-bg: var(--app-surface);
  --koda-text: var(--app-text);
  --koda-accent: var(--app-primary);
  --koda-border: var(--app-border);
  --koda-code-bg: var(--app-code-bg);
  --koda-radius: 8px;
  --koda-font-mono: 'JetBrains Mono', monospace;
}
```

---

## Package Structure (updated)

```
@koda/sdk            — Core: ACP client, agents, tools, sessions, tokens, powers
@koda/sdk-electron   — Electron: preload bridge, IPC, secure token storage
@koda/sdk-react      — React UI components: ChatPanel, PromptBar, PowerButton, etc.
```
