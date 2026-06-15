# Integration Guide

How to integrate Koda SDK into an existing Electron application.

## Step 1: Install

```bash
npm install @koda/sdk @koda/sdk-electron @koda/sdk-react
```

## Step 2: Initialize in Main Process

```typescript
// main.ts
import { app } from 'electron';
import { KodaSDK } from '@koda/sdk';

let koda: KodaSDK;

app.whenReady().then(() => {
  koda = new KodaSDK({
    appName: 'my-electron-app',
    workspace: '/path/to/workspace',
  });

  koda.on('error', (err) => console.error('Koda error:', err));
});

app.on('before-quit', () => koda?.disconnect());
```

## Step 3: Set Up Preload Bridge

```typescript
// preload.ts
import { createKodaBridge } from '@koda/sdk-electron';
createKodaBridge();
```

Configure your `BrowserWindow`:

```typescript
const win = new BrowserWindow({
  webPreferences: {
    preload: path.join(__dirname, 'preload.js'),
    contextIsolation: true,
    nodeIntegration: false,
  },
});
```

## Step 4: Use in Renderer

=== "React Components"

    ```tsx
    import { ChatPanel, PowerButton } from '@koda/sdk-react';

    function AIPanel() {
      return (
        <div>
          <PowerButton sdk={window.koda} powerId="analyze-sprint" params={{ teamId: 'bolt' }} />
          <ChatPanel sdk={window.koda} showToolCalls={true} />
        </div>
      );
    }
    ```

=== "Direct API"

    ```typescript
    const stream = window.koda.chat('What is our velocity trend?');
    stream.on('text', (chunk) => appendToDOM(chunk));
    stream.on('complete', ({ fullContent }) => renderFinal(fullContent));
    ```

## Step 5: Register App Tools

Expose app-specific data to agents:

```typescript
// main.ts — after SDK init
koda.tools.register({
  name: 'get_app_state',
  description: 'Returns current application state',
  parameters: {
    section: { type: 'string', description: 'State section to retrieve' },
  },
  handler: async ({ section }) => {
    return getState(section);
  },
});
```

## Step 6: Register Powers

Define high-level capabilities:

```typescript
koda.powers.register({
  id: 'daily-standup',
  name: 'Generate Standup',
  description: 'Generate daily standup from recent activity',
  agent: 'sprint_manager_agent',
  requiredTools: ['jira_search_issues'],
  requiredTokens: ['JIRA_PAT'],
  parameters: [
    { name: 'teamId', type: 'string', description: 'Team ID', required: true },
  ],
  promptTemplate: 'Generate a standup report for team {{teamId}} based on yesterday activity.',
});
```

!!! tip "Graceful Degradation"
    Check `koda.isConnected()` and power availability before rendering AI features. Show fallback UI when kiro-cli is unavailable.
