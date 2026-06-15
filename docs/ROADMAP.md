# Roadmap

## Phase 1: Extract Core (Week 1-2)

Extract Kite's `acp-bridge.ts` into a standalone, testable package.

### Deliverables
- [ ] `core/cli-resolver.ts` — Cross-platform kiro-cli discovery
- [ ] `core/acp-client.ts` — JSON-RPC 2.0 framing (send, receive, request tracking)
- [ ] `core/process-pool.ts` — Spawn, monitor, restart with backoff
- [ ] `core/mcp-loader.ts` — Read ~/.kiro/settings/mcp.json
- [ ] `streaming/router.ts` — Notification dispatcher
- [ ] `streaming/assembler.ts` — Chunk accumulation → complete message
- [ ] `streaming/async-iterator.ts` — `for await` interface
- [ ] Unit tests for all modules
- [ ] `KodaSDK` class with `chat()` method working end-to-end

### Success criteria
```typescript
const koda = new KodaSDK({ appName: 'test' });
const stream = koda.chat('Hello');
for await (const event of stream) { /* works */ }
```

---

## Phase 2: Agents & Tools (Week 3)

Multi-agent support and bidirectional tool registration.

### Deliverables
- [ ] `agents/index.ts` — `koda.agent('x').run(prompt)` API
- [ ] `agents/delegation.ts` — Track sub-agent spawning, forward results
- [ ] `tools/registry.ts` — App registers functions callable by agents
- [ ] `tools/permissions.ts` — Configurable auto-approve or callback
- [ ] Agent switching (new session with different agentId)

### Success criteria
```typescript
koda.tools.register({ name: 'get_sprint', handler: async (p) => fetchSprint(p.id) });
await koda.agent('sprint_manager_agent').run('Show current sprint');
// Agent calls get_sprint tool → handler runs → result returned to agent
```

---

## Phase 3: Sessions & Context (Week 4)

Persistence and workspace-aware context injection.

### Deliverables
- [ ] `sessions/store.ts` — SQLite message persistence
- [ ] `sessions/export.ts` — Markdown/JSON export
- [ ] `context/workspace.ts` — Sync workspace context before ACP session
- [ ] `context/identity.ts` — User identity resolution (git, env)
- [ ] Conversation history injection (recent N messages as context)

### Success criteria
```typescript
const session = await koda.sessions.create({ name: 'Sprint Review' });
await koda.chat('...', { sessionId: session.id }); // persisted
const history = await koda.sessions.messages(session.id);
```

---

## Phase 4: Electron Integration (Week 5)

Preload bridge and renderer-side helpers.

### Deliverables
- [ ] `electron/preload.ts` — Secure IPC bridge (contextBridge)
- [ ] `electron/window.ts` — Forward stream events to BrowserWindow
- [ ] Renderer-side types (`window.koda.*`)
- [ ] React hook: `useKodaChat()` (optional)

### Success criteria
```typescript
// In renderer:
const stream = window.koda.chat('Analyze this');
stream.on('text', (chunk) => appendToUI(chunk));
```

---

## Phase 5: Integrate into DCC (Week 6)

First consumer beyond Kite validates the API.

### Deliverables
- [ ] Add `@koda/sdk` dependency to DCC
- [ ] "AI Insights" panel — ask agent about sprint anomalies
- [ ] Register DCC-specific tools (get_burndown, get_velocity, get_team_health)
- [ ] Agent answers using DCC's live data

### Success criteria
- DCC users click "Analyze" → agent streams response using DCC data
- No ACP code in DCC itself, only SDK calls

---

## Phase 6: Refactor Kite onto SDK (Week 7-8)

Replace Kite's inline `acp-bridge.ts` with `@koda/sdk`.

### Deliverables
- [ ] Remove `acp-bridge.ts` from Kite
- [ ] Wire all IPC handlers through SDK
- [ ] Verify multi-workspace tabs still work (process pool)
- [ ] Verify delegation, streaming, tool calls all pass E2E

### Success criteria
- Kite's full test suite passes with SDK replacing inline ACP code
- Zero regression in features

---

## Phase 7: Scoring & Extensions (Week 9+)

### Deliverables
- [ ] `scoring/index.ts` — Prompt scoring integration
- [ ] Provider abstraction — swap kiro-cli for other ACP backends
- [ ] Plugin hooks — middleware for request/response transformation
- [ ] npm publish workflow (GHE registry)

---

## Non-Goals (v1)

- Web browser support (Electron-only for now)
- Multiple simultaneous LLM providers
- UI components (SDK is headless)
- Public npm registry (GHE private only)
