# Installation

## Prerequisites

- Node.js 18+
- kiro-cli installed and authenticated (`kiro auth login`)
- For Electron apps: Electron 25+

## Install Core SDK

```bash
npm install @koda/sdk
```

## Optional Packages

=== "Electron Bridge"

    ```bash
    npm install @koda/sdk-electron
    ```

=== "React Components"

    ```bash
    npm install @koda/sdk-react
    ```

=== "All Packages"

    ```bash
    npm install @koda/sdk @koda/sdk-electron @koda/sdk-react
    ```

## Verify kiro-cli

The SDK auto-discovers kiro-cli. Confirm it's available:

```bash
kiro --version
```

!!! note "CLI Resolution Order"
    The SDK searches for kiro-cli in this order:

    1. `options.cliPath` (explicit override)
    2. Known platform paths (`/usr/local/bin/kiro`, `~/.kiro/bin/kiro`)
    3. System `PATH`

## MCP Server Configuration

MCP servers are read from `~/.kiro/settings/mcp.json` by default. You can override at SDK initialization:

```typescript
const koda = new KodaSDK({
  appName: 'my-app',
  mcpServers: {
    'jira-jira': { command: 'mcp-jira', args: ['--host', 'jira.example.com'] },
  },
});
```

## Next Steps

- [Quickstart](quickstart.md) — Build your first integration
- [Electron Setup](electron-setup.md) — Configure for Electron apps
