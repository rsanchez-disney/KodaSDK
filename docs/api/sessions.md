# Sessions API

Manage conversation sessions — create, resume, export, and query message history.

## `koda.sessions`

```typescript
interface SessionManager {
  create(options?: { name?: string; agentId?: string }): Promise<Session>;
  list(): Promise<Session[]>;
  get(id: string): Promise<Session | null>;
  messages(id: string): Promise<Message[]>;
  resume(id: string): Promise<void>;
  delete(id: string): Promise<void>;
  export(id: string, format: 'markdown' | 'json'): Promise<string>;
}
```

## Interfaces

### Session

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
```

### Message

```typescript
interface Message {
  id: string;
  sessionId: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  createdAt: string;
}
```

## Usage

### Create a Session

```typescript
const session = await koda.sessions.create({
  name: 'Sprint Review',
  agentId: 'sprint_manager_agent',
});
```

### Resume a Session

```typescript
await koda.sessions.resume(session.id);

// Subsequent chat calls use this session
for await (const event of koda.chat('Continue the analysis')) {
  // ...
}
```

### Get Message History

```typescript
const messages = await koda.sessions.messages(session.id);
messages.forEach(m => console.log(`[${m.role}] ${m.content}`));
```

### Export Session

```typescript
const markdown = await koda.sessions.export(session.id, 'markdown');
const json = await koda.sessions.export(session.id, 'json');
```

### Delete a Session

```typescript
await koda.sessions.delete(session.id);
```

!!! note "Session Persistence"
    Sessions are stored locally in SQLite. They persist across SDK restarts and can be resumed at any time.
