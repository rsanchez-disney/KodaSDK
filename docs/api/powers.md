# Powers API

Powers are high-level capabilities composed from agents + tools + context. They represent what the SDK can *do* for an app.

## `koda.powers`

```typescript
interface PowerManager {
  list(): Promise<Power[]>;
  available(): Promise<Power[]>;
  invoke(id: string, params?: Record<string, unknown>): AsyncIterable<StreamEvent>;
  run(id: string, params?: Record<string, unknown>): Promise<PowerResult>;
  register(power: PowerDefinition): void;
}
```

## Interfaces

### Power

```typescript
interface Power {
  id: string;
  name: string;
  description: string;
  agent: string;
  requiredTools: string[];
  requiredTokens: string[];
  parameters: PowerParam[];
  available: boolean;       // All requirements met?
}
```

### PowerParam

```typescript
interface PowerParam {
  name: string;
  type: 'string' | 'number' | 'boolean' | 'object';
  description: string;
  required: boolean;
}
```

### PowerResult

```typescript
interface PowerResult {
  content: string;          // Final text response
  data?: unknown;           // Structured data (if power returns JSON)
  toolCalls: { name: string; result: unknown }[];
  duration: number;         // Execution time in ms
}
```

## Usage

### List Available Powers

```typescript
const powers = await koda.powers.available();
// Only returns powers where all requiredTools and requiredTokens are satisfied
```

### Invoke (Streaming)

```typescript
for await (const event of koda.powers.invoke('analyze-sprint', { teamId: 'bolt' })) {
  if (event.type === 'text') render(event.content);
}
```

### Run (Await Result)

```typescript
const result = await koda.powers.run('analyze-sprint', { teamId: 'lodging-sales' });
console.log(result.content);   // "Sprint is on track..."
console.log(result.duration);  // 3200 (ms)
```

### Register a Custom Power

```typescript
koda.powers.register({
  id: 'team-health-check',
  name: 'Team Health Check',
  description: 'Analyze sprint health for a team',
  agent: 'sprint_manager_agent',
  requiredTools: ['jira_search_issues'],
  requiredTokens: ['JIRA_PAT'],
  parameters: [
    { name: 'teamId', type: 'string', description: 'Team ID', required: true },
  ],
  promptTemplate: 'Analyze the current sprint health for team {{teamId}}. Check velocity, burndown, and blockers.',
});
```

### PowerDefinition

```typescript
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
```

!!! note "Availability Check"
    A power is `available: true` only when all `requiredTools` are provided by configured MCP servers **and** all `requiredTokens` exist in the token store.
