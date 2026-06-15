# Tokens API

Secure credential management. Tokens are stored in the system keychain and never exposed to the renderer process.

## `koda.tokens`

```typescript
interface TokenManager {
  get(key: string): Promise<string | null>;
  set(key: string, value: string): Promise<void>;
  delete(key: string): Promise<void>;
  list(): Promise<string[]>;
  has(key: string): Promise<boolean>;
}
```

## Usage

### Store a Token

```typescript
await koda.tokens.set('JIRA_PAT', 'pat-value-here');
await koda.tokens.set('GITHUB_TOKEN', 'ghp_...');
```

### Retrieve a Token

```typescript
const pat = await koda.tokens.get('JIRA_PAT');
if (pat) {
  // Use token for API calls
}
```

### Check Existence

```typescript
if (await koda.tokens.has('JIRA_PAT')) {
  // Token available
}
```

### List Token Keys

```typescript
const keys = await koda.tokens.list();
// ['JIRA_PAT', 'GITHUB_TOKEN', ...]
```

### Delete a Token

```typescript
await koda.tokens.delete('GITHUB_TOKEN');
```

## Resolution Order

Tokens are resolved from multiple sources (first match wins):

1. **System keychain** — macOS Keychain, Windows Credential Manager
2. **`~/.kiro/tokens.env`** — Fallback file (existing Koda convention)
3. **Process environment variables** — `process.env[key]`

## Security Guarantees

!!! warning "Renderer Access Restricted"
    In Electron apps, the renderer can only call `tokens.has(key)` — never `tokens.get()`. This prevents token exfiltration from the renderer process.

- Tokens never cross the IPC bridge to renderer
- All token access is logged (auditable)
- SDK passes tokens to MCP servers via env injection, not via app code
- No plaintext storage outside the system keychain
