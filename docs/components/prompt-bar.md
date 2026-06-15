# PromptBar

Standalone prompt input with send button, slash commands, and streaming state indicator.

## Import

```tsx
import { PromptBar } from '@koda/sdk-react';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `sdk` | `KodaSDK \| KodaBridge` | **required** | SDK instance or bridge |
| `onSubmit` | `(prompt: string) => void` | — | Callback on submit |
| `placeholder` | `string` | `'Ask a question...'` | Input placeholder |
| `enableSlashCommands` | `boolean` | `false` | Enable `/` command palette |
| `enableAtMentions` | `boolean` | `false` | Enable `@agent` mentions |
| `disabled` | `boolean` | `false` | Disable input |

## Usage

```tsx
<PromptBar
  sdk={koda}
  onSubmit={(prompt) => handlePrompt(prompt)}
  placeholder="Type a question..."
  enableSlashCommands={true}
  enableAtMentions={true}
/>
```

## Slash Commands

When `enableSlashCommands` is enabled, typing `/` shows a command palette:

- `/switch <agent>` — Switch active agent
- `/clear` — Clear session context
- `/export` — Export current session

## At-Mentions

When `enableAtMentions` is enabled, typing `@` shows an agent picker inline. The selected agent is used for that message only.

```
@qa_agent Find regressions in the last sprint
```
