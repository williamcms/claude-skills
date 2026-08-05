# Theming in Admin UI

Source: https://admin-ui.vercel.app/documentation/developing/theming

The theme object defines color palettes, typography scales, spacing, breakpoints, and border-radius values. Note: older Admin UI releases configured theming via `createSystem`; current versions (0.136.0+) use `ThemeProvider` props instead.

## ThemeProvider

Wrap the app's root component. Nothing renders correctly without it: components resolve tokens and global styles through this context.

```javascript
import { ThemeProvider } from '@vtex/admin-ui'

function RootComponent() {
  return <ThemeProvider>{/* your app code here */}</ThemeProvider>
}
```

## Consuming theme tokens

```javascript
import { csx } from '@vtex/admin-ui'

function Example() {
  return (
    <nav
      className={csx({
        bg: '$primary',
        'button + button': { marginLeft: '$s' },
      })}
    />
  )
}
```

## Customizing the theme

Override the default theme with `experimentalTheme`:

```javascript
<ThemeProvider experimentalTheme={customTheme}>{/* app code */}</ThemeProvider>
```

Disable global styles with `experimentalDisabledGlobalStyles`:

```javascript
<ThemeProvider experimentalDisabledGlobalStyles>{/* app code */}</ThemeProvider>
```

Rule from the docs: when overriding themes, use the `cn` function instead of `csx` for styling.

Both props carry the `experimental` prefix; treat them under the experimental policy (pin the exact package version, see [experimental.md](experimental.md)).

## Theme scales

Breakpoints:

| Name | Width |
|------|-------|
| mobile | 640px |
| tablet | 768px |
| desktop | 1024px |
| widescreen | 1200px |

Spacing: base unit 0.25rem (4px), steps 0-8 plus `px` and `2px` variants. Full scale in [spacing.md](spacing.md).

Typography tokens: `$pageTitle`, `$title1`, `$title2`, `$action1`, `$action2`, `$display`, `$body`, `$detail` (details in [typography.md](typography.md)).

Border radius:

| Name | Value |
|------|-------|
| default | 4px |
| flat | 0px |
| pill | 100px |
| circle | 100% |
