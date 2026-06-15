# PowerButton

One-click button that invokes a registered power and shows the result inline.

## Import

```tsx
import { PowerButton } from '@koda/sdk-react';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `sdk` | `KodaSDK \| KodaBridge` | **required** | SDK instance or bridge |
| `powerId` | `string` | **required** | Power to invoke |
| `params` | `Record<string, unknown>` | `{}` | Parameters for the power |
| `label` | `string` | Power name | Button label |
| `variant` | `'compact' \| 'full' \| 'icon'` | `'compact'` | Visual variant |
| `onResult` | `(result: PowerResult) => void` | — | Callback on completion |
| `onError` | `(error: Error) => void` | — | Error callback |

## Usage

```tsx
<PowerButton
  sdk={koda}
  powerId="analyze-sprint"
  params={{ teamId: 'lodging-sales' }}
  label="Analyze Sprint"
  variant="compact"
  onResult={(result) => showPanel(result.content)}
/>
```

## Variants

- **`compact`** — Small button, shows result in a tooltip/popover
- **`full`** — Standard button, expands inline to show streaming result
- **`icon`** — Icon-only, minimal footprint

## States

The button manages its own loading state:

1. **Idle** — Shows label/icon
2. **Loading** — Shows spinner, button disabled
3. **Streaming** — Shows partial result (in `full` variant)
4. **Complete** — Shows result or success indicator

!!! tip
    Use `available()` from the Powers API to conditionally render PowerButtons only when their requirements are met.
