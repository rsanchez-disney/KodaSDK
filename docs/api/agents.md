# Agents API

Discover, switch, and invoke agents.

## `koda.agents`

```typescript
interface AgentManager {
  list(): Promise<AgentInfo[]>;
  listByProfile(profile: string): Promise<AgentInfo[]>;
  get(id: string): Promise<AgentInfo | null>;
  switch(id: string): Promise<void>;
  current(): string;
}
```

## AgentInfo

```typescript
interface AgentInfo {
  id: string;
  name: string;
  description: string;
  tools: string[];
  profile: string;
  welcomeMessage?: string;
}
```

## Usage

### List All Agents

```typescript
const agents = await koda.agents.list();
agents.forEach(a => console.log(`${a.id}: ${a.description}`));
```

### Filter by Profile

```typescript
const qaAgents = await koda.agents.listByProfile('qa');
const devAgents = await koda.agents.listByProfile('dev');
```

### Switch Active Agent

```typescript
await koda.agents.switch('defect_analyst_agent');
console.log(koda.agents.current()); // 'defect_analyst_agent'
```

### Get Agent Details

```typescript
const agent = await koda.agents.get('sprint_manager_agent');
if (agent) {
  console.log(agent.tools); // ['jira_search_issues', 'jira_get_sprint', ...]
}
```

### Invoke Specific Agent (One-shot)

Use `ChatOptions.agent` to direct a single message to a specific agent without switching:

```typescript
for await (const event of koda.chat('Find regressions', { agent: 'qa_agent' })) {
  if (event.type === 'text') process.stdout.write(event.content);
}
```

## Delegation

When an agent delegates to sub-agents, you receive `delegation` events:

```typescript
for await (const event of koda.chat('Full sprint analysis')) {
  if (event.type === 'delegation') {
    event.subagents.forEach(sa => {
      console.log(`${sa.name}: ${sa.status}`);
    });
  }
}
```
