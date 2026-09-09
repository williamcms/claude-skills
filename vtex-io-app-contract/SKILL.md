---
name: vtex-io-app-contract
description: "Apply when defining or changing the contract of a VTEX IO app through manifest.json, builder declarations, dependencies, peerDependencies, billingOptions, and app identity. Covers how the app declares capabilities and integration boundaries. Use for scaffolding apps, splitting responsibilities across apps, or fixing contract-level link and publish issues."
---

# App Contract & Builder Boundaries

## When this skill applies

Use this skill when the main decision is about what a VTEX IO app is, what capabilities it declares, and which integration boundaries it publishes through `manifest.json`.

- Creating a new VTEX IO app and defining its initial contract
- Adding or removing builders to match app capabilities
- Choosing between `dependencies` and `peerDependencies`
- Deciding whether a capability belongs in the current app or should move to another app
- Troubleshooting link or publish failures caused by manifest-level contract issues

Do not use this skill for:
- service runtime behavior such as `service.json`, memory, workers, or route exposure
- HTTP handler implementation, middleware composition, or event processing
- GraphQL schema, resolver, or data-fetching implementation
- storefront, admin, or render-runtime frontend behavior
- policy modeling and security boundary enforcement

## Decision rules

- Treat `manifest.json` as the app contract. It declares identity, builders, dependencies, peer dependencies, and high-level capabilities that other apps or the platform rely on.
- Add a builder only when the app truly owns that capability. Builders are not placeholders for future work.
- Keep the contract narrow. If a manifest starts to represent unrelated concerns, split those concerns into separate apps instead of creating a catch-all app.
- Use `dependencies` only for apps that can be safely auto-installed as part of the current app contract. Use `peerDependencies` for apps that must already exist in the environment, remain externally managed, or declare `billingOptions`.
- Keep naming and versioning publishable: `vendor`, `name`, and `version` must form a stable identity that can be linked, published, and consumed safely.
- Keep `billingOptions` aligned with the commercial contract of the app. If the app has billing implications, declare them explicitly in the manifest rather than leaving pricing behavior implicit.
- Apps that declare `billingOptions` cannot be consumed through `dependencies`. If the current app requires a billable app, model that relationship with `peerDependencies` and require manual installation by the account or edition owner.
- Edition apps are compositions of app contracts, not exceptions to them. Keep each underlying app contract explicit, narrow, and semver-safe so composition stays predictable across host environments.
- `manifest.json` can also declare app-level permissions and configuration surfaces, but detailed policy modeling belongs in security-focused skills and detailed `settingsSchema` design belongs in app-settings skills.

This is not an exhaustive list of builders; see the official Builders docs for the full catalog.

Builder ownership reference:

| Builder | Own this builder when the app contract includes |
|---------|--------------------------------------------------|
| `node` | backend runtime capability owned by this app |
| `graphql` | GraphQL schema exposure owned by this app |
| `react` | React bundles owned by this app |
| `admin` | Admin UI surfaces owned by this app |
| `store` | Store Framework block registration owned by this app |
| `messages` | localized message bundles owned by this app |
| `pixel` | storefront pixel injection owned by this app |
| `masterdata` | Master Data schema assets owned by this app |

Contract boundary heuristic:
1. If the capability is shipped, versioned, and maintained with this app, declare its builder here.
2. If the capability is only consumed from another app, declare a dependency instead of duplicating the builder.
3. If the capability introduces a separate runtime, security model, or release cadence, consider splitting it into another app.

## Hard constraints

### Constraint: Every shipped capability must be declared in the manifest contract

If the app ships a processable VTEX IO capability, `manifest.json` MUST declare the corresponding builder. Do not rely on folder presence alone, and do not assume VTEX IO infers capabilities from the repository structure.
Symmetrically, do not declare builders for capabilities that are not actually shipped by this app.

**Why this matters**

VTEX IO compiles and links apps based on declared builders, not on intent. If the builder is missing, the platform ignores that capability and the app contract becomes misleading. The app may link partially while the expected feature is absent.

**Detection**

If you see a maintained `/node`, `/react`, `/graphql`, `/admin`, `/store`, `/messages`, `/pixel`, or `/masterdata` directory, STOP and verify that the matching builder exists in `manifest.json`. If the builder exists but the capability does not, STOP and remove the builder or move the capability back into scope.

**Correct**

```json
{
  "vendor": "acme",
  "name": "reviews-platform",
  "version": "0.4.0",
  "builders": {
    "node": "7.x",
    "graphql": "1.x",
    "messages": "1.x"
  }
}
```

**Wrong**

```json
{
  "vendor": "acme",
  "name": "reviews-platform",
  "version": "0.4.0",
  "builders": {
    "messages": "1.x"
  }
}
```

The app ships backend and GraphQL capabilities but declares only `messages`, so the runtime contract is incomplete and the platform ignores the missing builders.

### Constraint: App identity and versioning must stay publishable and semver-safe

The `vendor`, `name`, and `version` fields MUST identify a valid VTEX IO app contract. Use kebab-case for the app name, keep the vendor consistent with ownership, and use full semantic versioning.

**Why this matters**

Consumers, workspaces, and release flows rely on app identity stability. Invalid names or incomplete versions break linking and publishing, while identity drift creates unsafe upgrades and hard-to-debug dependency mismatches.

**Detection**

If you see uppercase characters, underscores, non-semver versions, or vendor/name changes mixed into unrelated work, STOP and validate whether the change is intentional and release-safe.

**Correct**

```json
{
  "vendor": "acme",
  "name": "order-status-dashboard",
  "version": "2.1.0"
}
```

**Wrong**

```json
{
  "vendor": "AcmeTeam",
  "name": "Order_Status_Dashboard",
  "version": "2.1"
}
```

This identity is not safely publishable because the name is not kebab-case and the version is not valid semver.

### Constraint: Major version bumps on content-holding apps invalidate stored content

If the app ships any of `store/blocks.json`, `store/routes.json`, `store/templates/`, or `store/contentSchemas.json` (i.e. it is a **content-holding app** in the Store Framework sense), the `version` field MUST be treated as merchant-facing. A `vtex release major` on this app changes the key prefix that `vtex.pages-graphql` uses to look up every Site Editor edit, every custom route, and every per-template render cache.

**Why this matters**

`vtex.pages-graphql` keys merchant content by `vendor.app@MAJOR.x:template`. A patch and a minor bump preserve the key. A major bump changes it. After the major bump, none of the previously authored content is visible to the resolver under the active major key, and the storefront falls back to default `vtex.store-theme` content. The official recovery path is the `updateThemeIds` mutation in `vtex.pages-graphql@2.x`, executed in a production-flag dev workspace after installing the new major and before promoting; it rekeys all Site Editor edits, Pages, and Redirects from the old major to the new one.

**Detection**

Before approving `vtex release major` on an app whose manifest declares `"store"` in `builders` (or that ships any file under `store/`), STOP and require an explicit `updateThemeIds` migration plan in a production-flag dev workspace. The same rule applies before installing any new MAJOR of such an app on a production account.

**Correct**

```bash
# patch and minor preserve content keys; safe for content-holding apps
vtex release patch
vtex release minor
```

**Wrong**

```bash
# major bump on a content-holding app installed straight to master:
# every Site Editor edit becomes invisible to the resolver and the
# storefront falls back to default vtex.store-theme content. The fix
# is to install the new major in a production-flag dev workspace and
# run updateThemeIds in vtex.pages-graphql@2.x before promote.
vtex release major
vtex publish
vtex install acme.store-theme@5.0.0
```

See `vtex-io-storefront-theme-versioning` for the safe install, `updateThemeIds` migration, smoke-test, promote, and rollback flow for content-holding apps.

### Constraint: Dependencies and peerDependencies must express installation intent correctly

Use `dependencies` only for apps that this app should install as part of its contract and that can be auto-installed safely. Use `peerDependencies` for apps that must already be present in the environment, should remain externally managed, or declare `billingOptions`.

**Why this matters**

This is the core contract boundary between your app and the rest of the VTEX IO workspace. Misclassifying a relationship causes broken installations, hidden coupling, and environment-specific behavior that only appears after link or publish. In particular, builder-hub rejects dependencies on apps that declare `billingOptions`.

**Detection**

If the app requires another app to function in every environment, STOP and confirm whether it belongs in `dependencies` or `peerDependencies`. If the target app declares `billingOptions`, STOP and move it to `peerDependencies`. If the app only integrates with a platform capability, host app, edition-managed app, or paid app that the account is expected to manage manually, STOP and keep it out of `dependencies`.

**Correct**

```json
{
  "dependencies": {
    "vtex.search-graphql": "0.x"
  },
  "peerDependencies": {
    "vtex.store": "2.x",
    "vtex.paid-app-example": "1.x"
  }
}
```

**Wrong**

```json
{
  "dependencies": {
    "vtex.store": "2.x",
    "vtex.paid-app-example": "1.x"
  },
  "peerDependencies": {}
}
```

This contract hard-installs a host app that should usually be externally managed and also attempts to auto-install a billable app, which builder-hub rejects.

### Constraint: Pin actively-maintained transitive packages in `resolutions` to an exact version, not a range

When a `resolutions` entry exists to work around a Node-engine or syntax incompatibility, pin the exact version (no `^`/`~`) unless the package is abandoned or the incompatible release was a major-version bump already excluded by the range.

**Why this matters**

A caret/tilde range still lets a fresh `yarn.lock` pick the newest patch satisfying it. Packages that never stop shipping patches (`node-releases`, `browserslist`, `caniuse-lite`, and similar) can silently raise their own `engines.node` requirement, or the syntax level of their published build, in a later patch still inside the same range. The `resolutions` entry keeps "working" on paper while a clean install quietly regresses.

**Detection**

If a `resolutions` entry pins a non-VTEX package with `^`/`~` to fix an `engines.node` or build-syntax incompatibility (as opposed to a peer-dependency conflict), STOP and switch to an exact version. Before trusting any test of whether a `resolutions` entry is still needed, delete `node_modules` and the lockfile and reinstall from scratch — a plain `yarn install` reuses the lockfile's already-resolved versions and hides whether the range would float upward on a clean install.

**Correct**

```json
{
  "resolutions": {
    "node-releases": "2.0.14",
    "graphql": "^16.6.0"
  }
}
```

`node-releases` is exact because it ships frequent patches that keep tightening `engines.node`. `graphql` is safe with a caret because the incompatible jump was the `17.0.0` major, already outside `^16.6.0`.

**Wrong**

```json
{
  "resolutions": {
    "node-releases": "^2.0.14"
  }
}
```

`^2.0.14` still resolves to whatever the newest `2.x` patch is at install time (e.g. `2.0.54`), which may already require a Node version the target builder's sandbox doesn't have. The pin gives a false sense of safety.

### Constraint: Builder-hub can run a different Node/TypeScript version per builder major, even within the same app

Each builder major (`node: 6.x` vs `7.x`, `react: 3.x`, etc.) compiles and installs in its own builder-hub sandbox, with its own Node.js runtime and, for `node`, its own bundled TypeScript expectation. Bumping one builder's major in `manifest.json` does not change the sandbox used by a different builder in the same app.

**Why this matters**

An old `react: 3.x` builder can still run on Node 16 long after `node` moves to `7.x` on a newer Node. A dependency that is fine for one builder can break another, and the failure signatures look different enough to get misdiagnosed as unrelated bugs:

- `yarn install --frozen-lockfile` failing with `error <pkg>: The engine "node" is incompatible with this module` — an installed package's own `engines.node` excludes the builder's sandbox Node version. Fix with an exact-version `resolutions` pin (see the constraint above).
- `tsc` failing with syntax errors inside a `.d.ts` file under `node_modules` — the builder's TypeScript can't parse a transitive `@types/*` package's newer syntax. See `payment-provider-framework`'s TS 3.9.7 constraint for the full pattern.
- Webpack failing with `Module parse failed: Unexpected token` inside a `node_modules/**/*.mjs` file, even though the same package's CommonJS build (its `main` field) is clean — webpack prefers a package's `module`/`exports.default` (ESM) build over `main` (CJS) by default, and only the ESM build was rebuilt with newer syntax (optional chaining, nullish coalescing). Check the package's `main` vs `module`/`exports` fields and its version history for the last release whose ESM build predates the newer syntax, then pin that exact version.

**Detection**

When a build fails only after adding or upgrading a devDependency, identify which builder failed (the error is tagged, e.g. `[node@7]` / `react@3.x builder`) and which of the three signatures above matches, before assuming the fix belongs to one specific builder's config.

**Correct**

```json
{
  "resolutions": {
    "node-releases": "2.0.14",
    "react-collapsed": "3.6.0"
  }
}
```

Both pinned exactly: `node-releases` for an `engines.node` mismatch, `react-collapsed` because its `3.7.0+` ESM build introduced optional chaining that an older builder's webpack can't parse (its `main`/CJS build stayed clean).

**Wrong**

Assuming a devDependency bump that works locally — where Node is usually much newer than any builder-hub sandbox — will also work on `vtex link`/`vtex publish`, and skipping a `resolutions` check entirely.

## Preferred pattern

Start by deciding the smallest useful contract for the app, then declare only the identity and builders required for that contract.

Recommended manifest for a focused service-plus-GraphQL app:

```json
{
  "vendor": "acme",
  "name": "reviews-platform",
  "version": "0.1.0",
  "title": "Reviews Platform",
  "description": "VTEX IO app that owns review APIs and review GraphQL exposure",
  "builders": {
    "node": "7.x",
    "graphql": "1.x",
    "messages": "1.x"
  },
  "billingOptions": {
    "type": "free"
  },
  "dependencies": {
    "vtex.search-graphql": "0.x"
  },
  "peerDependencies": {
    "vtex.store": "2.x"
  }
}
```

Recommended contract split:

```text
reviews-platform/
├── manifest.json        # identity, builders, dependencies, peerDependencies
├── node/                # backend capability owned by this app
├── graphql/             # GraphQL capability owned by this app
└── messages/            # app-owned translations

reviews-storefront/
├── manifest.json        # separate release surface for storefront concerns
├── react/
└── store/
```

Use this split when the backend/API contract and the storefront contract have different ownership, release cadence, or integration boundaries.

## Baseline repository files

Every new VTEX IO app repository should ship these root files in addition to `manifest.json` and the builder folders. Use the contents below (taken from a production app) as the model when scaffolding; adapt only the app name and author.

`.vtexignore` — keeps non-app files out of `vtex link`/`vtex publish` uploads:

```text
.DS_Store
.git
node_modules/
package.json
.vscode/
.gitignore
tsconfig.json
tslint.json
typings.d.ts
yarn.lock
render/node_modules/
service/node_modules/
react/__tests__/**
react/.babelrc
react/.eslintrc
react/setupTests.js
react/node_modules/
cypress
```

`.prettierrc`:

```json
{
  "$schema": "http://json.schemastore.org/prettierrc",
  "printWidth": 120,
  "tabWidth": 2,
  "useTabs": false,
  "semi": false,
  "singleQuote": true,
  "jsxSingleQuote": false,
  "quoteProps": "as-needed",
  "trailingComma": "es5",
  "bracketSpacing": true,
  "bracketSameLine": false,
  "arrowParens": "always"
}
```

`.eslintrc`:

```json
{
  "extends": ["vtex", "prettier"],
  "root": true,
  "env": {
    "node": true
  }
}
```

`.eslintignore`:

```text
node_modules/
coverage/
*.snap.ts
```

Root `tsconfig.json` (builder folders have their own tsconfigs; this one covers root-level tooling and editors):

```json
{
  "compilerOptions": {
    "target": "es2019",
    "module": "commonjs",
    "moduleResolution": "node",
    "allowJs": false,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "skipLibCheck": true,
    "strict": true,
    "esModuleInterop": true
  },
  "typeAcquisition": {
    "enable": false
  },
  "include": ["./**/*.ts"],
  "exclude": ["node_modules"]
}
```

`.gitignore`:

```text
node_modules/
.DS_Store
.env
react/node_modules
react/.DS_Store
react/vendor/.DS_Store
package-lock.json
```

Root `package.json` — private, carries the lint/format toolchain that `.eslintrc`/`.prettierrc` depend on, plus the link convenience scripts (adapt `name` and `author`):

```json
{
  "private": true,
  "name": "my-app",
  "scripts": {
    "lint": "eslint --ext js,jsx,ts,tsx .",
    "format": "prettier --write \"**/*.{ts,tsx,js,jsx,json}\"",
    "start": "concurrently \"npm:vlink\"",
    "vlink": "vtex link --verbose --clean"
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{ts,js,tsx,jsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,graphql,gql}": [
      "prettier --write"
    ]
  },
  "devDependencies": {
    "@types/node": "^12.12.21",
    "@vtex/danger": "^0.2.9",
    "@vtex/prettier-config": "0.1.4",
    "concurrently": "^8.2.2",
    "eslint": "7.12.1",
    "eslint-config-vtex": "12.3.2",
    "eslint-config-vtex-react": "^6.4.0",
    "husky": "4.2.3",
    "lint-staged": "10.1.1",
    "prettier": "2.0.2",
    "typescript": "3.8.3"
  },
  "dependencies": {},
  "license": "UNLICENSED"
}
```

## Testing setup

Adding `@vtex/test-tools` to a builder folder's `package.json` is optional, but when the app has (or needs) unit tests, follow this shape rather than reinventing test config. Builder-hub never runs tests itself — this is purely local/CI tooling.

**Choosing the `@vtex/test-tools` version**: it bundles its own Jest, Babel, and (for `react`) Testing Library. Check what the SAME builder already pins for `typescript`/`@vtex/api` and match the era rather than reaching for the newest test-tools:

- `typescript` around `3.9.x` with `@vtex/api` on a `6.x` line, or `typescript` on `5.x` with `@vtex/api` on `7.x` — both stacks use `@vtex/test-tools@^1.0.0` for `node` and `@vtex/test-tools@^3.3.2` for `react` in production apps on this platform. A modernized `typescript`/`@vtex/api` pair does not mean a newer test-tools major is required.
- Either line still bundles **Jest 24/25** — neither understands a dependency's `package.json` `"exports"` map (see the `@vtex/api@7.x` note below). Don't reach for a newer `@vtex/test-tools` expecting that to fix an `"exports"`-map resolution failure; it won't, and chasing it burns a full reinstall cycle for nothing.

**Folder convention**: a flat `__tests__/<name>.test.ts` per builder, where `<name>` is the source file's own basename regardless of which subfolder it lives in. Nest only to break a basename collision (e.g. a `handlers/collections.ts` and a `resolvers/collections.ts` in the same app); don't otherwise mirror the source tree with `__tests__/handlers/`, `__tests__/utils/`, etc.

**Mocking a `node` handler's `ctx`**: build the fake context with a small factory and cast it, instead of implementing the real `ServiceContext` shape:

```typescript
function buildCtx({ collectionId, documents = [] }: { collectionId: string; documents?: unknown[] }) {
  const searchDocuments = jest.fn().mockResolvedValue(documents)

  const ctx = ({
    clients: { masterdata: { searchDocuments } },
    query: { _page: 1 },
    vtex: { route: { params: { id: collectionId } } },
    set: jest.fn(),
  } as unknown) as Context

  return { ctx, searchDocuments }
}
```

**`@vtex/api@7.x` and OpenTelemetry**: importing anything from `@vtex/api` (even just an error class like `UserInputError`) eagerly loads `@vtex/diagnostics-nodejs`, which pulls in `@opentelemetry/*` packages using `package.json` `"exports"` subpaths. The Jest bundled by `@vtex/test-tools` can't resolve those, and the whole suite fails to run (`Cannot find module '@opentelemetry/otlp-exporter-base/node-http'`). Stub only `@vtex/diagnostics-nodejs`, not all of `@vtex/api` — this keeps the real error classes, so `instanceof` assertions still test the real thing:

```typescript
jest.mock('@vtex/diagnostics-nodejs', () => {
  // Telemetry is off by default in tests, so the missing NewTelemetryClient just makes
  // @vtex/api log a harmless "Failed to initialize ..." error instead of throwing. Silence
  // it right here in the factory: it runs at the exact point @vtex/api is still loading,
  // before a beforeAll spy would even be registered.
  console.error = () => {}

  return {}
})
```

**`ctx.query` typing on `@vtex/api@7.x`**: query values are typed `string | string[] | undefined` (Node's `ParsedUrlQuery`), stricter than on `6.x`. Code that destructures a query param with a numeric default (`const { _page = 1 } = ctx.query`) needs an explicit `Number(_page)` / `Array.isArray` coercion before using it as a number or single string — the runtime behavior doesn't change (JS already coerced this), only the type-check does.

## Common failure modes

- Declaring builders for aspirational capabilities that the app does not yet own, which makes the contract broader than the real implementation.
- Using one large manifest to represent backend runtime, frontend rendering, settings, policies, and integration concerns that should be separated into multiple skills or apps.
- Putting host-level apps in `dependencies` when they should remain `peerDependencies`.
- Pinning exact dependency versions instead of major-version ranges such as `0.x`, `1.x`, or `3.x`.
- Treating `manifest.json` as a dumping ground for runtime or security details that belong in more specific skills.
- Modeling `settingsSchema` here instead of using this skill only to decide whether app-level configuration belongs in the contract at all.
- Pinning a `resolutions` entry with `^`/`~` for an actively-maintained package, then finding the same bad version reappears after the next clean install.
- Diagnosing a builder-hub build failure without first checking which builder (and its Node/TypeScript sandbox) actually threw it.
- Reaching for a newer `@vtex/test-tools` major to fix an `"exports"`-map Jest resolution failure — both the `1.x` and `3.x` lines bundle the same old Jest and neither understands `"exports"`.

## Review checklist

- [ ] Does the manifest describe only capabilities this app actually owns and ships?
- [ ] Are the baseline root files present (`.vtexignore`, `.prettierrc`, `.eslintrc`, `.eslintignore`, root `tsconfig.json`, `.gitignore`, root `package.json`)?
- [ ] Does every shipped capability have a matching builder declaration?
- [ ] Is the app identity publishable: valid `vendor`, kebab-case `name`, and full semver `version`?
- [ ] If the app has billing behavior, is `billingOptions` explicit and aligned with the app contract?
- [ ] Are `dependencies` and `peerDependencies` separated by installation intent?
- [ ] Would splitting the contract into two apps reduce unrelated concerns or release coupling?
- [ ] Are runtime, route, GraphQL implementation, frontend, and security details kept out of this skill?
- [ ] If the app ships any file under `store/`, has the merchant-facing impact of any planned major version bump been reviewed?
- [ ] If the app has unit tests, is every non-VTEX `resolutions` entry pinned to an exact version unless it's a range-safe major-bump or an abandoned line?
- [ ] Was any `resolutions` bisect done against a truly clean install (`node_modules` and lockfile deleted), not a reused lockfile?

## Related skills

- [`vtex-io-storefront-theme-versioning`](../vtex-io-storefront-theme-versioning/SKILL.md) — Use when the app declares the `store` builder and a version change must preserve or migrate Site Editor content.
- [`vtex-io-storefront-theme-app`](../vtex-io-storefront-theme-app/SKILL.md) — Use when the question is what the theme app's `store/` files should own.
- [`payment-provider-framework`](../payment-provider-framework/SKILL.md) — Has the sibling constraint for `tsc`/`.d.ts` syntax failures on the TypeScript 3.9.7 line (the compile-time counterpart to the install-time and webpack failure modes covered here).

## Reference

- [Manifest](https://developers.vtex.com/docs/guides/vtex-io-documentation-manifest) - Complete reference for `manifest.json` fields
- [Builders](https://developers.vtex.com/docs/guides/vtex-io-documentation-builders) - Builder catalog and capability mapping
- [Dependencies](https://developers.vtex.com/docs/guides/vtex-io-documentation-dependencies) - Dependency and peer dependency behavior in VTEX IO
- [Billing Options](https://developers.vtex.com/docs/guides/vtex-io-documentation-manifest#billingoptions) - How app billing behavior is declared in the manifest
- [Creating the New App](https://developers.vtex.com/docs/guides/vtex-io-documentation-3-creating-the-new-app) - App initialization flow and manifest basics
