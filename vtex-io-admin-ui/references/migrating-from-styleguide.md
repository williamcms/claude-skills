# Migrating from vtex.styleguide to @vtex/admin-ui

Source: https://admin-ui.vercel.app/documentation/developing/migrating

The official migration page is a paradigm comparison, not a component-by-component mapping table. There is no published component map from `vtex.styleguide` to `@vtex/admin-ui`; migrate by concept, using the [components overview](components.md) to find the equivalent component.

## Concept-by-concept comparison

| Concept | vtex.styleguide | @vtex/admin-ui |
|---------|-----------------|----------------|
| CSS model | Functional CSS with VTEX Tachyons classes | Stitches: CSS-like, prop-based styling (`csx`) |
| Style overrides | Tachyons `className` combinations | `csx` prop directly on components |
| Responsive | Tachyons suffix classes (`w-30-m w-40-l`) | Responsive aliases in `csx` (`@tablet`, `@desktop`) |
| Accessibility | Many components lack accessibility support | Accessibility-first, built on Reakit and Downshift |
| Typography | Tachyons classes (`t-body lh-copy`) | Dedicated components: `Text`, `Heading`, `Paragraph`, plus the `text` style prop |
| Icons | Exported set, mostly undocumented | Categorized icon set, directional icons, custom icon support |
| Charts | Ready-to-use chart components | Not available yet ("will be implemented in the future") |
| Component state | Self-contained, manually controlled via `value`/`onChange` | State hooks exported by the library (`useDropdownState`, etc.) |
| Design reuse | Combining the same Tachyons class tokens | Semantic design tokens + layout components (`Bleed`, `Stack`, `Inline`, `Flex`, `Grid`) |

## Style overrides

```jsx
<Card csx={{ padding: '$s' }}>Card content</Card>
<Button csx={{ border: 'default' }}>Label</Button>
```

## Responsive design

Styleguide (Tachyons):

```jsx
<div className="w-10 w-30-m w-40-l">{content}</div>
```

Admin UI:

```jsx
<div className={csx({ width: ['10%', '30%', '40%'] })} />
```

Note: the migration page shows the array form, but the code styleguide recommends responsive aliases (`@tablet`, `@desktop`) instead of arrays. Prefer aliases in new code.

## State: manual vs hook-based

Styleguide:

```jsx
function Example() {
  const [state, setState] = useState('Item 1')
  const options = [
    { value: 'Item 1', label: '1' },
    { value: 'Item 2', label: '2' },
    { value: 'Item 3', label: '3' },
  ]

  return (
    <Dropdown
      label="dropdown"
      options={options}
      value={state}
      onChange={(_, v) => setState(v)}
    />
  )
}
```

Admin UI:

```jsx
function Example() {
  const items = ['Item 1', 'Item 2', 'Item 3']
  const state = useDropdownState({ items, initialSelectedItem: 'Item 1' })

  return <Dropdown items={items} state={state} label="dropdown" />
}
```

## Typography

Styleguide:

```jsx
<p className="t-body lh-copy">This is a paragraph!</p>
```

Admin UI:

```jsx
<Paragraph>This is a paragraph!</Paragraph>
<Heading>This is a Heading!</Heading>
<Text variant="subtitle">Text with subtitle style!</Text>
```

## Custom CSS property shorthands (tokens)

Border:

```jsx
// without token
<div
  className={cn({
    borderBottom: 'solid',
    borderBottomWidth: '1px',
    borderBottomColor: 'mid.secondary',
  })}
/>
// with token
<div className={cn({ border: 'divider-bottom' })} />
```

Size:

```jsx
// without token
<div className={cn({ width: 100, height: 100 })} />
// with token
<div className={cn({ size: 100 })} />
```

Text:

```jsx
// without token
<div
  className={cn({
    fontFamily: 'sans',
    lineHeight: 'headline',
    fontSettings: 'regular',
    fontSize: 4,
  })}
/>
// with token
<div className={cn({ text: 'headline' })} />
```

## Practical migration checklist

- [ ] Replace `vtex.styleguide` imports with `@vtex/admin-ui` equivalents (see [components.md](components.md))
- [ ] Replace Tachyons classNames with `csx` token-based styles
- [ ] Move component state to the library's state hooks
- [ ] Replace typography classes with `Text`/`Heading`/`Paragraph`
- [ ] Declare `"adminVersion": 4` in `admin/navigation.json` (see [admin-builder-v4.md](admin-builder-v4.md))
- [ ] Wrap the page root in `ThemeProvider`
- [ ] Charts have no Admin UI equivalent yet; plan those separately
