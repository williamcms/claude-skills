# The block tree: extensions, the runtime bridge, and isEditable

Source: `vtex-apps/admin-pages` (`react/components/EditorProvider.tsx`, `Sidebar/BlockSelector/`) and `vtex-apps/render-runtime` (`react/utils/registerComponent.tsx`, `react/typings/global.d.ts`).

The sidebar's block list is never fetched via GraphQL. It is built entirely client-side from data the storefront's own `vtex.render-runtime` already holds in the browser.

## Where `iframeRuntime` comes from: `__provideRuntime`

`admin-pages`' `EditorProvider.tsx` exposes a global bridge function:
```ts
if (canUseDOM) {
  window.top.__provideRuntime = window.self.__provideRuntime = async (
    runtime, messages, shouldUpdateRuntime
  ) => {
    this.setState({ iframeRuntime: runtime, ... })
  }
}
```
The `store-iframe` loads the storefront's normal URL. The `render-runtime` running inside it calls `window.top.__provideRuntime(runtime, ...)` whenever its own `RenderContext` initializes or updates, handing over the full runtime object (`extensions`, `pages`, `route`, `binding`, `history`). This is a direct cross-window function call, not `postMessage` — it only works because admin and storefront share an origin (`*.myvtex.com`). `iframeRuntime` in `admin-pages` **is** the storefront's own `RenderContext`, not a copy.

## Where `__RENDER_8_COMPONENTS__` comes from

Declared in `render-runtime`'s own typings:
```ts
// render-runtime/react/typings/global.d.ts
__RENDER_8_COMPONENTS__: ComponentsRegistry
```
Populated by `registerComponent` (`render-runtime/react/utils/registerComponent.tsx`), exported publicly from the package's main module:
```ts
export default (module, InitialImplementer, app, name, lazy = false) => {
  const componentLocators = [`${app}/${name}`, `${idToAppAtMajor(app)}/${name}`]
  componentLocators.forEach((locator) => {
    window.__RENDER_8_COMPONENTS__[locator] = wrappedComponent
  })
}
```
Each component's own generated bundle calls this exported `registerComponent` when it loads in the browser, registering itself under both a full-version and a `@major.x` locator. `admin-pages`' `getIframeRenderComponents()` just reads `iframe.contentWindow.__RENDER_8_COMPONENTS__` directly — the storefront populated it incrementally as its own blocks loaded, no separate fetch needed.

## Building the tree: `getComponents` + `normalize`

`Sidebar/BlockSelector/utils.ts`:
- `getComponents(extensions, components, page)` filters which `treePath`s show up (same page, has a title, is root or is editable) and resolves each one's schema:
  ```ts
  const getSchema = path([component, 'getSchema'], { ...components })
  return path([component, 'schema'], { ...components }) || (typeof getSchema === 'function' && getSchema(extensionProps))
  ```
- `normalize` (`buildTree` + `hoistSurroundingBlocks`/`hoistSubtrees`) turns the filtered flat list into a `NormalizedBlock[]` tree, hoisting `before`/`after`/`around` blocks to the parent's level.

## The `isEditable` rule

```ts
const isEditable = extension.hasContentSchema || hasContentPropsInSchema(schema)

export const hasContentPropsInSchema = (schema) =>
  schema.type === 'object' &&
  Object.values(schema.properties || {}).some(
    property => !property.isLayout || (property.type === 'object' && hasContentPropsInSchema(property))
  )
```
- `extension.hasContentSchema` is a flag the render-runtime/builder-hub already sets per extension point.
- Otherwise, walk `schema.properties` recursively: if any property is not `isLayout: true` (including inside a nested object), the block is editable.
- A block also needs a title (`extension.title` or `schema.title`) to actually render in the sidebar list — titleless schemas log `generateWarningMessage` and are skipped.

## Clicking a block: `editExtensionPoint`

```ts
public editExtensionPoint = (treePath) => {
  const iframe = document.getElementById('store-iframe') || {}
  const setHighlightTreePath = path(['contentWindow', '__setHighlightTreePath'], iframe)
  if (setHighlightTreePath) setHighlightTreePath({ openBlockTreePath: treePath })
  this.setState({ editTreePath: treePath, editMode: false })
}
```
Sets `editTreePath` in the shared `EditorContext` and highlights the block inside the iframe preview. `Sidebar`'s `useInitialEditingState` reacts to `editTreePath` leaving `null` by firing `ListContent` (see [queries.md](queries.md)) — no network call happens before this click.

## Rule of thumb

If a block silently does not appear in the sidebar, check `title` first, then `isLayout`/`hasContentSchema` — in that order, that is the actual filter chain.
