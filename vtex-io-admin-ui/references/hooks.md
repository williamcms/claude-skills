# Admin UI hooks

Source: https://admin-ui.vercel.app/documentation/developing/hooks

The hooks documentation page covers one general-purpose hook (`useMediaQuery`). All other hooks are state hooks documented alongside their components; they are cataloged below because they are the standard way to control Admin UI components.

## useMediaQuery

CSS media query hook: listens for matches and enables conditional rendering.

```javascript
import { useMediaQuery } from '@vtex/admin-ui'
```

- Parameter: `string | string[]` (required), a single media query or an array of queries.
- Returns: `boolean[]`, one match result per query passed.

## Component state hooks

All exported by `@vtex/admin-ui` (form hooks by `@vtex/admin-ui-form`). Pattern: call the hook, pass the returned `state` to the component. Full usage per component in [components.md](components.md).

| Hook | Controls | Notes |
|------|----------|-------|
| `useDataViewState()` | `DataView` | Statuses: ready, loading, empty, error, not-found; default illustrations for non-ready states |
| `usePaginationState({ pageSize, total })` | `Pagination` | Local state; standard page size is 25 |
| `useQueryPaginationState()` | `Pagination` | Same, synced to URL query params |
| `useSearchState({ timeout })` | `Search` | Exposes `getInputProps()`, debounced value |
| `useQuerySearchState()` | `Search` | Same, synced to URL query params |
| `useTableState({ columns, items, status, sort, length })` | `Table` | Returns `data`, `getTable()`, `getHeadCell()`, `getBodyCell()`; column resolvers: plain, text, menu, currency, date, image, bulk, selection, root |
| `useModalState()` | `Modal` | Open/close state |
| `useDropdownState({ items, initialSelectedItem })` | `Dropdown` | Built on Downshift `useSelect` |
| `useTabState({ defaultSelectedId })` | `Tab`/`TabList`/`TabPanel` | |
| `useCheckboxState({ initialValue })` | `Checkbox` group | |
| `useRadioState({ defaultValue })` | `RadioGroup` | |
| `useSwitchState()` | `Switch` groups | For switches sharing state |
| `useMenuState()` | `Menu`/`MenuButton` | |
| `useToolbarState()` | `Toolbar` | |
| `useFilterState()` | `Filter` (single selection) | |
| `useFilterMultipleState()` | `Filter` (multiple selection) | |
| `useFilterGroupState()` | `FilterGroup` | |
| `useBulkActions({ pageItems, totalItems, pageSize })` | `BulkActions` | |
| `useCollapse({ visible, onCollapseStart, onCollapseEnd, onExpandStart, onExpandEnd })` | Custom collapsibles | Returns `getToggleProps()`, `getCollapseProps()`, `visible`, `toggle()` |
| `useToast()` | Toast notifications | Returns a `showToast` function; requires `ToastProvider` |
| `useFormState()` (from `@vtex/admin-ui-form`) | `Form` | Plus `useWatch(form, name)` and the `Watch` component |

## Experimental hooks

`experimental_`-prefixed hooks (e.g. `experimental_useComboboxState`, `experimental_useDatePickerState`) follow the experimental policy: no semver guarantees, pin the exact package version. See [experimental.md](experimental.md).

## Rule of thumb

Never hand-roll state that a component's hook already manages (open/close, selection, pagination, debounced search). The hooks also carry the accessibility wiring.
