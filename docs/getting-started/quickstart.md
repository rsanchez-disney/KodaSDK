# Quickstart

Build a working AI chat in under 5 minutes.

## 1. Initialize the SDK

```typescript
import { KodaSDK } from '@koda/sdk';

const koda = new KodaSDK({
  appName: 'my-app',
  agent: 'orchestrator',    // default agent
  autoConnect: true,        // connects immediately
  trustAllTools: true,      // auto-approve tool calls
});
```

## 2. Send a Chat Message

```typescript
for await (const event of koda.chat('What tickets are blocked?')) {
  switch (event.type) {
    case 'text':
      process.stdout.write(event.content);
      break;
    case 'toolCall':
      console.log(`Tool: ${event.name} [${event.status}]`);
      break;
    case 'complete':
      console.log('\n---Done---');
      break;
  }
}
```

## 3. Register a Custom Tool

```typescript
koda.tools.register({
  name: 'get_team_velocity',
  description: 'Returns velocity for a team',
  parameters: {
    teamId: { type: 'string', description: 'Team identifier' },
  },
  handler: async (params) => {
    return { velocity: 42, sprint: 'Sprint 23' };
  },
});
```

## 4. Use a Power

```typescript
const result = await koda.powers.run('analyze-sprint', {
  teamId: 'lodging-sales',
});
console.log(result.content);
```

## 5. Clean Up

```typescript
koda.disconnect();
```

!!! tip "Connection Lifecycle"
    With `autoConnect: true` (default), the SDK connects on construction. Call `disconnect()` when your app exits to clean up the kiro-cli subprocess.

## Full Example

```typescript
import { KodaSDK } from '@koda/sdk';

async function main() {
  const koda = new KodaSDK({ appName: 'quickstart-demo' });
  await koda.connect();

  // Register a tool
  koda.tools.register({
    name: 'lookup_user',
    description: 'Look up user by ID',
    parameters: { userId: { type: 'string' } },
    handler: async ({ userId }) => ({ name: 'Alice', role: 'PM' }),
  });

  // Chat with streaming
  for await (const event of koda.chat('Who is user-123?')) {
    if (event.type === 'text') process.stdout.write(event.content);
  }

  koda.disconnect();
}

main();
```
