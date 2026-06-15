# KodaSDK

Main entry point. Instantiated once per app (typically in Electron main process).

## Constructor

```typescript
import { KodaSDK } from '@koda/sdk';

const koda = new KodaSDK(options: KodaSDKOptions);
```

### KodaSDKOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `appName` | `string` | **required** | Identifier sent in ACP `clientInfo` |
| `workspace` | `string` | auto-detect | Workspace ID or path |
| `agent` | `string` | `'orchestrator'` | Default agent ID |
| `autoConnect` | `boolean` | `true` | Start kiro-cli immediately |
| `mcpServers` | `Record<string, any>` | reads `~/.kiro/` | Override MCP config |
| `trustAllTools` | `boolean` | `true` | Auto-approve tool calls |

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `agents` | [`AgentManager`](agents.md) | List, switch, query agents |
| `tools` | [`ToolRegistry`](tools.md) | Register app-side functions |
| `sessions` | [`SessionManager`](sessions.md) | Create/resume/export sessions |
| `tokens` | [`TokenManager`](tokens.md) | Secure credential management |
| `mcp` | [`McpManager`](mcp.md) | MCP server discovery |
| `workspaces` | [`WorkspaceManager`](workspaces.md) | Workspace discovery |
| `powers` | [`PowerManager`](powers.md) | High-level capabilities |

## Methods

### `connect()`

Spawns kiro-cli and establishes an ACP session.

```typescript
await koda.connect();
```

!!! note
    Called automatically if `autoConnect: true` (default). Safe to call multiple times — no-ops if already connected.

### `disconnect()`

Terminates the kiro-cli subprocess and cleans up resources.

```typescript
koda.disconnect();
```

### `isConnected()`

```typescript
if (koda.isConnected()) { /* ready */ }
```

### `chat(prompt, options?)`

Sends a message and returns a streaming response. See [Chat API](chat.md).

```typescript
for await (const event of koda.chat('Hello', { agent: 'qa_agent' })) {
  // handle events
}
```

### `score(prompt)`

Evaluates prompt quality. See [Scoring API](scoring.md).

```typescript
const result = await koda.score('Explain velocity trend');
```

### `on(event, handler)`

Subscribe to lifecycle events.

```typescript
koda.on('connected', () => console.log('Ready'));
koda.on('disconnected', () => console.log('CLI exited'));
koda.on('error', (err) => console.error(err));
```

## Configuration Resolution

| Setting | Sources (priority order) |
|---------|--------------------------|
| kiro-cli path | `options.cliPath` → known paths → `PATH` |
| MCP servers | `options.mcpServers` → `~/.kiro/settings/mcp.json` |
| Agent | `options.agent` → workspace `default_agent` → `'orchestrator'` |
| Workspace | `options.workspace` → `~/.kiro/settings/kite.json` → auto-detect from cwd |

## Error Handling

```typescript
class KodaError extends Error {
  code: 'NOT_CONNECTED' | 'CLI_NOT_FOUND' | 'SESSION_FAILED' | 'TIMEOUT' | 'PROCESS_CRASHED';
}
```

Errors surface through:

1. `StreamEvent { type: 'error' }` during chat
2. Promise rejection on direct calls (`connect`, `sessions.create`, etc.)
3. Event emitter: `koda.on('error', handler)`
