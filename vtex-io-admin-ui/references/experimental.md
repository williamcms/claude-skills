# Experimental features

Source: https://admin-ui.vercel.app/experimental/about

Experimental features are newly released APIs still evolving from user feedback. They are NOT covered by semver: "Experimental features can introduce breaking changes on patches and minors."

There is no separate package or entrypoint. Experimental exports live in the main `@vtex/admin-ui` package with the `experimental_` prefix, typically aliased on import:

```javascript
import {
  experimental_Component as Component,
  experimental_useHook as useHook,
  experimental_utility as utility,
} from '@vtex/admin-ui'
```

## Risk mitigation (from the docs)

- Pin an exact version in `package.json` (e.g. `"@vtex/admin-ui": "0.118.0"`), never a caret/tilde range, so routine updates cannot pull breaking changes.
- Wrap experimental features in your own abstraction layer so API changes are absorbed in one place.

## Combobox

Filtered option selection with single/multiple selection, custom option rendering, and async data (loading/error states).

```javascript
import {
  experimental_ComboboxField as ComboboxField,
  experimental_ComboboxPopover as ComboboxPopover,
  experimental_useComboboxState as useComboboxState,
  experimental_ComboboxMultipleField as ComboboxMultipleField,
  experimental_ComboboxMultiplePopover as ComboboxMultiplePopover,
  experimental_useComboboxMultipleState as useComboboxMultipleState,
} from '@vtex/admin-ui'

function Single() {
  const combobox = useComboboxState({ list: [] })

  return (
    <>
      <ComboboxField id="id" label="Label" state={combobox} />
      <ComboboxPopover state={combobox} />
    </>
  )
}
```

- `useComboboxState({ list, getOptionValue, renderOption, timeoutMs })` returns state with `value`, `deferredValue`, `matches`, `selectedItem`, `setLoading`, `setError`.
- `useComboboxMultipleState` adds `shouldClearOnSelect`, `defaultSelected`, `renderTag`.
- Caveat: for non-string lists you must supply both `getOptionValue` and `renderOption`, otherwise behavior is unexpected.

## DatePicker

Date selection with formatting, validation ranges, and localization.

```javascript
import {
  experimental_DatePickerField as DatePickerField,
  experimental_DatePickerCalendar as DatePickerCalendar,
  experimental_useDatePickerState as useDatePickerState,
} from '@vtex/admin-ui'

function Example() {
  const state = useDatePickerState()

  return <DatePickerField label="Date" state={state} />
}
```

- `DatePickerField`: `state` (required), `label` (required), `helperText`, `criticalText`. `DatePickerCalendar`: `state` (required).
- Caveats: manual date typing and placeholder localization are pending features.

## Localization (I18nProvider)

Locale support for Admin UI built-in strings and RTL/LTR layout.

```javascript
import { experimental_I18nProvider as I18nProvider } from '@vtex/admin-ui'

function Example() {
  return (
    <I18nProvider locale="pt-BR">
      <RootApp />
    </I18nProvider>
  )
}
```

In IO apps, pass the runtime locale: `const { culture: { locale } } = useRuntime()`.

Localization guidance from the page:

- Leave room for at least 30% text expansion (up to 300%+ in buttons).
- Account for RTL languages and mixed-direction content.
- Make forms dynamic for country-specific address/phone formats; support non-ASCII input.
- Sort per user language, not with static single-language ordering.
- Prefer placeholder-based strings over runtime concatenation to preserve word order.

## Also under the experimental policy

- `ThemeProvider` props `experimentalTheme` and `experimentalDisabledGlobalStyles` (see [theming.md](theming.md)).
- Any other `experimental_`-prefixed export encountered in the package.
