# Typography

Source: https://admin-ui.vercel.app/foundations/typography

Admin UI uses the VTEX Trust typeface. Typography is consumed through the `Text`/`Heading`/`Paragraph` components or the `text` style prop with `$`-prefixed tokens.

## Variants and where they're used

| Variant | Token | Used for |
|---------|-------|----------|
| Page Title | `$pageTitle` | Page header title |
| Display | `$display` | Totalizer value |
| Title 1 | `$title1` | Tab title; table line title; card title; modal title; totalizer percentage |
| Title 2 | `$title2` | Nested card title |
| Body | `$body` | Text in tables, modals, toasts, alerts, cards; form fields and controls; placeholder text |
| Detail | `$detail` | Text in tag, tooltip, table pagination; form error and help text |
| Action 1 | `$action1` | Text in regular and small buttons; account name in top bar |
| Action 2 | `$action2` | Left nav and action menu items; links |

Each token bundles `fontVariationSettings`, `fontSize`, `lineHeight`, and `letterSpacing` (numeric values in [design-tokens.md](design-tokens.md)).

## Font stacks

Sans-serif:

```css
font-family: 'VTEX Trust', -apple-system, system-ui, BlinkMacSystemFont, sans-serif;
```

Monospace:

```css
font-family: 'MonoLisa', 'Operator Mono', 'Fira Code Retina', 'Fira Code', 'FiraCode-Retina', 'Dank Mono', 'Consolas', 'Monaco', 'Menlo', monospace;
```

## Typography components

| Component | HTML equivalent | Notes |
|-----------|-----------------|-------|
| `Heading` | `h1`-`h6` | Automatic hierarchy via `HeadingLevel`; the `as` prop is deprecated |
| `Text` | `span` (or via `as`) | `variant` prop takes the variants above; `tone` prop for semantic color |
| `Paragraph` | `p` | Reset-styled paragraph |
| `List` | `ul`, `ol`, `li` | Deprecated; use plain HTML list elements |

Usage:

```tsx
<Text variant="title1">Headline</Text>
<Text variant="body" tone="secondary">Supporting copy</Text>
<div className={csx({ text: '$detail' })}>Token-based text</div>
```
