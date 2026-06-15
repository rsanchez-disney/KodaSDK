# ChatPanel

Full chat interface — prompt input, streaming responses, and tool call indicators.

## Import

```tsx
import { ChatPanel } from '@koda/sdk-react';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `sdk` | `KodaSDK \| KodaBridge` | **required** | SDK instance or bridge |
| `sessionId` | `string` | auto-created | Session to use |
| `agent` | `string` | SDK default | Agent for this panel |
| `placeholder` | `string` | `'Ask a question...'` | Input placeholder |
| `maxHeight` | `number` | `undefined` | Max panel height (px) |
| `showToolCalls` | `boolean` | `false` | Show tool call indicators |
| `onMessage` | `(msg: Message) => void` | — | Callback on each message |

## Usage

```tsx
<ChatPanel
  sdk={koda}
  sessionId="sprint-review"
  agent="orchestrator"
  placeholder="Ask about your sprint..."
  maxHeight={400}
  showToolCalls={true}
  onMessage={(msg) => console.log(msg)}
/>
```

## Features

- Streaming markdown rendering
- Code block syntax highlighting
- Copy-to-clipboard for responses
- Tool call status indicators (when `showToolCalls` is enabled)
- Auto-scroll during streaming
- Keyboard shortcuts (Enter to send, Shift+Enter for newline)

## Accessibility

- Full keyboard navigation
- ARIA live regions for streaming content
- Focus management on message send/receive
- Screen reader announcements for tool calls
