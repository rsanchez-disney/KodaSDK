# Theming

All `@koda/sdk-react` components inherit styles via CSS custom properties. Set them on `:root` or a parent container to match your app's theme.

## CSS Variables

```css
:root {
  --koda-bg: var(--app-surface);
  --koda-text: var(--app-text);
  --koda-accent: var(--app-primary);
  --koda-border: var(--app-border);
  --koda-code-bg: var(--app-code-bg);
  --koda-radius: 8px;
  --koda-font-mono: 'JetBrains Mono', monospace;
}
```

## Variable Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `--koda-bg` | `#ffffff` | Panel/container background |
| `--koda-text` | `#1a1a1a` | Primary text color |
| `--koda-accent` | `#007acc` | Buttons, links, highlights |
| `--koda-border` | `#e0e0e0` | Border color |
| `--koda-code-bg` | `#f5f5f5` | Code block background |
| `--koda-radius` | `8px` | Border radius |
| `--koda-font-mono` | `monospace` | Code font family |
| `--koda-font-size` | `14px` | Base font size |
| `--koda-spacing` | `12px` | Internal padding |

## Dark Theme Example

```css
[data-theme="dark"] {
  --koda-bg: #1e1e1e;
  --koda-text: #e0e0e0;
  --koda-accent: #4fc3f7;
  --koda-border: #333333;
  --koda-code-bg: #2d2d2d;
}
```

## Scoped Theming

Apply variables to a container to theme Koda components independently of the rest of your app:

```tsx
<div className="koda-container" style={{ '--koda-accent': '#ff6b35' }}>
  <ChatPanel sdk={koda} />
</div>
```

!!! tip "Contrast Requirements"
    Ensure `--koda-text` against `--koda-bg` meets WCAG 2.1 AA contrast ratio (≥4.5:1).
