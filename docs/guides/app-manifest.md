# App Manifest (`koda-app.json`)

Standard manifest that enables any Electron app to be discoverable, installable, and updatable via `koda apps`.

## Schema

```json
{
  "name": "my-app",
  "displayName": "My App",
  "description": "Short description of what the app does",
  "version": "1.0.0",
  "author": "Team Name",
  "repo": "rsanchez-disney/my-app",
  "host": "github.com",
  "category": "productivity",
  "platforms": ["darwin-arm64", "darwin-amd64", "windows-amd64", "linux-amd64"],
  "artifactPattern": "{name}-{platform}.tar.gz.enc",
  "launch": {
    "darwin": "My App.app",
    "win32": "My App.exe",
    "linux": "my-app"
  },
  "sdk": {
    "version": "0.1.0",
    "powers": ["analyze-sprint", "explain-burndown"],
    "tools": ["get_burndown", "get_velocity"]
  },
  "requirements": {
    "kiro": true,
    "tokens": ["JIRA_PAT"]
  },
  "icon": "icon.png"
}
```

## Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | ✅ | Machine name (kebab-case, used in `koda apps install <name>`) |
| `displayName` | ✅ | Human-readable name |
| `description` | ✅ | One-line description |
| `version` | ✅ | Semver version (source of truth) |
| `author` | | Team or person |
| `repo` | ✅ | GitHub repo (releases published here) |
| `host` | | `github.com` (default) or GHE hostname |
| `category` | | `productivity`, `dashboard`, `devtools`, `utility` |
| `platforms` | ✅ | Supported platform-arch combinations |
| `artifactPattern` | | Download filename template (default: `{name}-{platform}.tar.gz.enc`) |
| `launch` | ✅ | Per-OS entry point relative to install dir |
| `sdk.version` | | Koda SDK version used |
| `sdk.powers` | | Powers this app registers |
| `sdk.tools` | | Bidirectional tools this app exposes |
| `requirements.kiro` | | Needs kiro-cli installed |
| `requirements.tokens` | | Required credentials |
| `icon` | | Path to icon file (shown in Koda TUI/apps list) |

## How Koda Uses It

### Discovery
Koda scans registered repos for `koda-app.json` at the repo root (via GitHub API). Apps are automatically added to the catalog without hardcoding.

### Install
```bash
koda apps install my-app
```
1. Fetch latest release from `repo`
2. Download artifact matching current platform via `artifactPattern`
3. Decrypt with STEER_RELEASE_KEY
4. Extract to `~/.koda/bin/{name}/`
5. Write `.version` file

### Launch
```bash
koda apps start my-app
```
Uses `launch.{platform}` to determine the executable path.

### Update
```bash
koda apps install my-app  # idempotent — skips if already at latest
```

### Requirements Check
Before launch, Koda verifies:
- `requirements.kiro` → is kiro-cli installed?
- `requirements.tokens` → are required tokens available?

Shows actionable error if missing.

## Template

Included with `npx @koda/sdk init`:

```json
{
  "name": "{{name}}",
  "displayName": "{{displayName}}",
  "description": "{{description}}",
  "version": "0.1.0",
  "repo": "rsanchez-disney/{{name}}",
  "category": "devtools",
  "platforms": ["darwin-arm64", "darwin-amd64", "windows-amd64"],
  "artifactPattern": "{name}-{platform}.tar.gz.enc",
  "launch": {
    "darwin": "{{displayName}}.app",
    "win32": "{{displayName}}.exe"
  },
  "sdk": {
    "version": "0.1.0"
  },
  "requirements": {
    "kiro": true
  }
}
```
