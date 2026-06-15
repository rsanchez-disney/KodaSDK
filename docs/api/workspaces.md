# Workspaces API

Discover and switch between workspaces.

## `koda.workspaces`

```typescript
interface WorkspaceManager {
  list(): Promise<WorkspaceInfo[]>;
  active(): Promise<WorkspaceInfo | null>;
  switch(id: string): Promise<void>;
  projects(id: string): Promise<{ name: string; repo?: string; host?: string }[]>;
}
```

## WorkspaceInfo

```typescript
interface WorkspaceInfo {
  id: string;
  name: string;
  team: string;
  description: string;
  jiraPrefix: string;
  profiles: string[];
  defaultAgent: string;
  projects: { name: string; repo?: string }[];
  installed: boolean;
}
```

## Usage

### List Workspaces

```typescript
const workspaces = await koda.workspaces.list();
workspaces.forEach(w => console.log(`${w.id}: ${w.name} (${w.team})`));
```

### Get Active Workspace

```typescript
const active = await koda.workspaces.active();
if (active) {
  console.log(`Current: ${active.name}`);
  console.log(`Profiles: ${active.profiles.join(', ')}`);
}
```

### Switch Workspace

```typescript
await koda.workspaces.switch('payments-vertical');
```

!!! note "Process Isolation"
    Switching workspaces spawns a new kiro-cli process. Each workspace gets its own isolated subprocess with its own MCP server set and agent configuration.

### List Projects

```typescript
const projects = await koda.workspaces.projects('payments-vertical');
// [{ name: 'config-services', repo: 'org/config-services', host: 'github.com' }]
```
