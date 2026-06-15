# Powers Guide

Powers are high-level capabilities that compose agents, tools, and context into reusable actions. Think of them as "what the SDK can do" for your app.

## Concept

A Power wraps:

- An **agent** to execute the task
- **Required tools** (MCP tools the agent needs)
- **Required tokens** (credentials for those tools)
- **Parameters** (user-provided inputs)
- A **prompt template** (parameterized instruction)

## Creating a Power

```typescript
koda.powers.register({
  id: 'sprint-analysis',
  name: 'Sprint Analysis',
  description: 'Analyze current sprint health including velocity, burndown, and blockers',
  agent: 'sprint_manager_agent',
  requiredTools: ['jira_search_issues', 'jira_get_sprint'],
  requiredTokens: ['JIRA_PAT'],
  parameters: [
    { name: 'teamId', type: 'string', description: 'Team identifier', required: true },
    { name: 'includeBlockers', type: 'boolean', description: 'Include blocker analysis', required: false },
  ],
  promptTemplate: 'Analyze the current sprint health for team {{teamId}}.{{#includeBlockers}} Include detailed blocker analysis.{{/includeBlockers}}',
});
```

## Invoking Powers

### Streaming (for UI)

```typescript
for await (const event of koda.powers.invoke('sprint-analysis', { teamId: 'bolt' })) {
  if (event.type === 'text') render(event.content);
  if (event.type === 'toolCall') showToolIndicator(event.name);
}
```

### Await Result (for background tasks)

```typescript
const result = await koda.powers.run('sprint-analysis', {
  teamId: 'lodging-sales',
  includeBlockers: true,
});

console.log(result.content);    // Full text response
console.log(result.data);       // Structured data (if any)
console.log(result.duration);   // Execution time in ms
```

## Availability Checks

Powers track whether their requirements are met:

```typescript
const available = await koda.powers.available();
// Only returns powers where ALL of:
// - requiredTools are provided by configured MCP servers
// - requiredTokens exist in the token store

const all = await koda.powers.list();
all.forEach(p => {
  console.log(`${p.name}: ${p.available ? '✓' : '✗ missing requirements'}`);
});
```

## UI Integration

Use `PowerButton` for one-click invocation:

```tsx
<PowerButton
  sdk={koda}
  powerId="sprint-analysis"
  params={{ teamId: 'bolt' }}
  label="Analyze Sprint"
  variant="full"
/>
```

!!! note "Template Syntax"
    Prompt templates use `{{param}}` for simple interpolation. Boolean parameters support `{{#param}}...{{/param}}` conditional blocks.
