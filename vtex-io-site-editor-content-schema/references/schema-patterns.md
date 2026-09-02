# Schema authoring patterns: oneOf/dependencies, custom widgets, and more

Sources: `react-jsonschema-form@1.0.1` core behavior (verified against `vtex-apps/admin-pages`), plus patterns documented by the community project [`gblcintra/schema-ui`](https://github.com/gblcintra/schema-ui) — a VTEX IO app that demos schema/widget combinations. Anything below marked "community pattern" comes from that project, not from `admin-pages`' own source; verify against the running Site Editor before relying on it for anything load-bearing.

## `oneOf` / `anyOf` / `allOf` / `dependencies`

Not custom logic in `admin-pages` — native `react-jsonschema-form`/JSON Schema behavior. `admin-pages` only makes sure the right `widget` ends up on the right path inside these branches (see `getUiSchema`'s `arrayAttrs` regex in [form-and-widgets.md](form-and-widgets.md), which strips `oneOf/N`, `dependencies/foo` segments before writing to `uiSchema`).

- **`allOf`**: merges all subschemas (AND) as if they were one.
- **`oneOf` / `anyOf`** on a node: rjsf renders a variant selector; only the chosen branch's fields render.
- **`dependencies` + `oneOf`**: "field X decides which other fields appear." rjsf reads the current value of `X` in `formData`, matches it against each branch's `enum`, and merges that branch's `properties`/`required` in.

```json
{
  "properties": {
    "showMoreConfig": { "type": "boolean", "title": "Mostrar mais configurações" }
  },
  "dependencies": {
    "showMoreConfig": {
      "oneOf": [
        {
          "properties": {
            "showMoreConfig": { "enum": [true] },
            "numberItem": { "type": "number", "title": "Campo de número", "default": 8 },
            "numberIntegerItem": {
              "type": "integer", "title": "Campo com range", "minimum": 10, "maximum": 100,
              "widget": { "ui:widget": "range" }
            }
          }
        },
        {
          "properties": {
            "showMoreConfig": { "enum": [false] },
            "alert": {
              "type": "string", "title": "Aviso",
              "description": "Ative o campo acima para ver mais opções.",
              "widget": { "ui:field": "(schema) => <div>{schema.description}</div>" }
            }
          }
        }
      ]
    }
  }
}
```
(community pattern, `gblcintra/schema-ui`) — a boolean toggle that reveals `numberItem`/`numberIntegerItem` when checked, and an `alert` message in the `false` branch instead. Every `oneOf` needs one branch per value the discriminant (`showMoreConfig`) can take — a `oneOf` with only the `true` branch leaves the `false` state with nothing declared. (The real repo nests `alert` oddly as a sibling key inside `showMoreConfig`'s own object rather than as a sibling property in the branch — that placement does not match any documented JSON Schema/rjsf keyword, so it is corrected here rather than reproduced.) Nested `oneOf` inside another `oneOf`/`dependencies` also works, confirmed in `admin-pages`' own test suite (`BlockConfigurationEditor/utils.test.ts`).

## Reusable enums, labels, and whole-tree defaults

Source: community pattern, `gblcintra/schema-ui`; all three are standard JSON Schema, not VTEX-specific.

- **`enumNames`**: a parallel array of human-readable labels for `enum`, when the stored value itself should not be the label (e.g. storing a hex color but showing a color name):
  ```jsx
  selectItem: {
    type: 'string', title: 'Seleção de Cor', default: '#000',
    enum: ['#000', '#fff'], enumNames: ['Preto', 'Branco'],
    widget: { 'ui:widget': 'select' },
  }
  ```
- **`definitions` + `$ref`**: declare a shared enum (or any subschema) once at the schema root, reuse it across properties instead of repeating the same `enum` list:
  ```jsx
  definitions: {
    largeEnum: { enum: ['option #0', 'option #1', 'option #2', 'option #3'] },
  },
  properties: {
    reference: { type: 'string', title: 'Seleção usando definitions como referência', $ref: '#/definitions/largeEnum' },
  }
  ```
- **Root-level `default` mirroring the whole schema**: instead of a `default` per property, a single object shaped like the entire schema tree can sit on the schema's own `default`, pre-filling every field (including nested objects and array items with their own `__editorItemTitle`) in one place. Useful for seeding a brand-new block with a complete, sensible starting configuration rather than a pile of empty fields.

## Writing a custom widget function

`'ui:widget'` does not have to be a string name — `getWidget` accepts a function directly (`typeof widget === "function"`) and uses it as-is, bypassing the registry entirely. This lets a single schema property ship a fully custom field without registering anything in `admin-pages`.

Community pattern (`gblcintra/schema-ui`) for wrapping a widget function around the same `SchemaField` rjsf uses internally, so it stays wired into validation and `formData`:
```tsx
type OnChange = (val: unknown) => void
type WidgetProps<T = unknown> = {
  schema: { title?: string; description?: string; enum?: string[] }
  value: T
  onChange: OnChange
  registry: { fields: Record<string, ComponentType<any>>; widgets: Record<string, ComponentType<any>> }
}

properties: {
  customField: {
    type: 'string',
    title: 'Custom field',
    widget: {
      'ui:widget': ({ schema, value, onChange, registry }: WidgetProps) => {
        const SchemaField = registry.fields.SchemaField
        return (
          <SchemaField
            name="innerField"
            schema={{ type: 'string', title: schema.title }}
            uiSchema={{ 'ui:widget': 'image-uploader' }} // delegate rendering to a native widget
            formData={value || ''}
            registry={registry}
            onChange={(v: string) => onChange(v)}
          />
        )
      },
    },
  },
}
```
This is how you build things a native widget cannot: options fetched from an external API, a visual picker (colors, star ratings), or a field whose displayed content changes based on its own current value. The examples below (all community pattern, `gblcintra/schema-ui`) cover each of those.

**Reuse what's already there instead of manually reconstructing it.** A custom widget function usually already has access to everything `SchemaField` needs — most directly its own `schema` prop, which already *is* that property's JSON Schema node, and often module-level constants (`enum`/`enumNames` arrays, label maps) the schema itself was built from. If the property's schema already carries what the delegated field needs (`type`, `title`, `enum`, `enumNames`, ...), pass it straight through (`schema={schema}`) instead of retyping a new literal, as the examples above do (`schema={{ type: 'string', title: schema.title, enum: options, ... }}`) — and where a shared constant (an enum list, its labels) is declared once for the schema, reference that same constant from the widget instead of re-listing the values inline. Reconstructing either duplicates a single source of truth into two places that can silently drift apart (the outer schema's `enum` changes and the widget's hand-copied version doesn't). Only build something fresh when the delegated field's shape genuinely isn't available yet — e.g. the API-driven example below, where the real `enum` isn't known until the fetch resolves and can't come from the static outer schema at all.

### Dynamic options from an API, with module-level caching

A widget function re-renders on every keystroke elsewhere in the form, not just its own — fetch inside it without guarding, and it re-fires the request on every unrelated field edit. Cache the result and the in-flight state in module scope (outside the widget function, so it survives re-renders), and force one re-render when the fetch resolves by calling `onChange(value)` with the field's own current value:

```tsx
let cachedNames: string[] | null = null
let isFetching = false

properties: {
  widgetCustomSelect: {
    type: 'string',
    title: 'Widget Select Customizado',
    widget: {
      'ui:widget': ({ schema, value, onChange, registry }: WidgetProps) => {
        const SchemaField = registry.fields.SchemaField

        if (!cachedNames && !isFetching) {
          isFetching = true
          fetch('/api/catalog_system/pub/category/tree/2')
            .then(res => res.json())
            .then((data: Category[]) => {
              cachedNames = data.map(i => i.name).slice(0, 5)
              isFetching = false
              onChange(value) // forces rjsf to re-render this field now that cachedNames is populated
            })
            .catch(() => {
              cachedNames = []
              isFetching = false
            })
        }

        const options = cachedNames === null
          ? ['Carregando...']
          : cachedNames.length ? cachedNames : ['Sem opções']

        return (
          <SchemaField
            name="selectChoice"
            schema={{ type: 'string', title: schema.title, enum: options }}
            uiSchema={{ 'ui:widget': 'select' }}
            formData={value || ''}
            registry={registry}
            onChange={(option: string) => onChange(option)}
          />
        )
      },
    },
  },
}
```
Three states to design for explicitly: `null` (never fetched — show a loading placeholder), empty array (fetched, nothing came back — show a distinct "no options" placeholder, not a blank list), populated (the real options). Collapsing "loading" and "empty" into the same placeholder hides a broken endpoint behind what looks like an empty catalog.

### Custom widgets that skip `SchemaField` entirely

The `SchemaField`-delegation pattern above is for wrapping a native widget with extra logic. When the rendering itself is fully custom (no rjsf field type matches it), control `value`/`onChange` directly instead — there is no native field to delegate to:

```tsx
// Star rating (number, 1-5), no SchemaField involved
widget: {
  'ui:widget': ({ value, onChange }: WidgetProps<number>) => (
    <div className="flex justify-center">
      {[1, 2, 3, 4, 5].map(star => (
        <span
          key={star}
          className={value >= star ? 'yellow' : 'gray'}
          onClick={() => onChange(star)}
        >★</span>
      ))}
    </div>
  ),
}

// Multi-select over an array type, via checkboxes instead of a native <select multiple>
widgetCustomMultiSelect: {
  type: 'array',
  items: { type: 'string', enum: ['Opção 1', 'Opção 2', 'Opção 3'] },
  widget: {
    'ui:widget': ({ schema, value, onChange }: WidgetProps) => (
      <>
        {schema.items.enum.map((option: string) => (
          <input
            key={option}
            type="checkbox"
            checked={(value as string[])?.includes(option)}
            onChange={(e) => onChange(
              e.target.checked
                ? [...(value as string[] || []), option]
                : (value as string[]).filter((v: string) => v !== option)
            )}
          />
        ))}
      </>
    ),
  },
}
```
Same shape works for a color swatch grid (`onClick={() => onChange(hex)}` per swatch) or any other direct-manipulation control — `value`/`onChange` is the entire contract, nothing else from rjsf is required.

### Conditional rendering inside the widget itself

A widget function can branch on its own current `value` to change what it renders — useful when the condition is internal to one field, not something `dependencies`/`oneOf` (which branches on *other* fields) should own:
```tsx
widget: {
  'ui:widget': ({ schema, value, onChange }: WidgetProps) => (
    <>
      <input value={(value as string) || ''} onChange={(e) => onChange(e.target.value)} />
      {value === 'mostrar'
        ? <div>Conteúdo revelado!</div>
        : <div>Digite 'mostrar' para revelar.</div>}
    </>
  ),
}
```
Rule of thumb between the two: if the branch depends on *another* property's value, use `dependencies`/`oneOf` (declarative, admin-pages/rjsf owns it). If it depends on the field's *own* value, it has to live inside the widget function — `dependencies` cannot target a property's own value.

### Styling a custom widget: Tachyons is available

Source: verified in practice (real widget customization work), not from reading `admin-pages` source directly. A custom `ui:widget` function's JSX renders inside the Site Editor sidebar itself — `admin-pages`' own bundle, the same one the sidebar UI uses — which ships Tachyons. Any element the widget function returns can take Tachyons utility classes directly via `className`, no extra import or dependency needed:

```tsx
// help text meant to read as attached to the select/input above it
<p className="i f6 bg-muted-5 ph3 pv2 br2 mt1 mb0">
  {helpText}
</p>
```
`i` italic, `f6` font-size (0.875rem/14px — `f7` is 0.75rem/12px; check the rest of the scale against the actual bundle before assuming a value, rather than guessing), `bg-muted-5` a neutral background token, `ph3`/`pv2` padding, `br2` border-radius, `mt1 mb0` tight margins so the block reads as attached to the field above it. Same toolkit for layout (`flex`, `justify-between`, `items-center`) and semantic tokens seen elsewhere in the Admin (`c-on-base`, `bg-warning--faded`, `bg-danger--faded`).

This is a **second, separate mechanism** from the schema-level `widget: { classNames: '...' }` key (see [form-and-widgets.md](form-and-widgets.md)): `classNames` is read by `admin-pages` itself and applied to the wrapper it renders around any field (native or custom), declared in the schema with no widget function needed; a `className` inside a custom widget function's own returned JSX is plain React, styling only what that function renders. Reach for `classNames` first for simple cases; a custom widget function's own JSX is only needed when the styling target isn't the field's outer wrapper.

**Scope of this**: it only covers JSX a widget function returns for rendering *inside the Site Editor sidebar*. It says nothing about whatever the storefront-facing component itself renders for shoppers (a modal, a popup, the block's normal output) — that runs in the theme's own bundle, a different one, which does not necessarily have Tachyons loaded. Do not carry this assumption over to storefront-rendered markup without checking that bundle separately.

### `ui:field` vs `ui:widget`

`ui:widget` replaces only the input control; rjsf still renders the label, description, and error list around it. `ui:field` replaces the entire field — label, description, errors included — with a fully custom component. Reach for `ui:widget` first; only use `ui:field` when the standard label/description/error chrome itself needs to change (e.g. a `type: 'null'` property used purely to inject explanatory text into the form, with no input at all):

```tsx
// community pattern, gblcintra/schema-ui — a null-type field used only to render
// static help text between other fields, with no input at all
fieldNull: {
  title: 'Texto com type Null',
  type: 'null',
  description: 'Campos nulos como este são ótimos para adicionar informações extras',
  widget: {
    'ui:field': ({ schema }: any) => {
      return <h4 className="ma0 f5 near-black">{schema?.description}</h4>
    },
  },
}
```
Note `type: 'null'` here — there is no value to collect, so `ui:widget` (which still wraps an input) would not fit; `ui:field` takes over the whole slot, label included, and renders whatever JSX the function returns.

The same `type: 'null'` + `ui:field` combination, named `__blockItemTitle` by convention (community pattern, `gblcintra/schema-ui`), doubles as a **section header inside a nested `object` group** — placed as the first property of the group so it renders before the group's real fields:
```jsx
textProps: {
  title: 'Text Props',
  type: 'object',
  widget: { classNames: 'bg-black-20 ba b--light-gray ph3 mb3' }, // background sets the group apart visually
  properties: {
    __blockItemTitle: {
      type: 'null',
      title: 'Configuração do schema relacionados a string',
      widget: {
        'ui:field': ({ schema }: any) => <h4 className="ma0 f5 near-black">{schema?.title}</h4>,
      },
    },
    titleItem: { type: 'string', title: 'Título', /* ... */ },
    passwordItem: { type: 'string', title: 'Senha', /* ... */ },
  },
}
```
Grouping related properties under an `object` with its own `classNames` (a distinct background per group), each opening with a `__blockItemTitle` header, is how a long flat form gets visually split into labeled sections without any layout component beyond what the schema itself declares.

### Array items: `__editorItemTitle`

Verified in `admin-pages` source (`utils/components/index.ts`, `setArraySchemaDefaultsDeep`), not `schema-ui`: every `array` schema's `items.properties` gets an extra `__editorItemTitle` string property injected automatically, used as the collapsed item's label in the sidebar list. Give it a sensible `default` and a short `description` (max length guidance) — it is what a content editor sees when the array has many collapsed entries, not the first real field's value.

## Rule of thumb

Reach for `dependencies`/`oneOf` before writing a custom widget just to hide/show sibling fields — that's exactly what it is for. Reach for a custom widget function only when the *rendering* itself, not just which fields are visible, needs to differ from every native widget.
