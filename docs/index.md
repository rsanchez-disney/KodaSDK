---
title: Koda SDK
---

# Koda SDK

**TypeScript SDK for building AI-powered applications on top of kiro-cli's Agent Communication Protocol (ACP).**

```bash
npm install @koda/sdk
```

---

<div class="grid cards" markdown>

-   :material-lightning-bolt:{ .lg .middle } **Streaming-native**

    ---

    All responses are `AsyncIterable<StreamEvent>`. Real-time text, tool calls, and delegation events out of the box.

-   :material-puzzle:{ .lg .middle } **Powers & Tools**

    ---

    Register app-side functions as MCP tools. Compose high-level Powers from agents + tools + context.

-   :material-desktop-classic:{ .lg .middle } **Electron-first**

    ---

    Runs in Node.js (Electron main). Optional IPC bridge exposes a safe subset to renderer.

-   :material-cog:{ .lg .middle } **Zero Config**

    ---

    Auto-discovers kiro-cli, reads MCP servers from `~/.kiro/`, resolves workspace from cwd.

-   :material-shield-lock:{ .lg .middle } **Secure Tokens**

    ---

    System keychain integration. Tokens never cross the IPC bridge to renderer.

-   :material-palette:{ .lg .middle } **React Components**

    ---

    Pre-built `ChatPanel`, `PromptBar`, `PowerButton`, and more — themed via CSS variables.

</div>

---

## Quick Example

```typescript
import { KodaSDK } from '@koda/sdk';

const koda = new KodaSDK({ appName: 'my-app' });

for await (const event of koda.chat('Analyze sprint health')) {
  if (event.type === 'text') process.stdout.write(event.content);
}
```

## Packages

| Package | Description |
|---------|-------------|
| `@koda/sdk` | Core: ACP client, agents, tools, sessions, tokens, powers |
| `@koda/sdk-electron` | Electron: preload bridge, IPC, secure token storage |
| `@koda/sdk-react` | React UI components: ChatPanel, PromptBar, PowerButton |

## Next Steps

- [Installation](getting-started/installation.md) — Set up your environment
- [Quickstart](getting-started/quickstart.md) — Build your first integration
- [API Reference](api/sdk.md) — Full SDK API docs

