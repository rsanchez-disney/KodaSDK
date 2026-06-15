# MCP API

Discover and inspect MCP (Model Context Protocol) server configurations.

## `koda.mcp`

```typescript
interface McpManager {
  list(): Promise<McpServer[]>;
  available(): Promise<McpServer[]>;
  tools(serverName: string): Promise<McpTool[]>;
  health(serverName: string): Promise<{ status: 'running' | 'stopped' | 'error'; error?: string }>;
}
```

## Interfaces

### McpServer

```typescript
interface McpServer {
  name: string;
  command: string;
  args: string[];
  env: Record<string, string>;
  disabled: boolean;
  source: 'global' | 'workspace' | 'fork';
}
```

### McpTool

```typescript
interface McpTool {
  name: string;
  description: string;
  server: string;
  inputSchema?: Record<string, unknown>;
}
```

## Usage

### List All MCP Servers

```typescript
const servers = await koda.mcp.list();
servers.forEach(s => console.log(`${s.name} [${s.source}] ${s.disabled ? '(disabled)' : ''}`));
```

### List Available Servers (Current Workspace)

```typescript
const available = await koda.mcp.available();
```

### Inspect Tools from a Server

```typescript
const tools = await koda.mcp.tools('jira-jira');
tools.forEach(t => console.log(`${t.name}: ${t.description}`));
```

### Check Server Health

```typescript
const health = await koda.mcp.health('jira-jira');
if (health.status === 'error') {
  console.error(`Server error: ${health.error}`);
}
```

## Configuration Sources

MCP servers are merged from three sources (later overrides earlier):

| Source | Location | Scope |
|--------|----------|-------|
| `global` | `~/.kiro/settings/mcp.json` | All workspaces |
| `workspace` | Workspace config | Current workspace |
| `fork` | `options.mcpServers` | SDK instance override |

!!! note "Server Lifecycle"
    MCP servers are managed by kiro-cli. The SDK reads their configuration and inspects their tools, but does not start/stop servers directly.
