# Architecture

## Overview

Koda SDK provides a TypeScript abstraction over kiro-cli's ACP (Agent Communication Protocol). It manages the subprocess lifecycle, speaks JSON-RPC 2.0 over stdio, and exposes a high-level async/streaming API to consumer apps.

```
┌─────────────────────────────────────────────┐
│  Consumer App (Electron main process)        │
│  ┌─────────────────────────────────────────┐ │
│  │              @koda/sdk                   │ │
│  │  ┌──────────┐  ┌────────┐  ┌────────┐  │ │
│  │  │AcpClient │  │AgentMgr│  │ToolReg │  │ │
│  │  │          │  │        │  │        │  │ │
│  │  │•connect  │  │•switch │  │•register│  │ │
│  │  │•request  │  │•list   │  │•handle │  │ │
│  │  │•stream   │  │•delegate│  │•expose │  │ │
│  │  └────┬─────┘  └────────┘  └────────┘  │ │
│  │       │                                  │ │
│  │  ┌────┴──────────────────────────────┐   │ │
│  │  │         ProcessPool               │   │ │
│  │  │  •spawn kiro-cli acp              │   │ │
│  │  │  •crash recovery / auto-restart   │   │ │
│  │  │  •per-workspace isolation         │   │ │
│  │  └────┬──────────────────────────────┘   │ │
│  └───────┼──────────────────────────────────┘ │
│          │ stdio (JSON-RPC 2.0, newline-delimited)
│  ┌───────┴──────────────────────────────────┐ │
│  │           kiro-cli acp                    │ │
│  │  •Agent runtime  •MCP tools  •LLM calls  │ │
│  └───────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

## Extracted From

The SDK is extracted from Kite's `packages/main/src/acp-bridge.ts` (385 lines), which handles:

| Responsibility | Kite Implementation | SDK Module |
|---|---|---|
| CLI discovery | `resolveCliPath()` — scans 10+ candidate paths | `core/cli-resolver` |
| Process spawn | `spawn(cliPath, ['acp', '--trust-all-tools', '--agent', id])` | `core/process-pool` |
| JSON-RPC framing | Line-buffered stdout parsing, `send()` via stdin | `core/acp-client` |
| Initialize handshake | `{ method: 'initialize', params: { protocolVersion, clientInfo } }` | `core/acp-client` |
| Session creation | `session/new` with cwd, mcpServers, agentId | `core/acp-client` |
| Prompt sending | `session/prompt` with text array, context injection | `agents/prompt` |
| Stream routing | `routeAcpMessage()` — notification dispatcher | `streaming/router` |
| Chunk assembly | `session/update` → `agent_message_chunk` accumulation | `streaming/assembler` |
| Tool calls | `tool_call` / `tool_call_update` notifications | `tools/handler` |
| Tool permissions | Auto-approve `session/request_permission` | `tools/permissions` |
| Agent switching | Kill session, `session/new` with new agentId | `agents/switcher` |
| Delegation detection | Parse `Spawning agent` tool calls, `_kiro.dev/subagent/list_update` | `agents/delegation` |
| MCP loading | Read `~/.kiro/settings/mcp.json`, filter disabled | `core/mcp-loader` |
| Crash recovery | `process.on('exit')` → emit disconnected event | `core/process-pool` |
| Context metadata | `_kiro.dev/metadata` → contextUsagePercentage | `streaming/metadata` |

## ACP Protocol Summary

ACP is JSON-RPC 2.0 over stdio (newline-delimited JSON).

### Lifecycle

```
Client                              kiro-cli
  │                                    │
  │─── initialize ──────────────────►  │
  │◄── result: { capabilities } ─────  │
  │                                    │
  │─── session/new { cwd, agent } ──►  │
  │◄── result: { sessionId } ────────  │
  │                                    │
  │─── session/prompt { text } ──────► │
  │◄── notification: session/update ── │  (streaming chunks)
  │◄── notification: session/update ── │
  │◄── notification: tool_call ─────── │
  │◄── notification: _kiro.dev/meta ── │
  │◄── result: { stopReason } ──────── │  (completion)
  │                                    │
```

### Key Methods

| Method | Direction | Purpose |
|--------|-----------|---------|
| `initialize` | Client → CLI | Handshake, declare capabilities |
| `session/new` | Client → CLI | Create conversation session |
| `session/prompt` | Client → CLI | Send user message |
| `session/cancel` | Client → CLI | Abort current generation |
| `session/request_permission` | CLI → Client | Tool permission request |
| `session/permission_response` | Client → CLI | Approve/deny tool use |

### Key Notifications (CLI → Client)

| Method | Purpose |
|--------|---------|
| `session/update` (agent_message_chunk) | Streaming text token |
| `session/update` (tool_call) | Tool invocation started |
| `session/update` (tool_call_update) | Tool progress |
| `_kiro.dev/metadata` | Context usage percentage |
| `_kiro.dev/subagent/list_update` | Delegation status |

## Module Design

```
@koda/sdk/
├── core/
│   ├── acp-client.ts      — JSON-RPC send/receive, request tracking
│   ├── cli-resolver.ts    — Find kiro-cli binary (cross-platform)
│   ├── process-pool.ts    — Spawn, monitor, restart, per-workspace
│   └── mcp-loader.ts      — Load MCP server configs from ~/.kiro/
├── agents/
│   ├── index.ts           — Agent API (run, list, switch)
│   ├── delegation.ts      — Detect and track sub-agent spawning
│   └── prompt.ts          — Context injection, identity, history
├── streaming/
│   ├── router.ts          — Notification dispatcher
│   ├── assembler.ts       — Chunk → complete message assembly
│   └── async-iterator.ts  — AsyncIterator interface for consumers
├── tools/
│   ├── registry.ts        — Register app functions as MCP tools
│   ├── permissions.ts     — Auto-approve or prompt-based approval
│   └── handler.ts         — Route tool calls to registered handlers
├── sessions/
│   ├── store.ts           — SQLite persistence (messages, metadata)
│   └── export.ts          — Export as markdown, JSON
├── context/
│   ├── workspace.ts       — Workspace sync, context injection
│   └── identity.ts        — User identity resolution
├── scoring/
│   └── index.ts           — Prompt scoring (POST /score)
└── electron/
    ├── preload.ts         — IPC bridge for renderer access
    └── window.ts          — Main window event forwarding
```

## Design Principles

1. **Electron-first, framework-agnostic** — Core runs in Node.js (Electron main). Renderer bridge is optional.
2. **One process per workspace** — Isolation between workspaces. Pool manages lifecycle.
3. **Streaming-native** — All responses are `AsyncIterable<StreamEvent>`. No callback hell.
4. **Zero config** — Auto-discovers kiro-cli, reads MCP from ~/.kiro/, resolves workspace from cwd.
5. **Fail gracefully** — If kiro-cli isn't installed or crashes, SDK emits events, never throws unhandled.
6. **Bidirectional tools** — Apps register functions that agents can call (not just agents calling external tools).
