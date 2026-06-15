# AgentPicker

Dropdown/modal for discovering and switching between available agents.

## Import

```tsx
import { AgentPicker } from '@koda/sdk-react';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `sdk` | `KodaSDK \| KodaBridge` | **required** | SDK instance or bridge |
| `current` | `string` | — | Currently active agent ID |
| `onSwitch` | `(id: string) => void` | — | Callback on agent switch |
| `filter` | `{ profile?: string }` | — | Filter agents by profile |

## Usage

```tsx
const [agent, setAgent] = useState('orchestrator');

<AgentPicker
  sdk={koda}
  current={agent}
  onSwitch={(id) => {
    setAgent(id);
    koda.agents.switch(id);
  }}
  filter={{ profile: 'dev' }}
/>
```

## Features

- Shows agent name, description, and available tools
- Groups agents by profile
- Highlights currently active agent
- Search/filter within the picker
- Keyboard navigable (arrow keys, Enter to select)
