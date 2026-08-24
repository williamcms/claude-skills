# oneOf/anyOf/allOf/dependencies, and writing a custom widget function

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
            "numberItem": { "type": "number", "title": "Campo de número", "default": 8 }
          }
        }
      ]
    }
  }
}
```
(community pattern, `gblcintra/schema-ui`) — a boolean toggle that reveals `numberItem` only when checked. Nested `oneOf` inside another `oneOf`/`dependencies` also works, confirmed in `admin-pages`' own test suite (`BlockConfigurationEditor/utils.test.ts`).

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

### Array items: `__editorItemTitle`

Verified in `admin-pages` source (`utils/components/index.ts`, `setArraySchemaDefaultsDeep`), not `schema-ui`: every `array` schema's `items.properties` gets an extra `__editorItemTitle` string property injected automatically, used as the collapsed item's label in the sidebar list. Give it a sensible `default` and a short `description` (max length guidance) — it is what a content editor sees when the array has many collapsed entries, not the first real field's value.

## Rule of thumb

Reach for `dependencies`/`oneOf` before writing a custom widget just to hide/show sibling fields — that's exactly what it is for. Reach for a custom widget function only when the *rendering* itself, not just which fields are visible, needs to differ from every native widget.
