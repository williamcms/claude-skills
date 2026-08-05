# Spacing

Source: https://admin-ui.vercel.app/foundations/spacing

The spacing scale uses multiples of 4px with a broad range so it applies to both large and small elements. Consume through `$space-*` tokens on `padding`, `margin`, `gap`, and related props.

Note on t-shirt aliases: official docs examples sometimes use `$xs`/`$s`/`$m` spacing values, and some component props default to them (`Inline` `$s`, `Columns` `$m`). The `$space-*` scale is the documented token set and the one verified in production admin apps; prefer `$space-*` in `csx` styles.

## Scale

| Token | Pixels | Rem | Primary use cases |
|-------|--------|-----|-------------------|
| $space-0 | 0 | 0 | Vertical space between body and detail text |
| $space-05 | 2 | 0.125 | Body-to-detail text transitions |
| $space-1 | 4 | 0.25 | Label-to-input spacing; text-to-detail spacing |
| $space-2 | 8 | 0.5 | Trigger-to-overlay gaps; control-to-text spacing |
| $space-3 | 12 | 0.75 | Icon-to-text spacing; field-to-text spacing |
| $space-4 | 16 | 1 | Control/input fields; cards; toolbars |
| $space-5 | 20 | 1.25 | Control field spacing; container padding |
| $space-6 | 24 | 1.5 | Section header-to-content spacing |
| $space-7 | 28 | 1.75 | Large optical container padding |
| $space-8 | 32 | 2 | Large even container padding |
| $space-10 | 40 | 2.5 | Standard page margin-top |
| $space-12 | 48 | 3 | Page margin-bottom; margin-top |
| $space-16 | 64 | 4 | Form section spacing |
| $space-20 | 80 | 5 | Standard page margin-bottom |
| $space-24 | 96 | 6 | — |
| $space-28 | 112 | 7 | — |
| $space-32 | 128 | 8 | Narrow page margin-bottom |

## Container padding types

| Type | Definition | Examples |
|------|------------|----------|
| Wide | Horizontal padding much larger than vertical | Tag, Page Header |
| Narrow | Vertical padding much larger than horizontal | Tabs |
| Even | Same value on both axes | Card, Modal |
| Optical | Horizontal slightly larger than vertical | Alert, Toast |

## Action heights

- Small: up to 28px (e.g. combobox selected option)
- Medium: 28px-44px (buttons, menus, filters)
- Large: 44px+ (large buttons, menus, tabs)

## Principles

- Relationships and hierarchy: meaningfully related elements should be closer to each other; elements with more spacing around them read as more relevant.
- Visual rhythm: spacing creates rhythm, improving predictability, scannability, and hierarchy.
- Consistent density: tighter spacing increases scannability for large data sets; looser spacing increases hierarchy or defines groups.
- Visual weight: the same value on both axes doesn't always balance; that's what the optical padding type is for.
