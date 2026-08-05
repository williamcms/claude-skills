# Components overview

Source: https://admin-ui.vercel.app/components/ (one page per component, e.g. `/components/alert`)

All components import from `@vtex/admin-ui` unless noted (`Form` comes from `@vtex/admin-ui-form`). This is an index with the key facts from each component's documentation, not a full API reference; check the component's docs page for complete props.

Recurring patterns:

- State hooks: many components take a `state` prop from their `use*State` hook (catalog in [hooks.md](hooks.md)).
- `csx` prop for token-based style overrides.
- Error convention on form controls: `error: boolean` + `errorText: string`, plus `helpText`.

## Layout

- **Page** — pattern for a complete admin screen. Subcomponents: `Page` (`layout`: wide/standard/narrow), `PageHeader` (`onPopNavigation` back button), `PageHeaderTop`, `PageHeaderTitle`, `PageHeaderTags`/`PageHeaderTag`, `PageHeaderActions` with `PageHeaderButton`/`PageHeaderMenuButton`, `PageHeaderBottom`, `PageContent`. Use as the top-level structure of every screen.
- **Flex** — flexbox container with shorthand props (`direction`, `wrap`, `basis`, `shrink`, `grow`, `align`, `justify`, `order`), all responsive. `FlexSpacer` creates adjustable space between children.
- **Grid** — CSS grid: `gap`, `rowGap`, `columnGap`, `templateAreas`, `templateRows`, `templateColumns` (responsive). `GridItem` takes `area`.
- **Stack** — stacks children with consistent spacing: `direction` (column default), `space`, `fluid`, `align`; responsive values.
- **Columns** — 12-column flexbox grid. `Columns` takes `space` (default `$m`); `Column` takes `units` (1-12) and `offset`.
- **Inline** — children in a row with equal spacing: `hSpace`/`vSpace` (default `$s`), `noWrap`, `align`.
- **Center** — centers its child; the child needs explicit width/height.
- **Bleed** — lets a child trespass the parent's padding (negative margins): `top`/`bottom`/`left`/`right` space tokens.
- **Card** — container for a focused task. Subcomponents: `CardHeader`, `CardInfo`, `CardTitle`, `CardActions`, `CardContent`, `CardImage`. Limit nesting to 3 levels; avoid primary buttons inside cards.
- **Divider** — accessible `hr`: `orientation` (horizontal default, vertical).

## Data display

- **DataView** — renders and controls a data set inside `PageContent`. `DataView` takes `state` from `useDataViewState()` (statuses: ready, loading, empty, error, not-found, with default illustrations for non-ready). `DataViewHeader` holds `Search`, filters, and `DataViewActions`. Wrap `Table` with it.
- **Table** — item list with columns and actions. Composable: `Table`, `THead`, `THeadCell`, `TBody`, `TBodyRow`, `TBodyCell` driven by `useTableState({ columns, items, status, sort, length })` which returns `data`, `getTable()`, `getHeadCell()`, `getBodyCell()`. Declare columns with the typed helper `createColumns<RowType>([{ id, header, resolver? }, ...])`, not a raw array (a raw array typechecks poorly and needs manual `useMemo`; `createColumns` doesn't). Column resolvers: plain, text, menu, currency, date, image, bulk, selection, root. Always wrap with `DataView`. Default row height 44px.
- **Pagination** — `Pagination` takes `state` from `usePaginationState({ pageSize, total })` or `useQueryPaginationState` (URL-synced), plus `loading`. Admin standard is 25 items/page; position top-right; avoid user-customizable page size; keep state in the URL, not localStorage.
- **Search** — filter input: `value`, `onChange`, `onClear`, `loading`, `disabled`; `useSearchState`/`useQuerySearchState` expose `getInputProps()`. Unique `aria-label` per Search; position top-left; keep previous results visible while loading; placeholder ≤36 chars; prefer Filters over search operators.
- **Filter** — criteria selection: `FilterGroup` + `FilterDisclosure` + `FilterPopover` + `FilterListbox` with `FilterOptionRadio` (single, `useFilterState`) or `FilterOptionCheckbox` (multiple, `useFilterMultipleState`); `FilterSearchbox` when options exceed five; always include `FilterFooter`. Store filter state in the URL; don't pre-select values.
- **BulkActions** — actions over selected items: `useBulkActions({ pageItems, totalItems, pageSize })` + `BulkActions state={...}`. Max 4 actions, tertiary/criticalTertiary buttons with icons; overlays the bottom of the DataView; confirm multi-item actions with a Modal.
- **Tag** — read-only colored label: `label` (required), `variant` (lightBlue default, green, red, orange, cyan, teal, purple, gray, pink), `size`. Non-interactive; <30 chars; gray = unclassified, red = negative, green = positive, orange = pending action.
- **Avatar** — first letter of `label` in a colored circle; `palette` (lightBlue default, green, orange, cyan, purple, teal, grey).

## Forms

- **Form** — from `@vtex/admin-ui-form` (separate package, install it explicitly). `Form` takes `state` from `useFormState()` and `onSubmit`. Form-aware inputs take `state`, `name`, `validation`. Utilities: `useWatch`, `Watch`, and `watch()`/`register()`/`setValue()`/`getValues()` on the form state.
- **TextInput** — single-line input: `label` (required), `value`/`onChange`, `helpText`, `error`/`errorText`, `prefix`/`suffix` (text only). Avoid placeholders (use helpText); validate on blur/submit, not while typing.
- **TextArea** — multi-line input: `label`, `helpText`, `error`/`errorText`, `maxLength`, `disabled`.
- **NumberInput** — numeric input with stepper: `value`, `onChange` (receives the new value directly), `step`, `min`/`max`, `label`, `error` props. Always provide a default value; intended for small numbers; put units in the label.
- **Select** — native single-choice dropdown: `label` (required), `value`/`onChange`, `helpText`, `error`/`errorText`, `optional`, `disabled`; options as `<option>` children. Prefer Radio when there are fewer than 4 options.
- **Dropdown** — compact single-selection control: `variant` (solid, soft, text), `items`, `renderItem`, `itemToString`, `state` from `useDropdownState`. Built on Downshift.
- **Checkbox** — multiple independent choices (≤5): `Checkbox` (`label`, `value`, `state`, `helpText`, `error`/`errorText`) inside `CheckboxGroup` (`label`, `direction`, `optional`), state via `useCheckboxState({ initialValue })`. Use the group even for one checkbox; indeterminate state for "select all".
- **Radio** — single choice among ≤5 options: `Radio` (`value`, `label`) inside `RadioGroup` (`state` from `useRadioState`, `label`, `direction`, `error` props). Never standalone; more than 5 options means Select.
- **Switch** — instant on/off: `label` (required), `checked`, `helpText`, `disabled`; `useSwitchState` for groups. `aria-label` when standalone; never show errors on a switch, disable incompatible settings instead; label starts with an infinitive verb.
- **Label** — plain reset-styled `<label>`; rarely needed since controls take a `label` prop.

## Feedback

- **Alert** — persistent notification: `variant` (info default, positive, warning, critical), `onDismiss`, `action`. Full width below the header; one sentence, max 240 chars; one Alert at a time (summarize). Use Toast for auto-dismissing feedback.
- **Toast** — temporary feedback via the `useToast()` hook (no JSX component): `showToast({ message, variant, dismissible, duration, action, key })`. Defaults: info, dismissible, 10s. Requires `ToastProvider` at the root (inside `ThemeProvider`). Max 240 chars.
- **Spinner** — indeterminate loading: `size` (default 24px); inherits `currentColor`. Prefer Skeleton for block-level loading.
- **Skeleton** — content-shaped loading placeholder: `shape` (rect default, circle), `csx`; fluid by default. Only for dynamic content.
- **Tooltip** — hover clarification: `text` (required), `placement` (top default), `visible`, custom trigger via `children`. Use a Popover instead if content exceeds 240 chars or needs interaction; don't repeat the label.

## Navigation and overlay

- **Modal** — overlay window driven by `useModalState`. Subcomponents: `ModalHeader`/`ModalTitle`/`ModalDismiss`/`ModalContent`/`ModalFooter`/`ModalButton` (`dismissModal` closes on click). Sizes: small (confirmations), medium, large. Open/close only on user action; Cancel left, submit right; short titles.
- **Menu** — contextual actions: `MenuButton` (`variant`, `size`, `label`, `labelHidden`) + `Menu` (`state` from `useMenuState()`, always set `aria-label`) + `MenuItem` (`label`, `icon`, `critical` for irreversible actions) + `MenuDivider`. Disable items, never the MenuButton.
- **Tab** — up to five same-hierarchy views: `TabList` (takes `state` from `useTabState`) with `Tab` children, matched by `id` to `TabPanel`s. No icons in tabs; no nested tabs; no page reload on switch.
- **Toolbar** — WAI-ARIA toolbar: `Toolbar` (`state` from `useToolbarState`, always `aria-label`), `ToolbarButton`, `ToolbarItem` (focusable elements only). Arrow-key and Home/End navigation.
- **Collapsible** — build-your-own with the `useCollapse` hook: `getToggleProps()` on the trigger, `getCollapseProps()` on the content wrapper; options `visible`, `onCollapseStart/End`, `onExpandStart/End`.
- **Anchor** — inline text link with standard `<a>` attributes. Links only; use Button for actions; link text should reflect the destination page title.
- **Button** — primary action trigger: `variant` (primary, secondary, tertiary, critical, criticalSecondary, criticalTertiary, neutralTertiary), `size` (normal, large), `icon` + `iconPosition`, `disabled`, `loading`. Icon-only buttons need `aria-label`; one visible primary action at a time; `loading` for 1-5s actions; no tooltips on disabled buttons.

## Typography and accessibility

- **Heading** — semantic headings; hierarchy via the `HeadingLevel` context wrapper (the `as` prop is deprecated); `level` 1-6.
- **Text** — styled `<span>`: `variant` (pageTitle, title1, title2, action1, action2, display, body default, detail), `tone` (primary default, secondary, info, critical, warning, positive), `as`.
- **Paragraph** — reset-styled `<p>`.
- **List** — DEPRECATED. Docs: use plain `<ul>`/`<ol>` elements instead.
- **VisuallyHidden** — hides content visually, keeps it for screen readers; extends `span` props.

## General notes from the docs

- The docs (v0.136.0) state the library is "still in development and available only for internal usage at VTEX"; in practice it is the design system of the updated Admin (`adminVersion: 4`) and is consumed by IO admin apps.
- Slugs on the docs site match the kebab-case component name (`/components/number-input`, `/components/data-view`, `/components/visually-hidden`).
