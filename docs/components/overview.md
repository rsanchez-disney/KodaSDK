# UI Components

`@koda/sdk-react` provides pre-built React components for common AI interaction patterns. All components are themed via CSS variables to match your host app.

## Installation

```bash
npm install @koda/sdk-react
```

## Available Components

| Component | Description |
|-----------|-------------|
| [`ChatPanel`](chat-panel.md) | Full chat interface with prompt, streaming responses, tool indicators |
| [`PromptBar`](prompt-bar.md) | Standalone prompt input with send button and slash commands |
| [`PowerButton`](power-button.md) | One-click button that invokes a power and shows result inline |
| [`AgentPicker`](agent-picker.md) | Dropdown for switching agents |

## Quick Example

```tsx
import { ChatPanel } from '@koda/sdk-react';

function App() {
  return (
    <ChatPanel
      sdk={window.koda}
      placeholder="Ask about your sprint..."
      showToolCalls={true}
      maxHeight={400}
    />
  );
}
```

## Theming

All components inherit styles from CSS variables. See [Theming](theming.md) for the full variable reference.

```css
:root {
  --koda-bg: #1e1e1e;
  --koda-text: #ffffff;
  --koda-accent: #007acc;
}
```
