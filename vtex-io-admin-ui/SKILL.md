---
name: vtex-io-admin-ui
description: "Apply when building VTEX IO admin interfaces with the modern Admin UI design system (@vtex/admin-ui) under the admin builder with adminVersion 4. Covers Admin UI components and state hooks, csx styling with design tokens, theming, foundations (colors, icons, spacing, typography), experimental features, and migrating from vtex.styleguide. Use for new-generation VTEX Admin pages, settings screens, dashboards, and operational tools that target the updated Admin experience."
---

# Admin UI Interfaces (adminVersion 4)

## When this skill applies

Use this skill when building administrative React interfaces for the updated VTEX Admin experience using `@vtex/admin-ui`.

- New admin apps targeting `adminVersion: 4`
- Settings pages, dashboards, moderation and operational tools styled with Admin UI
- Migrating an existing admin app from `vtex.styleguide` to `@vtex/admin-ui`
- Styling admin screens with `csx` and design tokens

Do not use this skill for:
- legacy admin apps that stay on `vtex.styleguide` (use `vtex-io-admin-react`)
- storefront components or render-runtime blocks
- route authorization design
- backend service structure

## Topic index

Each topic is self-sufficient. Load only what the task needs.

| Topic | File |
|-------|------|
| App wiring: manifest, admin builder, adminVersion 4, navigation/routes | [references/admin-builder-v4.md](references/admin-builder-v4.md) |
| Migrating from vtex.styleguide to @vtex/admin-ui | [references/migrating-from-styleguide.md](references/migrating-from-styleguide.md) |
| Code styleguide (component structure, typing, exports) | [references/code-styleguide.md](references/code-styleguide.md) |
| Styling with csx, cx, dataAttr, responsive aliases | [references/styling.md](references/styling.md) |
| Theming with ThemeProvider | [references/theming.md](references/theming.md) |
| Hooks (useMediaQuery and component state hooks) | [references/hooks.md](references/hooks.md) |
| Design tokens (background, foreground, border, shadow, text, space) | [references/design-tokens.md](references/design-tokens.md) |
| Colors (palettes, tones, usage guidance) | [references/colors.md](references/colors.md) |
| Icons (naming, list, props) | [references/icons.md](references/icons.md) |
| Spacing (scale, padding types, action heights) | [references/spacing.md](references/spacing.md) |
| Typography (variants, font stacks) | [references/typography.md](references/typography.md) |
| Components overview (all components, imports, key props, notes) | [references/components.md](references/components.md) |
| Experimental features (Combobox, DatePicker, Localization) | [references/experimental.md](references/experimental.md) |

## Decision rules

- Use `@vtex/admin-ui` as the design system for admin apps targeting `adminVersion: 4`. Do not mix it with `vtex.styleguide` components on the same screen.
- Declare `"adminVersion": 4` in `admin/navigation.json` so the app renders in the updated Admin experience.
- Wrap every page root in `ThemeProvider`. Add `ToastProvider` when using `useToast` and `experimental_I18nProvider` with the runtime locale for localized built-in strings.
- Use the `Page` pattern (`Page`, `PageHeader`, `PageHeaderTop`, `PageHeaderTitle`, `PageContent`) for screen structure.
- Use the library's state hooks (`useDataViewState`, `usePaginationState`, `useSearchState`, `useTableState`, `useModalState`, `useDropdownState`, `useTabState`, and others) instead of hand-rolling component state.
- Wrap `Table` with `DataView` so loading, empty, error, and not-found states come from `useDataViewState` instead of custom code.
- In `createColumns`, plain columns render skeletons automatically while the DataView status is `loading`, but `root` resolvers do not: their `render` receives `({ item, context })` where `context` is the DataView status, and must return a `Skeleton` when it is `'loading'` (the table feeds placeholder items with `__table_skeleton-N__` strings, which otherwise leak into the cells). Never track the loading state through external closure variables.
- Let `createColumns<T>` infer the `render` parameter types from the column configuration; do not re-annotate `item` manually.
- Style through the `csx` prop (or `csx` function on native elements) with `$`-prefixed design tokens. Prefer shorthand aliases (`bg`, `fg`, `paddingX`, `size`).
- Use responsive aliases (`@tablet`, `@desktop`, `@widescreen`) for responsive styles, not value arrays or custom media queries.
- Use `Form` and `useFormState` from `@vtex/admin-ui-form` (a separate package) for form state and validation.
- Use plain `<ul>`/`<ol>` instead of the deprecated `List` component, and `HeadingLevel` instead of the deprecated `as` prop on `Heading`.
- Prefer `usePaginationState` defaults (25 items per page is the Admin standard) and keep search/filter/pagination state in the URL with the `useQuery*` variants, not in localStorage.
- Use `Toast` (via `useToast`) for temporary feedback and `Alert` for persistent messages.
- Keep experimental imports (`experimental_` prefix) behind a local abstraction and pin the exact `@vtex/admin-ui` version when using them.
- Internationalize labels and feedback through `react-intl` message IDs declared in `messages/` instead of hardcoded copy.

## Hard constraints

### Constraint: adminVersion 4 apps must use @vtex/admin-ui

Admin apps declared with `adminVersion: 4` MUST build their UI with `@vtex/admin-ui`, not `vtex.styleguide` or generic third-party UI libraries.

**Why this matters**

The updated Admin has its own design language, accessibility baseline (Reakit and Downshift underneath), and visual tokens. `vtex.styleguide` components look foreign in it, and third-party libraries break consistency and reviews.

**Detection**

If a `navigation.json` declares `"adminVersion": 4` and page components import from `vtex.styleguide`, Material UI, Chakra, or Ant Design, STOP and replace them with `@vtex/admin-ui` equivalents.

**Correct**

```tsx
import { Page, PageHeader, PageHeaderTop, PageHeaderTitle, PageContent, Button } from '@vtex/admin-ui'
```

**Wrong**

```tsx
import { Layout, PageHeader, PageBlock } from 'vtex.styleguide'
```

### Constraint: the app root must be wrapped in ThemeProvider

Every Admin UI page MUST have `ThemeProvider` at its root. `useToast` additionally requires `ToastProvider` inside it.

**Why this matters**

Admin UI resolves design tokens, global styles, and component styling through the theme context. Components rendered outside `ThemeProvider` lose tokens and global styles; toasts silently fail without `ToastProvider`.

**Detection**

If a route component renders Admin UI components without `ThemeProvider` at the top of its tree, or calls `useToast` without a `ToastProvider` ancestor, STOP and add the providers.

**Correct**

```tsx
import { ThemeProvider, ToastProvider, experimental_I18nProvider as I18nProvider } from '@vtex/admin-ui'
import { useRuntime } from 'vtex.render-runtime'

function App() {
  const { culture: { locale } } = useRuntime()

  return (
    <ThemeProvider>
      <I18nProvider locale={locale}>
        <ToastProvider>{/* page */}</ToastProvider>
      </I18nProvider>
    </ThemeProvider>
  )
}
```

**Wrong**

```tsx
function App() {
  return <Page>{/* page */}</Page>
}
```

### Constraint: styles must use design tokens, not hardcoded values

Styling through `csx` MUST use `$`-prefixed design tokens instead of hardcoded hex colors or pixel values.

**Why this matters**

Tokens carry the Admin's semantic look and feel (states, accessibility contrast, future theme changes). Hardcoded values silently drift from the design system and fail theme updates.

**Detection**

If a `csx` object contains raw hex colors, rgba values, or pixel paddings where a token exists, STOP and replace them with tokens.

**Correct**

```tsx
<div className={csx({ bg: '$secondary', padding: '$space-4' })} />
```

**Wrong**

```tsx
<div className={csx({ bg: '#f4f4f4', padding: '16px' })} />
```

### Constraint: data screens must use DataView state

Screens that list remote data MUST use `useDataViewState` (with `DataView`) so loading, empty, error, and not-found states are explicit.

**Why this matters**

`useDataViewState` ships default illustrations and semantics for every non-ready status. Skipping it recreates the blank-screen problem the pattern exists to solve.

**Detection**

If a page renders a `Table` or list of remote data without `DataView`/`useDataViewState` and has no explicit loading/empty/error rendering, STOP and wrap it.

**Correct**

```tsx
const view = useDataViewState()

return (
  <DataView state={view}>
    <DataViewHeader>{/* Search, Filters, actions */}</DataViewHeader>
    <Table {...getTable()}>{/* ... */}</Table>
  </DataView>
)
```

**Wrong**

```tsx
return <Table items={data?.items ?? []} />
```

### Constraint: experimental features must be version-pinned

Apps importing `experimental_`-prefixed exports MUST pin an exact `@vtex/admin-ui` version in `react/package.json`.

**Why this matters**

Experimental features are explicitly excluded from semver: the docs state they "can introduce breaking changes on patches and minors". A caret range lets a routine dependency update break the build.

**Detection**

If any file imports an `experimental_` export and `react/package.json` declares `@vtex/admin-ui` with `^` or `~`, STOP and pin the exact version (and prefer wrapping the experimental API in a local module).

**Correct**

```json
{ "dependencies": { "@vtex/admin-ui": "0.138.8" } }
```

**Wrong**

```json
{ "dependencies": { "@vtex/admin-ui": "^0.138.8" } }
```

## Preferred pattern

Declare the app with the admin builder and `adminVersion: 4`, route each path to a `react/` component, and wrap the page in `ThemeProvider` (plus `ToastProvider` and `I18nProvider`). Compose the screen with the `Page` pattern, drive lists with `DataView` + `Table` + `Pagination` + `Search` state hooks, style with `csx` and tokens, and give feedback with `useToast` and button `loading` states. See [references/admin-builder-v4.md](references/admin-builder-v4.md) for the full wiring.

## Common failure modes

- Declaring `adminVersion: 4` but importing `vtex.styleguide` components.
- Rendering Admin UI components outside `ThemeProvider`, or calling `useToast` without `ToastProvider`.
- Hardcoding hex colors and pixel values in `csx` instead of tokens.
- Hand-rolling table/pagination/search state instead of using the library hooks.
- Rendering real content from a `root` column resolver during `loading` (placeholder `__table_skeleton-N__` items show in the cells) instead of checking the `context` status.
- Importing `Form` from `@vtex/admin-ui` (it lives in `@vtex/admin-ui-form`).
- Using the deprecated `List` component or the deprecated `as` prop on `Heading`.
- Using responsive value arrays instead of `@tablet`/`@desktop` aliases.
- Using `experimental_` exports with a caret version range.
- Keeping search/filter/pagination state in localStorage instead of the URL.

## Review checklist

- [ ] Does `admin/navigation.json` declare `adminVersion: 4` and match the paths in `admin/routes.json`?
- [ ] Do all UI imports come from `@vtex/admin-ui` (and `@vtex/admin-ui-form` for forms)?
- [ ] Is the page root wrapped in `ThemeProvider` (with `ToastProvider`/`I18nProvider` where needed)?
- [ ] Does the screen use the `Page` pattern for structure?
- [ ] Are lists driven by `DataView`, `useTableState`, `usePaginationState`, and `useSearchState`?
- [ ] Are all `csx` styles token-based, with responsive aliases instead of arrays?
- [ ] Are write actions reporting feedback via `useToast` and button `loading` states?
- [ ] Are labels internationalized through `messages/` and `react-intl`?
- [ ] If experimental exports are used, is the package version pinned exactly?

## Reference

- [Admin UI docs](https://admin-ui.vercel.app/) - Official Admin UI documentation (source of truth)
- [Designing with Admin UI](https://admin-ui.vercel.app/documentation/introduction/designing) - Design principles
- [Admin apps](https://developers.vtex.com/docs/guides/vtex-io-documentation-admin-builder) - Admin builder context
