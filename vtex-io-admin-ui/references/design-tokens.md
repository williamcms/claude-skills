# Design tokens

Source: https://admin-ui.vercel.app/foundations/design-tokens

Design tokens are variables that express the look and feel of the VTEX Admin. Each token is a `$`-prefixed string valid for a specific style prop (`background`, `color`, `border`, `boxShadow`, `text`, `space`) and generates a CSS variable in the format `--admin-ui-[type]-[name]` (`bg`, `fg`, `border`, `shadow`). All tokens are marked AA accessibility level.

Usage: pass the token as the style prop value in `csx` (see [styling.md](styling.md)):

```tsx
<div className={csx({ bg: '$secondary', color: '$critical', padding: '$space-4', text: '$title1' })} />
```

## Naming conventions

- Base color scales run light to dark: `blue05` through `blue60` (also lightBlue, red, green, orange, cyan, purple, pink, teal, gray).
- Semantic aliases: `$primary`, `$secondary`, `$positive`, `$critical`, `$warning`, `$info`, `$disabled`.
- State suffixes: `Hover`, `Pressed`, `Checked`, `Active`, `Inactive`, `Focus`, `Selected`.
- Dotted names for contextual groups: `$action.main.primary`, `$form.neutralChecked`, `$ring.main`, `$overlay.center`.

## Background tokens (prop: `background` / `bg`)

Semantic:

| Token | CSS Variable | Value |
|-------|--------------|-------|
| $primary | --admin-ui-bg-primary | #FFFFFF |
| $secondary | --admin-ui-bg-secondary | #F4F4F4 |
| $disabled | --admin-ui-bg-disabled | #E6E6E6 |
| $positive | --admin-ui-bg-positive | #DFF1E0 |
| $critical | --admin-ui-bg-critical | #F8E3E3 |
| $warning | --admin-ui-bg-warning | #FFEBD7 |
| $info | --admin-ui-bg-info | #DEECF7 |
| $inverted | --admin-ui-bg-inverted | #000000 |
| $overlay | --admin-ui-bg-overlay | rgba(0, 0, 0, 0.5) |
| $skeleton | --admin-ui-bg-skeleton | linear-gradient(90deg, transparent, #E6E6E6, transparent) |

Base scale example (blue; other palettes follow the same 05-60 numbering, values in [colors.md](colors.md)):

| Token | Value |
|-------|-------|
| $white | #FFFFFF |
| $black | #000000 |
| $blue05 | #F1F4FC |
| $blue10 | #DDE5F7 |
| $blue20 | #BBCBF0 |
| $blue30 | #96B2F2 |
| $blue40 | #2953B2 |
| $blue50 | #20408C |
| $blue60 | #172F66 |

Action tokens (examples):

| Token | Value |
|-------|-------|
| $action.main.primary | #2953B2 |
| $action.main.primaryHover | #20408C |
| $action.critical.primary | #CC3D3D |
| $action.neutral.tertiary | transparent |
| $action.neutral.tertiaryHover | rgba(0, 0, 0, 0.05) |

Form tokens:

| Token | Value |
|-------|-------|
| $form.neutral | transparent |
| $form.neutralHover | #E6E6E6 |
| $form.neutralPressed | #B9B9B9 |
| $form.neutralChecked | #000000 |
| $form.controlHover | transparent |
| $form.controlChecked | #2953B2 |

Transparency: `$grayTransparent05` = rgba(0,0,0,0.05) through `$grayTransparent90` = rgba(0,0,0,0.9).

## Foreground tokens (prop: `color` / `fg`)

Mirror the background structure with the `--admin-ui-fg-` prefix. Key semantic values:

| Token | Value |
|-------|-------|
| $primary | #000000 |
| $secondary | #545454 |
| $disabled | #747474 |
| $positive | #38853C |
| $info | #2978B5 |
| $critical | #CC3D3D |
| $warning | #D56A00 |

## Border tokens (prop: `border`)

| Token | Value |
|-------|-------|
| $neutral | 1px solid #DDDDDD |
| $mainSelected | 2px solid #2953B2 |
| $positive | 1px solid #83D187 |
| $info | 1px solid #7CBBEA |
| $critical | 1px solid #F49494 |
| $warning | 1px solid #F5AE70 |
| $disabled | 1px solid #747474 |

## Shadow tokens (prop: `boxShadow`)

| Token | Value |
|-------|-------|
| $ring.main | 0 0 0 0.0625rem #F1F4FC, 0 0 0 0.1875rem #96B2F2 |
| $ring.neutral | 0 0 0 0.0625rem #F4F4F4, 0 0 0 0.1875rem #B9B9B9 |
| $ring.critical | 0 0 0 0.0625rem #FBF2F2, 0 0 0 0.1875rem #F49494 |
| $overlay.center | 0 0 1rem 0 rgba(0,0,0,0.10) |
| $overlay.bottom | 0 0 1.5rem 0 rgba(0,0,0,0.05) |

## Typography tokens (prop: `text`)

All use `font-family: "VTEX Trust", -apple-system, system-ui, BlinkMacSystemFont, sans-serif`.

| Token | Weight | Size | Line Height | Letter Spacing |
|-------|--------|------|-------------|----------------|
| $display | 500 | 1.5rem | ~1.33 | -0.004em |
| $pageTitle | 400 | 1.25rem | 1.2 | -0.004em |
| $title1 | 500 | 1rem | 1.5 | 0em |
| $title2 | 400 | 1rem | 1.5 | 0em |
| $action1 | 500 | 0.875rem | ~1.14 | 0.01em |
| $action2 | 400 | 0.875rem | ~1.14 | 0.01em |
| $body | 400 | 0.875rem | ~1.43 | 0.002em |
| $detail | 400 | 0.75rem | ~1.33 | 0.004em |

Where each variant is used: see [typography.md](typography.md).

## Spacing tokens (props: `padding`, `margin`, `gap`, etc.)

| Token | Value |
|-------|-------|
| $space-0 | 0rem |
| $space-05 | 0.125rem |
| $space-1 | 0.25rem |
| $space-2 | 0.5rem |
| $space-3 | 0.75rem |
| $space-4 | 1rem |
| $space-5 | 1.25rem |
| $space-6 | 1.5rem |
| $space-7 | 1.75rem |
| $space-8 | 2rem |
| $space-10 | 2.5rem |
| $space-12 | 3rem |
| $space-16 | 4rem |
| $space-20 | 5rem |
| $space-24 | 6rem |
| $space-28 | 7rem |
| $space-32 | 8rem |

Use-case guidance per step: see [spacing.md](spacing.md). Official docs examples sometimes use t-shirt spacing aliases (`$xs`, `$s`, `$m`); prefer the `$space-*` scale, which is the documented set and the one verified in production apps.
