# Chat API

Send prompts and receive streaming responses from agents.

## `koda.chat(prompt, options?)`

Returns an `AsyncIterable<StreamEvent>` — consume with `for await`.

```typescript
const stream = koda.chat('Analyze sprint health', {
  agent: 'sprint_manager_agent',
  sessionId: 'existing-session-id',
});

for await (const event of stream) {
  switch (event.type) {
    case 'text':       // streaming text chunk
    case 'toolCall':   // agent invoked a tool
    case 'delegation': // sub-agent spawned
    case 'metadata':   // context usage update
    case 'complete':   // generation finished
    case 'error':      // error occurred
  }
}
```

## ChatOptions

```typescript
interface ChatOptions {
  sessionId?: string;       // Resume existing session
  agent?: string;           // Override default agent for this call
  context?: ContextEntry[]; // Additional context to inject
}
```

| Property | Type | Description |
|----------|------|-------------|
| `sessionId` | `string` | Resume an existing conversation |
| `agent` | `string` | Override the default agent |
| `context` | `ContextEntry[]` | Inject additional context entries |

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

### Event Types

| Type | Description | Key Fields |
|------|-------------|------------|
| `text` | Streaming text token | `content` |
| `toolCall` | Agent called an MCP tool | `name`, `params`, `status` |
| `delegation` | Sub-agent spawned/completed | `subagents[]` |
| `metadata` | Context window usage update | `contextUsage` (0–100) |
| `complete` | Generation finished | `stopReason`, `fullContent` |
| `error` | Error during generation | `message` |

## Context Injection

Provide additional context with each message:

```typescript
for await (const event of koda.chat('Summarize this', {
  context: [
    { type: 'text', content: 'Sprint velocity is 42 points', label: 'velocity' },
    { type: 'file', content: fileContents, label: 'report.csv' },
  ],
})) {
  // ...
}
```

## ContextEntry

```typescript
interface ContextEntry {
  type: 'text' | 'file' | 'data';
  content: string;
  label?: string;
}
```

!!! tip "Session Continuity"
    Pass `sessionId` to continue a conversation. The agent retains memory of prior messages in that session.
