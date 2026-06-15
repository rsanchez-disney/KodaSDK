# Implementation Plan

## Overview

Build the Koda SDK as a monorepo with three packages, extracted iteratively from Kite's production code. Each sprint delivers a working, tested slice that can be consumed immediately.

```
Sprint 1-2: Core (ACP, process, streaming)
Sprint 3:   Discovery (agents, workspaces, MCP)
Sprint 4:   Security (tokens, permissions)
Sprint 5:   Powers (invocation, registration)
Sprint 6:   UI Components (@koda/sdk-react)
Sprint 7:   Integration (DCC + Kite refactor)
Sprint 8:   Polish (docs, tests, publish)
```

---

## Sprint 1 — Foundation (Days 1-5)

**Goal:** Monorepo scaffold + ACP client that can connect and stream.

### Tasks

| # | Task | Output |
|---|------|--------|
| 1.1 | Init monorepo (npm workspaces, tsconfig, eslint, vitest) | `packages/sdk`, `packages/sdk-electron`, `packages/sdk-react` |
| 1.2 | Extract `cli-resolver.ts` from Kite | Cross-platform kiro-cli discovery |
| 1.3 | Extract `acp-client.ts` — JSON-RPC framing | `send()`, `request()`, line-buffered receive |
| 1.4 | Extract `process-pool.ts` — spawn + monitor | Single process lifecycle with crash recovery |
| 1.5 | Implement `streaming/assembler.ts` | Chunk accumulation → StreamEvent emission |
| 1.6 | Implement `streaming/async-iterator.ts` | `for await (const event of stream)` interface |
| 1.7 | Wire into `KodaSDK` class with `chat()` | End-to-end: `new KodaSDK() → chat() → events` |
| 1.8 | Unit tests (vitest) | CLI resolver, JSON-RPC framing, assembler |

### Acceptance
```typescript
const koda = new KodaSDK({ appName: 'test' });
for await (const e of koda.chat('Hello')) { console.log(e); }
```

---

## Sprint 2 — Process Pool & Sessions (Days 6-10)

**Goal:** Multi-process support (per workspace) and message persistence.

### Tasks

| # | Task | Output |
|---|------|--------|
| 2.1 | Multi-process pool — spawn N kiro-cli processes | `pool.get(workspaceId)` returns dedicated process |
| 2.2 | Process health monitoring + auto-restart with backoff | Resilient to kiro-cli crashes |
| 2.3 | `sessions/store.ts` — SQLite via better-sqlite3 | Create, list, persist messages |
| 2.4 | `sessions/export.ts` — markdown + JSON export | `koda.sessions.export(id, 'markdown')` |
| 2.5 | Auto-persist messages during chat | Every user/assistant message saved |
| 2.6 | Session resume — inject history as context | `koda.sessions.resume(id)` |
| 2.7 | Integration tests (real kiro-cli subprocess) | Spawn, chat, verify events arrive |

### Acceptance
```typescript
const session = await koda.sessions.create({ name: 'Test' });
for await (const e of koda.chat('Hi', { sessionId: session.id })) {}
const msgs = await koda.sessions.messages(session.id); // persisted
```

---

## Sprint 3 — Discovery APIs (Days 11-15)

**Goal:** List and switch workspaces, agents, MCP servers.

### Tasks

| # | Task | Output |
|---|------|--------|
| 3.1 | `workspaces.list()` — scan steer-runtime + installed | Full WorkspaceInfo with projects |
| 3.2 | `workspaces.switch(id)` — sync context + restart ACP | Workspace isolation |
| 3.3 | `agents.list()` — read from ~/.kiro/agents/*.json | AgentInfo with tools, profile |
| 3.4 | `agents.listByProfile(profile)` — filter by profile | Profile-aware discovery |
| 3.5 | `agents.switch(id)` — new ACP session with agent | Agent switching |
| 3.6 | `mcp.list()` — merge global + workspace + fork servers | McpServer[] with source |
| 3.7 | `mcp.tools(server)` — list tools from a server | McpTool[] |
| 3.8 | `mcp-loader.ts` — read + merge mcp.json sources | Unified MCP config |

### Acceptance
```typescript
const ws = await koda.workspaces.list();
await koda.workspaces.switch('payments-vertical');
const agents = await koda.agents.listByProfile('dev');
const tools = await koda.mcp.tools('jira-jira');
```

---

## Sprint 4 — Security & Tokens (Days 16-20)

**Goal:** Secure credential management, never expose tokens to renderer.

### Tasks

| # | Task | Output |
|---|------|--------|
| 4.1 | `tokens/keychain.ts` — macOS Keychain via `keytar` | Set/get/delete from system keychain |
| 4.2 | `tokens/resolver.ts` — keychain → tokens.env → env | Priority-based resolution |
| 4.3 | `tokens.get(key)` / `tokens.set(key, value)` API | Public interface |
| 4.4 | `tokens.has(key)` — safe check for renderer | Boolean only, no value |
| 4.5 | Audit logging — log token access (key, timestamp, caller) | Security trail |
| 4.6 | `tools/permissions.ts` — configurable auto-approve | Policy: `trust-all`, `ask`, `deny-by-default` |
| 4.7 | IPC boundary enforcement — tokens never cross to renderer | Preload only exposes `has()` |
| 4.8 | MCP env injection — pass tokens to servers without app code | Servers get tokens from SDK |

### Acceptance
```typescript
await koda.tokens.set('JIRA_PAT', 'NjY...');
const has = await koda.tokens.has('JIRA_PAT'); // true (safe for renderer)
// MCP servers automatically get JIRA_PAT in their env
```

---

## Sprint 5 — Powers (Days 21-25)

**Goal:** High-level AI capabilities with dependency checking and invocation.

### Tasks

| # | Task | Output |
|---|------|--------|
| 5.1 | `powers/registry.ts` — register power definitions | In-memory + file-based registration |
| 5.2 | `powers/resolver.ts` — check requirements (tools, tokens) | `power.available` boolean |
| 5.3 | `powers.list()` / `powers.available()` | Discovery API |
| 5.4 | `powers.invoke(id, params)` — streaming execution | AsyncIterable<StreamEvent> |
| 5.5 | `powers.run(id, params)` — await full result | PowerResult with content + data |
| 5.6 | Prompt template expansion — `{{param}}` interpolation | Parameterized prompts |
| 5.7 | Built-in powers catalog (analyze-sprint, explain-error, etc.) | Default powers shipped with SDK |
| 5.8 | `koda.tools.register()` — bidirectional tool handler | App functions callable by agents |

### Acceptance
```typescript
koda.powers.register({
  id: 'team-health',
  name: 'Team Health',
  agent: 'sprint_manager_agent',
  requiredTokens: ['JIRA_PAT'],
  promptTemplate: 'Analyze sprint health for team {{teamId}}',
  parameters: [{ name: 'teamId', type: 'string', required: true }],
});
const result = await koda.powers.run('team-health', { teamId: 'bolt' });
```

---

## Sprint 6 — UI Components (Days 26-30)

**Goal:** `@koda/sdk-react` with drop-in components.

### Tasks

| # | Task | Output |
|---|------|--------|
| 6.1 | Package scaffold — React 18+, CSS modules, CSS vars | `packages/sdk-react` |
| 6.2 | `ChatPanel` — full chat with streaming, markdown, tools | Main component |
| 6.3 | `PromptBar` — input with send, slash commands, @mentions | Standalone input |
| 6.4 | `ResponseBubble` — markdown + code + copy button | Single message render |
| 6.5 | `PowerButton` — invoke power, show inline result | Action button |
| 6.6 | `AgentPicker` — dropdown/modal for agent switching | Selection component |
| 6.7 | CSS variable theming — host app controls colors | `--koda-*` variables |
| 6.8 | Storybook for component development/demo | Visual testing |

### Acceptance
```tsx
<ChatPanel sdk={koda} placeholder="Ask about your sprint..." maxHeight={400} />
<PowerButton sdk={koda} powerId="team-health" params={{ teamId: 'bolt' }} label="Check Health" />
```

---

## Sprint 7 — Integration (Days 31-37)

**Goal:** DCC consumes SDK (first external app), Kite refactors onto SDK.

### Tasks

| # | Task | Output |
|---|------|--------|
| 7.1 | Add `@koda/sdk` + `@koda/sdk-react` to DCC | Dependencies wired |
| 7.2 | DCC "AI Insights" panel using ChatPanel | Streaming AI in DCC |
| 7.3 | DCC registers powers (analyze-sprint, explain-burndown) | App-specific powers |
| 7.4 | DCC registers tools (get_burndown, get_velocity) | Agent calls DCC functions |
| 7.5 | Kite: replace `acp-bridge.ts` with `@koda/sdk` | Remove 385 lines |
| 7.6 | Kite: replace custom streaming with SDK events | Unified event model |
| 7.7 | Kite: verify multi-workspace tabs with SDK pool | Process pool works |
| 7.8 | E2E tests for both apps on SDK | Regression-free |

### Acceptance
- DCC has working AI panel with zero ACP code
- Kite passes all E2E tests on SDK
- Both apps share same `@koda/sdk` version

---

## Sprint 8 — Polish & Publish (Days 38-42)

**Goal:** Production-ready, documented, publishable.

### Tasks

| # | Task | Output |
|---|------|--------|
| 8.1 | API documentation (TypeDoc generation) | Published API docs |
| 8.2 | Migration guide (Kite → SDK) | For other apps to follow |
| 8.3 | Error handling audit — all edge cases covered | Graceful degradation |
| 8.4 | Performance benchmarks — startup time, memory | < 100ms to first event |
| 8.5 | GHE npm registry publish workflow | `npm publish` to private registry |
| 8.6 | Koda integration — `koda sdk init <app>` scaffolding | CLI helper |
| 8.7 | Add to steer-platform workspace catalog | Ecosystem visibility |
| 8.8 | Release v1.0.0 | Stable, documented, tested |

---

## Dependencies Between Sprints

```
Sprint 1 (Core) ──────► Sprint 2 (Pool/Sessions)
         │                      │
         ▼                      ▼
Sprint 3 (Discovery) ──► Sprint 5 (Powers)
         │                      │
         ▼                      ▼
Sprint 4 (Security) ──► Sprint 7 (Integration)
                               │
Sprint 6 (UI) ────────────────►│
                               ▼
                        Sprint 8 (Publish)
```

Sprints 3, 4, and 6 can run in parallel once Sprint 1-2 are done.

---

## Tech Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Monorepo tool | npm workspaces | Already used in Kite, zero new tooling |
| Build | tsup (esbuild) | Fast, outputs CJS + ESM |
| Test | vitest | Fast, native ESM, compatible with Kite |
| DB (sessions) | better-sqlite3 | Sync, no native rebuild issues in Electron |
| Keychain | keytar | Cross-platform, battle-tested in VS Code |
| UI framework | React 18+ | All consumers already use React |
| Styling | CSS modules + vars | No runtime CSS-in-JS, themeable |
| Docs | TypeDoc | Auto-generated from TSDoc comments |

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| kiro-cli ACP protocol changes | Breaks SDK | Version-pin protocol, add compatibility layer |
| Electron version mismatch (native modules) | Build failures | Use `electron-rebuild`, test on 33+ |
| keytar deprecation | Security gap | Fallback to `safeStorage` (Electron 15+) |
| Performance overhead vs inline code | Slower startup | Lazy-load modules, benchmark in Sprint 8 |
| UI components too opinionated | Low adoption | CSS vars for 100% theming, headless mode option |
