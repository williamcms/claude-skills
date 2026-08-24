# Site Editor GraphQL: ListContent, saveContent, DeleteContent

Source: `vtex-apps/admin-pages`, `react/components/EditorContainer/graphql/*.graphql`. All three are resolved by `vtex.pages-graphql@2.x` (declared as a manifest dependency, not via an inline `@context(provider: ...)` directive — the whole operation belongs to that app's schema, unlike fields such as `redirect`, which are stitched in from `vtex.rewriter` via `@context`).

## `ListContent` (query)

Fired by `Sidebar/hooks.ts`'s `useInitialEditingState`, with `fetchPolicy: 'network-only'`, whenever `editor.editTreePath` transitions away from `null` (a block was clicked in the sidebar).

```graphql
query ListContent(
  $bindingId: String
  $blockId: String
  $pageContext: PageContextInput
  $template: String
  $treePath: String
) {
  listContentWithSchema(
    bindingId: $bindingId
    blockId: $blockId
    pageContext: $pageContext
    template: $template
    treePath: $treePath
  ) {
    content {
      condition {
        allMatches
        id
        pageContext { id type }
        statements { objectJSON subject verb }
      }
      contentId
      contentJSON
      label
      origin
    }
    schemaJSON
  }
}
```

Returns `content[]` (every content entry already saved for that block, each with its display `condition`) and `schemaJSON` (the component's `ComponentSchema`, used to build the sidebar form).

Real variables (clicking `valisere-initial-popup` inside `store.home`):
```json
{
  "bindingId": "ba57eb83-b847-4cb5-916b-41ff05f2a477",
  "blockId": "valisere.store@4.x:valisere-initial-popup#home",
  "pageContext": { "id": "store.home", "type": "route" },
  "template": "valisere.store@4.x:store.home",
  "treePath": "store.home/valisere-initial-popup#home"
}
```
`treePath` = the path clicked in the sidebar tree; `blockId` = `iframeRuntime.extensions[treePath].blockId`; `template` = `iframeRuntime.pages[iframeRuntime.page].blockId` (or `'*'` for a sitewide block); `pageContext` = `iframeRuntime.route.pageContext`; `bindingId` = the active binding in the topbar.

## `saveContent` (mutation)

```graphql
mutation saveContent(
  $bindingId: String
  $blockId: String
  $configuration: ContentConfigurationInput
  $template: String
  $treePath: String
  $lang: String
) {
  saveContent(
    bindingId: $bindingId
    blockId: $blockId
    configuration: $configuration
    template: $template
    treePath: $treePath
    lang: $lang
  ) {
    contentId
  }
}
```

Real variables (saving content for the same popup):
```json
{
  "blockId": "valisere.store@4.x:valisere-initial-popup#home",
  "template": "valisere.store@4.x:store.home",
  "treePath": "store.home/valisere-initial-popup#home",
  "configuration": {
    "condition": {
      "allMatches": true,
      "id": "be4fa4c4ff2661651aa33e656d1aa9d61e096392",
      "pageContext": { "id": "*", "type": "*" },
      "statements": []
    },
    "contentId": "kyQQ8H7p9BiQ5bPtd85g6Z~abXKaHKP2it1ixAiXE3U6t",
    "contentJSON": "{\"title\":\"CADASTRE-SE E GANHE\",\"subtitle\":\"10% OFF\", ...}",
    "label": null,
    "origin": "valisere.store@4.x:valisere-initial-popup#home"
  }
}
```
`contentJSON` is the rjsf `formData` serialized to a string. `condition.pageContext: {"id": "*", "type": "*"}` with `allMatches: true` and `statements: []` is the wildcard — "apply on any page context" — used when the editor did not restrict the content by route/device/etc.

## `DeleteContent` (mutation)

```graphql
mutation DeleteContent($template: String, $treePath: String, $pageContext: PageContextInput, $contentId: String) {
  deleteContent(template: $template, treePath: $treePath, pageContext: $pageContext, contentId: $contentId)
}
```

## Rule of thumb

`ListContent` always fires with `network-only` — never trust an Apollo cache hit here when debugging stale content in the editor; the freshest state is whatever the last `ListContent` response returned.
