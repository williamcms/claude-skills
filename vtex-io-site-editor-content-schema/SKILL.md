---
name: vtex-io-site-editor-content-schema
description: "Apply when defining or debugging the content schema (JSON Schema + widgets) that VTEX Site Editor uses to edit a storefront block's content, or when investigating vtex.admin-pages' Site Editor internals (ListContent/SaveContent GraphQL, the block tree, the react-jsonschema-form-based editor). Covers how a component's schema/getSchema/uiSchema controls what shows up in the sidebar and how it is edited, the widget registry (native rjsf widgets, VTEX customs IOMessage/RichText/image-uploader, and inline custom widget functions), oneOf/anyOf/allOf/dependencies for conditional fields, and the underlying GraphQL calls. Use when writing a component's content schema, adding a custom widget, or debugging why a block is not editable or a field does not render as expected."
---

# Site Editor Content Schema (vtex.admin-pages)

## When this skill applies

Use this skill when:

- Writing or reviewing a storefront component's `schema`/`getSchema()`/`uiSchema` so it edits well in VTEX Site Editor.
- Adding a custom widget (`ui:widget`) to a content schema — reusing a native name or writing an inline widget function.
- Debugging why a block does not show up as editable in the sidebar, why a field does not render the expected widget, or why `oneOf`/`dependencies` conditional fields do not appear.
- Investigating `vtex.admin-pages` internals: the `ListContent`/`SaveContent`/`DeleteContent` GraphQL operations, the block tree, or the `__provideRuntime`/`__RENDER_8_COMPONENTS__` bridges between the admin and the storefront iframe.

Do not use this skill for:

- General VTEX IO React component/block authoring unrelated to Site Editor content (see `vtex-io-react-apps` / `vtex-io-render-runtime-and-blocks`).
- Admin UI (`@vtex/admin-ui`) screens under `adminVersion: 4` (see `vtex-io-admin-ui`) — Site Editor is a separate, older admin surface (`vtex.admin-pages`, `react-jsonschema-form`-based), not Admin UI.
- Designing the messages/i18n infrastructure itself (see `vtex-io-messages-and-i18n`) — this skill only covers the `IOMessage` widget's specific behavior inside a content schema.

## Topic index

Each topic is self-sufficient. Load only what the task needs.

| Topic                                                                                                        | File                                                             |
| ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------- |
| GraphQL: `ListContent`/`SaveContent`/`DeleteContent`, providers, real variable payloads                      | [references/queries.md](references/queries.md)                   |
| Block tree: `iframeRuntime.extensions`, `__provideRuntime`, `__RENDER_8_COMPONENTS__`, the `isEditable` rule | [references/block-tree.md](references/block-tree.md)             |
| Content form: where `uiSchema` comes from, the widget registry, the `IOMessage` widget                       | [references/form-and-widgets.md](references/form-and-widgets.md) |
| `oneOf`/`anyOf`/`allOf`/`dependencies`, and authoring a custom widget function                               | [references/schema-patterns.md](references/schema-patterns.md)   |

## Decision rules

- A block only shows up as editable in the sidebar if `extension.hasContentSchema` is `true` (set upstream by the render-runtime/builder-hub) or the schema has at least one property that is not `isLayout: true`. Give every editable content property a `title` — without one, the block is silently skipped from the sidebar list (with a console warning, not an error).
- The schema's own `widget: { 'ui:widget': '<name>' }` and a component-level static `uiSchema` export can both exist; on a conflicting path, the one derived from `widget` wins. Do not maintain the same field's widget in both places.
- Reach for a native widget name (`textarea`, `color`, `date`, `select`, `radio`, `range`, `image-uploader`, etc — see [references/form-and-widgets.md](references/form-and-widgets.md)) before writing an inline custom widget function. Only fall back to `'ui:widget': (props) => <JSX/>` when no native widget covers the UX (options loaded from an API, a visual picker, conditional rendering inside the field itself).
- When writing a custom widget function, pull `SchemaField` from `registry.fields.SchemaField` and delegate to it instead of hand-rolling form wiring — it keeps the custom widget compatible with the rest of the rjsf lifecycle (validation, `formData` sync, nested schema resolution).
- For a property whose value should be locale/binding-translatable, use `'ui:widget': 'IOMessage'` and type plain literal text — only pass an existing message id when it is genuinely registered in some app's `messages/*.json` already loaded on the page; an unmatched id renders literally on the storefront, which reads as broken content.
- Use `dependencies` + `oneOf` for "field A's value decides which other fields appear." This is native JSON Schema/rjsf behavior — do not reimplement that branching in component code.
- Reach for `array`/`object` schema nesting before inventing a flat, delimiter-packed string property to hold structured content.
- A custom widget that fetches data (e.g., API-backed options) re-renders on every keystroke elsewhere in the form, not just its own — cache the result and the in-flight flag at module scope, and force one re-render via `onChange(value)` once the fetch resolves. See [references/schema-patterns.md](references/schema-patterns.md).
- Use `dependencies`/`oneOf` when a field's visibility depends on *another* field's value; branch inside the widget function itself only when it depends on the field's *own* current value — `dependencies` cannot target that.

## Common failure modes

- A content property with no `title` in its schema — silently absent from the sidebar, easy to mistake for a bug in `hasContentSchema`/`isLayout`.
- A property marked `isLayout: true` that is actually editorial content (or the reverse) — it lands in the wrong tab (style vs. content) of the editor.
- Duplicating a widget declaration in both `schema.widget` and the component's static `uiSchema` and editing only one, so a later change silently has no effect (the `schema.widget`-derived one always wins).
- Typing a plain sentence into an `IOMessage` field that happens to collide with a real message id elsewhere in the account — the storefront shows the unrelated translation instead of the typed text.
- Writing a custom widget function that ignores `registry.fields.SchemaField` and manually re-renders the field — breaks validation/error display that the rest of the form relies on (only applies when delegating to a native widget; a fully custom control like a rating/color-picker legitimately skips `SchemaField`, see [references/schema-patterns.md](references/schema-patterns.md)).
- An API-backed custom widget with no module-level cache — every unrelated keystroke in the form re-fires the fetch.
- Collapsing a fetch's "still loading" and "came back empty" states into the same placeholder — reads as an empty catalog instead of a stuck/broken request.

## Reference

- [react-jsonschema-form (rjsf) v1 source](https://github.com/rjsf-team/react-jsonschema-form/tree/main/packages/core) - underlying form library; VTEX pins `^1.0.1`
- [vtex-apps/admin-pages](https://github.com/vtex-apps/admin-pages) - source of truth for the Site Editor app itself
- [vtex-apps/render-runtime](https://github.com/vtex-apps/render-runtime) - source of `__RENDER_8_COMPONENTS__` / `__provideRuntime`
