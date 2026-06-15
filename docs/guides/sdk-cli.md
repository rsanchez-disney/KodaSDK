# SDK CLI (`npx @koda/sdk`)

Scaffolding and release tooling for Koda SDK-powered Electron apps.

## Commands

### `init` — Scaffold a new app

```bash
npx @koda/sdk init my-app
```

Interactive prompts:
```
? App name: my-app
? Display name: My App
? Description: A sprint analytics dashboard
? Category (productivity/dashboard/devtools/utility): dashboard
? Frontend framework (react/angular/vanilla): react
? Include backend? (y/n): y
? Public release repo: rsanchez-disney/my-app
```

Generates:
```
my-app/
├── koda-app.json            ← App manifest
├── Makefile                  ← build, package, encrypt, release, publish
├── .gitignore
├── desktop/
│   ├── electron-builder.yml
│   ├── package.json
│   └── electron/
│       ├── main.ts          ← SDK initialized here
│       ├── preload.ts       ← window.koda bridge
│       └── tsconfig.json
├── frontend/                ← React or Angular scaffold
│   ├── src/
│   │   ├── App.tsx          ← (React) with ChatPanel example
│   │   └── main.ts
│   └── package.json
├── backend/                 ← (optional) Express API
│   ├── src/
│   │   └── index.ts
│   └── package.json
└── README.md
```

### `release` — Build + encrypt + publish

```bash
npx @koda/sdk release v1.0.0
```

Equivalent to `make release TAG=v1.0.0` but portable (no Makefile dependency):
1. Reads `koda-app.json` for config
2. Bumps version in all package.json + manifest
3. Builds frontend + backend + electron
4. Packages via electron-builder (all platforms in manifest)
5. Encrypts artifacts (STEER_RELEASE_KEY)
6. Creates GitHub release with artifacts
7. Commits version bump + tags

### `validate` — Check manifest + structure

```bash
npx @koda/sdk validate
```

Checks:
- `koda-app.json` exists and is valid
- All `platforms` have matching electron-builder config
- `launch` entries are correct
- SDK dependency is installed
- Required tokens are documented in README

### `status` — Show app info

```bash
npx @koda/sdk status
```

Output:
```
My App v1.0.0
  Repo: github.com/rsanchez-disney/my-app
  Platforms: darwin-arm64, darwin-amd64, windows-amd64
  SDK: @koda/sdk@0.1.0
  Powers: analyze-sprint, explain-burndown
  Tools: get_burndown, get_velocity
  Latest release: v1.0.0 (2026-06-12)
```

## How It Fits Together

```
Developer                     Koda Marketplace
    │                              │
    │  npx @koda/sdk init          │
    │  ───────────────────►        │
    │  (scaffold app)              │
    │                              │
    │  npx @koda/sdk release v1    │
    │  ───────────────────►  ┌─────┴─────┐
    │  (build+encrypt+push)  │  GitHub    │
    │                        │  Releases  │
    │                        └─────┬─────┘
    │                              │
    │                              │  koda apps install my-app
    │                              │◄────────────────────────
    │                              │  (user installs)
    │                              │
```

## Implementation Location

```
packages/sdk-cli/
├── package.json        ← bin: { "koda-sdk": "./dist/index.js" }
├── src/
│   ├── index.ts        ← CLI entry (commander.js)
│   ├── init.ts         ← Scaffold generator
│   ├── release.ts      ← Build + encrypt + publish
│   ├── validate.ts     ← Manifest validation
│   └── templates/      ← File templates (Makefile, main.ts, etc.)
└── tsconfig.json
```

Added as 4th package: `@koda/sdk-cli` (global install or npx).
