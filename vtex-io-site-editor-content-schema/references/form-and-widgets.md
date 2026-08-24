# The content form: uiSchema origin, the widget registry, IOMessage

Source: `vtex-apps/admin-pages` (`react/components/EditorContainer/Sidebar/BlockEditor/BlockConfigurationEditor/`, `react/utils/components/index.ts`), cross-checked against the `react-jsonschema-form@1.0.1` source (`getWidget`, `widgetMap`, `widgets/index.js`).

The editor form is built with `react-jsonschema-form@1.0.1` — not a custom form engine.

## Where `uiSchema` comes from

Always from the storefront component, never hardcoded per-page in `admin-pages`. Two sources merged in `getSchemas` (`BlockConfigurationEditor/utils.ts`):
```ts
const componentUiSchema = componentImplementation?.uiSchema ?? {}         // static export from the component module
const uiSchemaFromComponent = getUiSchema(componentUiSchema, componentSchema) // derived from `schema.widget`
return mergeDeepLeft(uiSchemaFromComponent, componentUiSchema) // on conflict, the `widget`-derived one wins
```
`widget?: Record<string, string>` on `ComponentSchema` is a VTEX-specific extension, not standard JSON Schema. `getUiSchema` walks the schema (`json-schema-traverse`) looking for any node with `widget`, and writes it into `uiSchema` at the matching path — stripping `oneOf/N`, `dependencies/foo` segments from the JSON Pointer first, since `uiSchema` does not mirror that nesting (see [schema-patterns.md](schema-patterns.md)).

## Widget resolution order (`getWidget`, rjsf core)

```
1. If the "ui:widget" value is already a key in the registered widgets map, use it directly.
2. Else, look it up as a type-scoped alias in widgetMap (e.g. "textarea" -> "TextareaWidget" for type: string).
3. Else, throw ("No widget ... for type ...").
```
So both the alias (`textarea`) and its canonical name (`TextareaWidget`) work in `ui:widget` — they resolve to the same component.

## Widgets registered by admin-pages (`Form.tsx`)

```ts
export const widgets = {
  BaseInput, CheckboxWidget: Toggle, IOMessage: I18nInput, RadioWidget: Radio,
  RichText, SelectWidget: Dropdown, TextareaWidget: TextArea,
  'image-uploader': MediaGalleryWidget, // uses vtex.admin-cms/MediaGallery
}
```

| Type | Alias (`ui:widget`) | Canonical | Implementation |
|---|---|---|---|
| boolean/string | `checkbox` | `CheckboxWidget` | VTEX (`Toggle`) |
| boolean/string/number/integer | `radio` | `RadioWidget` | VTEX (`Radio`) |
| boolean/string/number/integer/array | `select` | `SelectWidget` | VTEX (`Dropdown`) |
| string | `textarea` | `TextareaWidget` | VTEX (`TextArea`) |
| string | `text`, `hostname`, `ipv4`, `ipv6` (or no `ui:widget` at all) | `TextWidget` | rjsf stock, but delegates to `registry.widgets.BaseInput` at render time → VTEX (`BaseInput`) |
| string | `hidden` | `HiddenWidget` | rjsf stock |
| string | `password` | `PasswordWidget` | rjsf stock |
| string | `email` | `EmailWidget` | rjsf stock |
| string | `uri` | `URLWidget` | rjsf stock |
| string | `data-url`, `file` | `FileWidget` | rjsf stock |
| string | `date` / `datetime`/`date-time` / `alt-date` / `alt-datetime` | `DateWidget`/`DateTimeWidget`/`AltDateWidget`/`AltDateTimeWidget` | rjsf stock |
| string | `color` | `ColorWidget` | rjsf stock |
| number/integer | `updown` / `range` | `UpDownWidget`/`RangeWidget` | rjsf stock |
| array | `checkboxes` / `files` | `CheckboxesWidget`/`FileWidget` | rjsf stock |
| — (no alias, exact name only) | `IOMessage` | — | VTEX (`I18nInput`) |
| — (no alias, exact name only) | `RichText` | — | VTEX (`RichText`) |
| — (no alias, exact name only) | `image-uploader` | — | VTEX (`MediaGalleryWidget`) |

`TextWidget` has no implementation of its own — it is `props.registry.widgets.BaseInput`, read from the live registry. Because `admin-pages` overrides the `BaseInput` key, any plain string field (no `ui:widget`, no `format`) still renders VTEX's own input.

Extra `uiSchema`-style keys observed on real schemas (community-documented, not verified in `admin-pages` source): `ui:placeholder`, `ui:inputType`, `ui:help`, `ui:options`, `ui:disabled`, `ui:readonly`, `ui:field` (a full custom field renderer, distinct from `ui:widget`), and a VTEX-specific `classNames` for Tachyons utility classes on the field wrapper.

## The `IOMessage` widget

`components/form/I18nInput/index.tsx`, a thin wrapper around `BaseInput`/`TextArea`. The field's value acts as a **message id** in the `vtex.messages` catalog: if a translation exists under that exact string, it renders; otherwise it falls back to the literal text (`translateMessage` in `utils/components/index.ts`).

Because the Site Editor preview is live, a typed value that happens to collide with a real message id would flip the preview to that translation mid-typing. To prevent that, every keystroke appends an invisible zero-width character (`​`) to the value while editing; it is stripped before saving (`removeInvisibleCharacters`). After saving, the field behaves as a normal `IOMessage` lookup again.

The lookup dictionary comes from `editorMessagesFromRuntime` (`DomainMessages/editorMessagesFromRuntime.ts`): the union of `messages/*.json` from every app/component currently rendered on the page, fetched via the `MessagesForDomain` query.

**Practical rule**: type plain literal text in an `IOMessage` field by default. Only pass an existing message id when it is verified to exist in some loaded app's message catalog — an id with no match renders literally on the storefront.

## Rule of thumb

Before writing a custom widget function for a type of input, check this table first — five of these names are VTEX overrides of rjsf's own defaults, so they already look and behave like the rest of the Admin.
