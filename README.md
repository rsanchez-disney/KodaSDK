# Koda SDK

A TypeScript SDK for integrating AI agent capabilities into Electron desktop apps. Provides a unified abstraction over [kiro-cli](https://github.disney.com/SANCR225/steer-runtime)'s ACP protocol — process lifecycle, streaming, agents, tools, sessions, and workspace context.

## Why

Every Electron app in the Steer ecosystem (Kite, DCC, Mouseketool) needs to communicate with kiro-cli for AI features. Without a shared SDK, each app re-implements:

- kiro-cli subprocess spawning and crash recovery
- ACP JSON-RPC protocol (initialize, session/new, session/prompt)
- Streaming response assembly from chunked notifications
- Agent switching and delegation
- MCP tool routing and permission handling
- Workspace context synchronization
- Session persistence

Koda SDK extracts these into a single, tested package.

## Quick Start

```typescript
import { KodaSDK } from '@koda/sdk';

const koda = new KodaSDK({ appName: 'my-app' });

// Stream a response
const stream = koda.chat('Analyze this sprint');
for await (const event of stream) {
  if (event.type === 'text') process.stdout.write(event.content);
  if (event.type === 'toolCall') console.log('Tool:', event.name);
}

// Use a specific agent
await koda.agent('defect_analyst_agent').run('Why did velocity drop?');

// Invoke a power (high-level capability)
const result = await koda.powers.run('analyze-sprint', { teamId: 'bolt' });

// Discover what's available
const powers = await koda.powers.available();
const agents = await koda.agents.list();
const servers = await koda.mcp.list();

// Expose app functions as tools
koda.tools.register({
  name: 'get_burndown',
  description: 'Fetch burndown data for a team',
  handler: async ({ teamId }) => fetchBurndown(teamId),
});

// Secure token access (never exposed to renderer)
const hasPat = await koda.tokens.has('JIRA_PAT');
```

### UI Components

```tsx
import { ChatPanel, PowerButton } from '@koda/sdk-react';

// Drop-in chat panel
<ChatPanel sdk={koda} placeholder="Ask about your sprint..." />

// One-click AI action
<PowerButton sdk={koda} powerId="analyze-sprint" params={{ teamId }} label="Analyze" />
```

## Packages

| Package | Description |
|---------|-------------|
| `@koda/sdk` | Core — ACP client, agents, tools, powers, tokens, sessions, MCP |
| `@koda/sdk-electron` | Electron — preload bridge, secure keychain storage, IPC |
| `@koda/sdk-react` | UI components — ChatPanel, PromptBar, PowerButton, AgentPicker |

## Consumers

- **Kite** — AI desktop chat (primary, extracted from)
- **DCC** — Sprint insights, anomaly detection
- **Mouseketool** — Config generation assistance

## Documentation

📖 **[Full Documentation](https://rsanchez-disney.github.io/KodaSDK/)**

- [Getting Started](https://rsanchez-disney.github.io/KodaSDK/getting-started/quickstart/)
- [API Reference](https://rsanchez-disney.github.io/KodaSDK/api/sdk/)
- [Architecture](https://rsanchez-disney.github.io/KodaSDK/architecture/overview/)
- [Guides](https://rsanchez-disney.github.io/KodaSDK/guides/integration/)

## Status

🚧 **Pre-alpha** — Extracting from Kite's production ACP layer.
