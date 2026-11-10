---
name: vtex-io-rewriter-routing
description: "Apply when working with VTEX storefront URL resolution, dynamic routes, or the vtex.rewriter app — how a public path resolves to a catalog entity, how store-indexer writes and translates routes per binding, how product/category/search not-found routes are classified, and how redirects work. Covers the rewriter internal/redirect GraphQL (get, routes, save, delete), the InternalInput shape, routesVersion, declarer/origin/resolveAs semantics, and the hard limits on customizing a product URL. Use for route indexing issues, 404/not-found behavior, translated URLs, or changing a product/category URL."
---

# Rewriter & Storefront Routing

## When this skill applies

Use this skill when the task touches how a public storefront URL maps to a catalog entity on a VTEX IO store (Store Framework), or the behavior of `vtex.rewriter`.

- Debugging why a URL 404s, redirects, or resolves to the wrong entity
- Understanding or changing product/category/brand/search URLs
- Translated (per-binding/per-locale) URLs and their persistence
- Product/search not-found (404) classification and rendering
- Reading or writing routes through the rewriter GraphQL (internal routes, redirects)

Do not use this skill for:

- Theme `store/routes.json` custom routes and page templates — use `vtex-io-storefront-theme-app`
- The catalog events that trigger indexing — use `vtex-io-events-and-workers`
- Translated strings/messages infrastructure — use `vtex-io-messages-and-i18n`

## Key facts

- **`vtex.rewriter` is a private platform app.** You cannot clone it. You consume it over GraphQL with an `AppGraphQLClient` targeting `vtex.rewriter@1.x`, or inspect its data in the account's admin GraphQL IDE. It holds the store's **route map** in VBase.
- **Two namespaces:** `internal` (the storefront route map: public path to entity) and `redirect` (301/302 redirects).
- **An internal route binds a public `from` path to a catalog entity**, scoped by binding. The fields store-indexer actually writes (verified in `vtex.store-indexer`):

  | Field | Meaning |
  |---|---|
  | `from` | the public path, e.g. `/my-slug/p` |
  | `id` | the entity id (product id, category id, ...) |
  | `type` | page type: `product`, `category`, `brand`, `department`, `subcategory`, `notFoundProduct`, `notFoundSearch` |
  | `binding` | the binding id — routes are per binding (per locale/store) |
  | `declarer` | the app that owns the route; store-indexer uses `vtex.store@2.x` |
  | `origin` | who wrote it; store-indexer uses `<appId>:routes-indexing` |
  | `resolveAs` | the internal rewrite target (canonical tenant path); `null` under multi-language search |

  The rewriter GraphQL `InternalInput` exposes additional optional fields (dates, image/sitemap metadata, alternates); query the admin GraphQL IDE to see the full current input rather than assuming.

- **Routes are per binding.** The same entity has one route per binding, each with its own (possibly translated) `from` path. The reverse index returns one entry per binding.
- **The canonical path shape comes from `vtex.store@2.x`**, not from the rewriter. store-indexer reads `vtex.store@2.x`'s routes JSON (`dist/vtex.store-indexer/build.json`) and formats each `from` from the page type's `canonical` template. The product canonical is a single segment: `/:slug/p`.

## Rewriter GraphQL operations

Verified against the `vtex.store-indexer` rewriter client:

```graphql
# Forward: resolve a public path to its entity
query { internal { get(path: "/my-slug/p") { id type binding } } }

# Reverse index: all routes for an entity, one per binding
query { internal { routes(locator: { id: "17625", type: "product" }) { route binding } } }

# Write (upsert) one or many routes
mutation { internal { save(route: $route /* InternalInput */) { type } } }
mutation { internal { saveMany(routes: $routes) } }

# Delete by locator ({ from, binding })
mutation { internal { delete(path: $from, locator: $locator) { id } } }
mutation { internal { deleteMany(paths: $paths, locators: $locators) } }

# Redirects
mutation { redirect { save(route: $route /* RedirectInput */) { type } } }
```

The `internal` namespace also exposes a listing operation (`listInternals`) to enumerate routes rather than look one up, and `redirect` has its own `save` plus listing/delete operations. Confirm their exact arguments in the admin GraphQL IDE — this skill verifies only the operations above against the `vtex.store-indexer` client.

## Listing a store's bindings

Routes, slugs, and translations are per binding, so you usually need the store's binding ids first. From a backend IO app, register the `@vtex/api` `Tenant` client and call it:

```typescript
const tenant = await ctx.clients.tenant.info()
// tenant.bindings: Binding[]
```

Or, with no code, in the admin GraphQL IDE (install `vtex.admin-graphql-ide`, then select `vtex.tenant-graphql` in the dropdown):

```graphql
query {
  tenantInfo {
    bindings {
      id
      canonicalBaseAddress
      defaultLocale
    }
  }
}
```

Under the hood this is a Janus call, `GET /api/tenant/tenants?q={account}` (internal, not a public storefront URL). Each `Binding` carries:

- `id` — the binding id used as the `binding` field on every rewriter route
- `canonicalBaseAddress`, `alternateBaseAddresses` — the binding's host/path
- `defaultLocale`, `supportedLocales` — drive per-binding slug translation
- `defaultCurrency`, `supportedCurrencies`
- `targetProduct` — `vtex-storefront` for a storefront binding, otherwise admin/other
- `extraContext` — e.g. `extraContext.portal.salesChannel` ties the binding to a sales channel

Only `targetProduct === 'vtex-storefront'` bindings have storefront routes — filter to those before expecting rewriter internals (this is what store-indexer's `filterStoreBindings` does). Match a product to its bindings by sales channel through `extraContext.portal.salesChannel`.

## Hard constraints

### `routesVersion` is platform-managed, not an input

`routesVersion` appears on the `Internal` output but is **not** a field of `InternalInput` — you cannot set it on `save`. It is stamped by the platform and tied to the rewriter app's routes-version setting (changing that setting invalidates VBase routes). Do not treat a given `routesVersion` number as the cause of a route not serving: a working route and a broken one on the same store carry the same version, so a version mismatch is not a not-found diagnosis. Look at `type` (e.g. `notFoundProduct`) and `binding` instead.

### A product URL is a single `/:slug/p` segment — you cannot add path segments

The product canonical is `/:slug/p`. Product pages are classified at the segment-routing layer by that single-segment shape **before** theme `store/routes.json` is applied, and the PDP resolves the product by the id injected through segment-routing — the slug must be the real `linkId`. Consequences, confirmed by testing:

- You cannot make a multi-segment or custom product URL (e.g. `/dept/my-slug/p`) through the rewriter or a theme custom route — it falls through to not-found or the product listing.
- To change a product URL, change the product `linkId` in the catalog, or create a `redirect` (301) from the old path. A rewriter `internal.save` with a custom `from`/`resolveAs` does not make the PDP load the product.

### Do not try to clone or fork `vtex.rewriter`

It is a private app. Interact with it only through its GraphQL. Route-writing logic that must be customized belongs in an indexer-style app (see below), not in a fork of the rewriter.

## How store-indexer populates routes

`vtex.store-indexer` reacts to catalog broadcaster events and writes rewriter internals:

- For each active entity, for each binding, it builds the `from` path from the `vtex.store@2.x` canonical, translating the slug per binding locale via `vtex.messages`, and sets `resolveAs` to the canonical tenant path (`null` when multi-language search is on, since IS returns already-translated data).
- A **deactivated** entity is not deleted — it is written with `type: notFoundProduct` (products) or `notFoundSearch` (categories/brands/search), keeping the same `id`. That is what renders the not-found page while preserving the entity reference.
- `declarer` is `vtex.store@2.x` and `origin` is `<appId>:routes-indexing`, so store-indexer only manages the routes it owns.

To customize route or not-found behavior, add your own event-driven app that writes internals through the rewriter GraphQL, rather than editing the rewriter or store-indexer.

## Common failure modes

- Blaming `routesVersion` for a 404. Check `type` and `binding` first; version parity between working and broken routes rules it out.
- Expecting a custom `from`/`resolveAs` on a product route to change the PDP URL. The PDP loads by id via segment-routing; the slug must be the real `linkId`.
- Forgetting routes are per binding. A route that works on one binding can be missing or untranslated on another.
- Treating a deactivated product as a deleted route. It still exists as `notFoundProduct` with the id retained.
- Trying to clone `vtex.rewriter`. It is private; use its GraphQL.

## Review checklist

- [ ] Is the URL issue actually route classification (`type`) rather than `routesVersion`?
- [ ] Checked the route on the specific `binding` in question?
- [ ] For a URL change, is it a `linkId` change or a `redirect`, not a custom product `from`?
- [ ] Is custom route-writing in a separate indexer app, not a fork of rewriter/store-indexer?
- [ ] For translated URLs, is `resolveAs` consistent with the store's multi-language mode?

## Related skills

- [`vtex-io-storefront-theme-app`](../vtex-io-storefront-theme-app/SKILL.md) — theme `store/routes.json`, page templates, and custom route contexts.
- [`vtex-io-events-and-workers`](../vtex-io-events-and-workers/SKILL.md) — the catalog events that drive store-indexer and any custom indexer.
- [`vtex-io-messages-and-i18n`](../vtex-io-messages-and-i18n/SKILL.md) — how per-binding slug translation is stored and resolved.
- [`vtex-io-graphql-api`](../vtex-io-graphql-api/SKILL.md) — building the `AppGraphQLClient` used to call the rewriter.
- [`vtex-io-rootpath`](../vtex-io-rootpath/SKILL.md) — multi-binding stores with path prefixes, where per-binding routing matters most.

## Reference

- [vtex.store-indexer](https://github.com/vtex-apps/store-indexer) — public source that writes rewriter internals (route formatting, not-found classification, per-binding translation).
- Account admin GraphQL IDE (`vtex.admin-graphql-ide`) — inspect the live `vtex.rewriter` schema and query the current route map; select `vtex.tenant-graphql` to query bindings.
- [Checking your store's binding ID](https://developers.vtex.com/docs/guides/checking-your-stores-binding-id) — the `tenantInfo { bindings }` query and where to run it.
