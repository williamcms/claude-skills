# Admin builder wiring for Admin UI (adminVersion 4)

How a VTEX IO app exposes pages in the updated Admin experience. Verified against a production admin app using `@vtex/admin-ui`.

Official docs: [VTEX Admin overview](https://help.vtex.com/en/tutorial/vtex-admin-start-here--531cHtUCUi3puRXNDmKziw) · [Admin Applications course](https://learn.vtex.com/docs/course-admin-lang-en)

## manifest.json

The app needs the `admin` builder plus `react` for the page components. `messages` internationalizes navigation labels; `node`/`graphql` only when the app has a backend.

```json
{
  "builders": {
    "admin": "0.x",
    "react": "3.x",
    "messages": "1.x"
  }
}
```

## admin/navigation.json

`adminVersion: 4` is what opts the app into the updated Admin. Without it the link renders in the legacy Admin experience.

```json
{
  "adminVersion": 4,
  "section": "storeSettings",
  "subSection": "emailTemplates",
  "subSectionItems": [
    {
      "labelId": "admin/my-app.navigation.label",
      "path": "/admin/my-app"
    }
  ]
}
```

- `section` / `subSection`: where the link appears in the Admin navigation tree. Valid values are listed in [Valid sections and subSections](#valid-sections-and-subsections) below.
- `subSectionItems`: the array of links the app contributes to that subSection group; each item renders as one entry in the Admin sidebar (one app can register several pages by listing several items).
- `subSectionItems[].labelId`: a message ID declared in `messages/` (prefix `admin/`), the entry's visible label.
- `subSectionItems[].path`: the URL users navigate to; must correspond to a route in `admin/routes.json`.
- `subSectionItems[].searchKeyWordsHelpers` (optional): message ID with extra keywords that feed the Admin's global search.

## Valid sections and subSections

Extracted from the Admin shell navigation response (the same data that renders the v4 sidebar). Known sections for the `section` field:

`home`, `gettingStarted`, `orders`, `products`, `promotions`, `storeFront`, `shipping`, `marketplace`, `apps` (rendered with the `appStore` icon), `ads`, `storeSettings`, `accountSettings`, `billing`, `userSettings`

For `subSection`, the ID is the suffix of the group's `untranslatedTitleId` after `appframe.navigation.` (e.g. `appframe.navigation.emailTemplates` gives `subSection: "emailTemplates"`). SubSections are specific to each section:

| Section         | subSection ID       | PT title                   | English title    |
| --------------- | ------------------- | -------------------------- | ---------------- |
| orders          | `orders`            | Pedidos                    | Orders           |
| products        | `catalog.title`     | Catálogo                   | Catalog          |
| products        | `prices`            | Preços                     | Prices           |
| promotions      | `promotions`        | Promoções                  | Promotions       |
| storeFront      | `storeFront`        | Storefront                 | Storefront       |
| shipping        | `shipping`          | Envio                      | Shipping         |
| marketplace     | `marketplace`       | Marketplace                | Marketplace      |
| marketplace     | `seller`            | Sellers                    | Sellers          |
| marketplace     | `connections`       | Conexões                   | Connections      |
| apps            | `installed-apps`    | Apps instalados            | Installed Apps   |
| storeSettings   | `payment`           | Pagamentos                 | Payment          |
| storeSettings   | `orders-label`      | Pedidos                    | Orders           |
| storeSettings   | `emailTemplates`    | Templates de email         | Email Templates  |
| storeSettings   | `shipping`          | Envio                      | Shipping         |
| storeSettings   | `product`           | Produtos                   | Product          |
| storeSettings   | `storeFront`        | Storefront                 | Storefront       |
| storeSettings   | `channels`          | Canais                     | Channels         |
| accountSettings | `accountManagement` | Configurações da conta     | Account Settings |
| billing         | `billing`           | Informações de faturamento | Billing          |

Notes:

- Some sidebar groups are registered by apps themselves and carry the app's own message ID instead of an `appframe.navigation.*` one (e.g. `admin/subscriptions.section` for Subscriptions under orders, `admin/intelligentSearch.navigation.label-group` for Intelligent Search, `content-platform.navigation.label-main` for Content, `admin/planner.admin.menu-section` for Planner, `admin/admin-apps.navigation.section.title` for Extensions Hub, `admin-customer-credit.navigation.label-group` for Customer Credit). See [Creating your own subSection group](#creating-your-own-subsection-group).
- `ads`, `home`, and `gettingStarted` had no reusable subSection groups in the captured data.
- `userSettings` renders its items without a titled group.
- The list reflects one account's navigation snapshot; sections/subSections may vary with installed apps and account features.

## Creating your own subSection group

An app can create its own navigation group instead of joining a native one. Attach a custom subSection to an existing top-level `section`: invent your own `subSection` key and provide a `titleId` message ID from your app's `messages/` as the group title. Verified against VTEX's own `admin-example` app:

```json
{
  "section": "orders",
  "titleId": "admin-example.navigation.label-group",
  "subSection": "example",
  "searchKeyWordsHelpers": "admin-example.navigation.search.kws",
  "subSectionItems": [
    {
      "labelId": "admin-example.navigation.label-table",
      "path": "/admin/example",
      "searchKeyWordsHelpers": "admin-example.navigation.table.search.kws"
    }
  ]
}
```

- `section`: must be an existing top-level section. Creating a new top-level section is only documented for the legacy v3 Admin (root-level `titleId` + `path`, no `subSection`).
- `subSection`: a new key of your choice; it does not need to exist in the known subSection list.
- `titleId`: the group title shown in the sidebar, declared in your app's `messages/`. In the Admin shell data these app-created groups appear with the app's message ID as `untranslatedTitleId`, while native groups use `appframe.navigation.*` IDs.
- `searchKeyWordsHelpers` (group-level and per-item): message IDs with extra keywords that feed the Admin's global search.

## Gating a nav entry by License Manager product (`LMProductId`)

Field-verified: adding `LMProductId` makes the Admin shell treat the page as covered by that License Manager **product**. When the current admin's role has no resources granted under that product, the shell renders its own native "ERRO 403 / Você não tem acesso a essa página" page **instead of mounting the app's iframe at all** — the check runs before the linked React app ever loads, driven by a `forbiddenPaths` list the shell fetches with the navigation tree. This is a coarser, earlier gate than anything the app's own backend checks (e.g. a `ctx.clients.licenseManager.canAccessResource(...)` call inside a route handler — see [`vtex-io-rbac`](../../vtex-io-rbac/SKILL.md) for API-level authorization); use both together for defense in depth: `LMProductId` keeps unauthorized admins from ever loading the page, the backend check protects the API regardless of how it's reached.

```json
{
  "section": "orders",
  "subSection": "pickup-codes",
  "LMProductId": "19",
  "titleId": "admin/my-orders-app.navigation.group",
  "subSectionItems": [
    {
      "labelId": "admin/my-orders-app.navigation.label",
      "path": "/admin/pickup-codes"
    }
  ]
}
```

- `LMProductId`: the numeric License Manager **product** ID (not a resource key) as a string, e.g. `"19"` for the OMS product (the same product that owns the `OMSViewer` resource key). Look up the product ID from the License Manager resource table for whatever native permission should gate the page.
- Observed in the wild at both levels: top-level, sibling to `section`/`subSection` (`vtex.reviews-and-ratings`), and per-item inside `subSectionItems[]` (`vtex.wish-list`). Both are real, shipped VTEX apps; there's no documented rule for which to prefer.

## Registering for both Admin versions at once

`admin/navigation.json` can be an array of two navigation objects for the same page: one **without** `adminVersion` (legacy v3 Admin) and one **with** `"adminVersion": 4` (updated Admin). Verified against `vtex.reviews-and-ratings`:

```json
[
  {
    "section": "products",
    "subSection": "catalog",
    "LMProductId": "2",
    "subSectionItems": [
      {
        "labelId": "admin/reviews.title",
        "path": "/admin/reviews-ratings/pending"
      }
    ]
  },
  {
    "adminVersion": 4,
    "section": "products",
    "subSection": "catalog",
    "LMProductId": "2",
    "subSectionItems": [
      {
        "id": "98",
        "labelId": "admin/reviews.title",
        "path": "/admin/reviews-ratings/pending",
        "position": { "index": 8, "operation": "add" }
      }
    ]
  }
]
```

- Both objects target the same page, described once per Admin version; the legacy (non-`adminVersion: 4`) entry doesn't use `id`/`position`.
- `subSectionItems[].id` (adminVersion 4 only): a stable identifier for the item, required when using `position`.
- `subSectionItems[].position`: controls where the item is inserted relative to the group's existing (often native) items.
  - `index`: the position to insert at.
  - `operation`: `"add"` inserts the item at `index` without removing anything else already in the list.

**Difference between including `adminVersion: 4` and omitting it**: each object only registers the page for one Admin experience — they don't merge or fall back into each other.

- Object **with** `"adminVersion": 4`: the entry renders only in the redesigned Admin (help.vtex.com/en/v4). Accounts still on legacy Admin won't see this entry at all.
- Object **without** `adminVersion`: the entry renders only in legacy Admin (v3). It's invisible in the redesigned Admin's sidebar.
- Registering **both** (the array form above) makes the page reachable regardless of which Admin experience the account currently renders — needed while an account base is split across v3 and v4 during VTEX's migration.

For an app built with this skill (targeting `@vtex/admin-ui` exclusively), a single `adminVersion: 4` object is normally enough — only add the legacy variant if the app must also work for accounts still on legacy Admin.

## admin/routes.json

Maps Admin paths to React components in `react/`. Dynamic params use `:param` syntax and arrive in the component's `props.params`.

```json
{
  "admin.app.my-app": {
    "component": "MyAppPage",
    "path": "/admin/app/my-app"
  },
  "admin.app.my-app-details": {
    "component": "MyAppDetails",
    "path": "/admin/app/my-app/:id"
  }
}
```

## react/package.json

`@vtex/admin-ui` is a regular dependency of the react builder. Pin the exact version if the app imports any `experimental_` export (experimental APIs may break on patch/minor releases). Verified working set (react builder `3.x`):

```json
{
  "dependencies": {
    "@vtex/admin-ui": "^0.138.8",
    "apollo-client": "^2.6.8",
    "react": "^16.12.0",
    "react-apollo": "^3.1.3",
    "react-dom": "^16.12.0",
    "react-intl": "^10.1.16"
  },
  "devDependencies": {
    "@types/graphql": "^14.5.0",
    "@types/node": "^13.9.8",
    "@types/react": "^16.9.31",
    "@vtex/tsconfig": "^0.6.0",
    "graphql": "^14.6.0",
    "typescript": "3.9.7"
  }
}
```

Forms need the separate `@vtex/admin-ui-form` package.

Version rules for the react `3.x` builder (verified on a linked production app):

- `typescript` is ALWAYS 3.9.7 and cannot be changed; the builder resets it back to the correct value on its own.
- `graphql` cannot go past `^14.6.0`.
- `react`, `react-dom`, and `react-intl` declared versions CAN be raised when other packages require it, but the builder likely runs its own pinned runtime versions, so features from newer versions are not guaranteed. A mismatched pair (`react` ^16 with `react-dom` ^19) even links without errors, which only works if the builder supplies the actual runtime — still, keep `react` and `react-dom` on MATCHING versions, and their typings (`@types/react`, `@types/react-dom`) matching too when listed. See the community threads on [updating versions for package-compatibility limits](https://community.vtex.com/t/graphql-16-5-0-the-engine-node-is-incompatible-with-this-module/32306/7) and [updating the react version](https://community.vtex.com/t/can-i-update-the-react-version-in-my-project/36484).

Confirmed failures:

- `@types/node` ^20 (with or without a `resolutions` block) breaks the react builder install/compile; ^13 is the verified ceiling here.
- Adding the `vtex.render-runtime` typings URL as a devDependency (`http://vtex.vtexassets.com/_v/public/typings/...`) is unnecessary and contributed to build errors; the builder resolves `vtex.render-runtime` imports on its own.
- Hand-picked newer typescript/tsconfig combos produce errors like `Only ECMAScript imports may use 'import type'`.

## react/tsconfig.json

Use the standard template config verbatim. The builder compiles with its own settings, so a hand-rolled tsconfig (custom `jsx`, `allowSyntheticDefaultImports`, `resolveJsonModule`, custom `include` lists) desyncs local typechecking from the builder and produces the `Only ECMAScript imports may use 'import type'` class of errors.

```json
{
  "extends": "@vtex/tsconfig",
  "compilerOptions": {
    "lib": ["es2017", "dom"],
    "module": "esnext",
    "strict": true,
    "moduleResolution": "node",
    "target": "es2017"
  },
  "include": ["./**/*"],
  "exclude": ["node_modules"]
}
```

## react/.eslintrc.js

```js
module.exports = {
  extends: "vtex-react/io",
  rules: {
    "no-console": "warn",
    "no-restricted-imports": "warn",
    "@typescript-eslint/camelcase": "off",
    "prettier/prettier": ["error", { endOfLine: "auto" }],
    "react/jsx-filename-extension": [1, { extensions: [".js", ".jsx", ".tsx"] }],
    "react/jsx-no-bind": "warn",
  },
  parserOptions: {
    ecmaVersion: 2020,
    tsconfigRootDir: __dirname,
    project: "tsconfig.json",
  },
  settings: {
    react: {
      version: "detect",
    },
  },
  env: {
    browser: true,
  },
};
```

## Page shell

Every route component wraps its content in `ThemeProvider`. `experimental_I18nProvider` (with the runtime locale) localizes Admin UI built-in strings; `ToastProvider` enables `useToast`.

```tsx
import {
  ThemeProvider,
  ToastProvider,
  experimental_I18nProvider as I18nProvider,
  Page,
  PageHeader,
  PageHeaderTop,
  PageHeaderTitle,
  PageContent,
} from "@vtex/admin-ui";
import React from "react";
import { FormattedMessage } from "react-intl";
import { useRuntime } from "vtex.render-runtime";

function MyAppPage() {
  const {
    culture: { locale },
  } = useRuntime();

  return (
    <ThemeProvider>
      <I18nProvider locale={locale}>
        <ToastProvider>
          <Page>
            <PageHeader>
              <PageHeaderTop>
                <PageHeaderTitle>
                  <FormattedMessage id="admin/my-app.navigation.label" />
                </PageHeaderTitle>
              </PageHeaderTop>
            </PageHeader>
            <PageContent>{/* screen content */}</PageContent>
          </Page>
        </ToastProvider>
      </I18nProvider>
    </ThemeProvider>
  );
}

export default MyAppPage;
```

## messages/

Navigation `labelId` values and all UI copy live in `messages/en.json`, `messages/pt.json`, etc., keyed with the `admin/` prefix:

```json
{
  "admin/my-app.navigation.label": "My App"
}
```

## Root toolchain files

Version caps differ per builder: react `3.x` locks typescript at 3.9.7 and `@types/node` around ^13, while node `7.x` caps typescript at 5.5.3 and `@types/node` at ^20 (other builder versions may cap differently). The ROOT `package.json` toolchain (eslint/prettier/husky/lint-staged) CAN be upgraded, but incompatible version combos surface as `node-releases: The engine "node" is incompatible with this module` during the builder's yarn install — when that happens, walk versions back until it installs (see [this community thread](https://community.vtex.com/t/graphql-16-5-0-the-engine-node-is-incompatible-with-this-module/32306/7)). The set below is verified working.

First-resort fix for builder errors: delete `yarn.lock` and `node_modules` (root and per-builder folder) and reinstall/relink. Many install and compile errors come from a stale lockfile, so test this BEFORE any more complex debugging.

### package.json (root)

```json
{
  "private": true,
  "name": "my-app",
  "scripts": {
    "lint": "eslint --ext js,jsx,ts,tsx .",
    "format": "prettier --write \"**/*.{ts,tsx,js,jsx,json}\"",
    "start": "concurrently \"npm:vlink\"",
    "vlink": "vtex link --verbose --clean",
    "prepare": "husky"
  },
  "lint-staged": {
    "*.{ts,js,tsx,jsx}": ["eslint --fix", "prettier --write"],
    "*.{json,graphql,gql}": ["prettier --write"]
  },
  "devDependencies": {
    "@vtex/danger": "^0.2.9",
    "@vtex/prettier-config": "1.0.0",
    "concurrently": "^10.0.3",
    "eslint": "^8.57.0",
    "eslint-config-prettier": "^10.1.8",
    "eslint-config-vtex": "15.1.0",
    "eslint-config-vtex-react": "^9.1.0",
    "eslint-plugin-prettier": "^5.5.0",
    "husky": "9.1.7",
    "lint-staged": "17.0.8",
    "prettier": "^3.9.5",
    "typescript": "^4.9.5"
  },
  "dependencies": {},
  "license": "UNLICENSED"
}
```

Do not add `@types/node` to the root devDependencies.

Lint toolchain rules (verified by actually RUNNING lint, not just `vtex link` — wrong versions install and link fine but crash on the first lint):

- `eslint` = ^8, exactly. v9/v10 dropped `.eslintrc`/`.eslintignore` support (flat config only) and the whole VTEX template ecosystem (`.eslintrc`, `react/.eslintrc.js`, `node/.eslintrc.js`, `eslint-config-vtex`) is legacy format; `eslint-config-vtex@15` peers `eslint: ^8`. Reverting to the template's original 7.12.1 also breaks: `eslint-plugin-prettier` ≥4.1 calls `context.getPhysicalFilename`, which needs eslint ≥7.28.
- `prettier` ^3 WITH `eslint-plugin-prettier` ^5 declared explicitly in the root devDependencies (as above). `eslint-config-vtex` pins plugin ^4, which crashes on prettier 3 (`prettier.resolveConfig.sync is not a function`), but no `resolutions` entry is needed: eslint resolves plugins flat from the CWD, so the root ^5 copy wins over the ^4 nested under `eslint-config-vtex`. Prettier ^2 with the default plugin ^4 also works, but then the VS Code Prettier extension (whose bundled prettier is 3.x) formats differently from what `prettier/prettier` expects, and format-on-save fights lint. Align on 3 when the team formats in VS Code.
- Root `typescript` ≤ ^4: the root copy only serves linting (each builder folder has its own), and `eslint-config-vtex`'s `@typescript-eslint` crashes on newer TS (`Cannot read properties of undefined (reading 'Any')`). Peer range is `^3 || ^4`.
- Folder configs must re-apply prettier LAST: `react/.eslintrc.js` should extend `['vtex-react/io', 'prettier']`, otherwise the folder config re-enables stylistic rules (e.g. `@typescript-eslint/no-extra-semi`) that the root's `prettier` extend had disabled. Check with `npx eslint-config-prettier <file>`.

Husky 9 note: the VTEX template ships husky 4 config (a `"husky": { "hooks": ... }` block in package.json plus shims in `.git/hooks` calling `husky-run`). On husky 9 that block is IGNORED and the stale shims fail commits with `Command "husky-run" not found`. Migration: remove the `"husky"` block, add `"prepare": "husky"` to scripts, create `.husky/pre-commit` containing `yarn lint-staged`, run `yarn prepare` once, and delete the old husky shim files from `.git/hooks`.

### .eslintrc (root)

```json
{
  "extends": ["vtex", "prettier"],
  "root": true,
  "rules": {
    "@typescript-eslint/camelcase": "off"
  },
  "env": {
    "node": true
  }
}
```

### .eslintignore

```
node_modules/
coverage/
*.snap.ts
```

### .prettierrc

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

### .gitignore

The VTEX template gitignore; the entries that matter beyond the usual node noise:

```
node_modules
coverage
logs
*.log
render_build
accounts
.vscode
lib
entrypoints.json
jsconfig.json
.DS_Store
**/yarn-error.log
```

### .vtexignore

Keeps tooling and test files out of the link/publish upload:

```
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

### .editorconfig

```
root = true

[*]
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 2
charset = utf-8
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false
```

## node builder files

The node `7.x` builder tolerates a modern toolchain, with caps: `typescript` up to 5.5.3 and `@types/node` up to ^20.0.0 (other node builder versions may cap at different values).

### node/package.json

```json
{
  "name": "my-app-node",
  "version": "0.1.0",
  "dependencies": {
    "@vtex/clients": "^2.21.0",
    "co-body": "^6.0.0",
    "ramda": "^0.32.0"
  },
  "devDependencies": {
    "@types/co-body": "^6.1.3",
    "@types/node": "^20.0.0",
    "@types/ramda": "types/npm-ramda#dist",
    "@vtex/api": "^7.4.0",
    "@vtex/test-tools": "^3.4.3",
    "@vtex/tsconfig": "^0.6.0",
    "typescript": "5.5.3"
  },
  "resolutions": {
    "@types/testing-library__dom": "6.12.1",
    "@types/express": "4.16.0",
    "@types/express-serve-static-core": "4.16.0",
    "@types/graphql": "14.0.7",
    "@types/node": "20.0.0"
  },
  "scripts": {
    "lint": "tsc --noEmit --pretty"
  }
}
```

### node/tsconfig.json

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

### node/.eslintrc.js

```js
module.exports = {
  rules: {
    "no-console": "warn",
    "no-restricted-imports": "warn",
    "@typescript-eslint/camelcase": "off",
    "prettier/prettier": ["error", { endOfLine: "auto" }],
  },
  parserOptions: {
    ecmaVersion: 2020,
    tsconfigRootDir: __dirname,
    project: "tsconfig.json",
  },
};
```

## Checklist

- [ ] `admin` and `react` builders declared in `manifest.json`
- [ ] `admin/navigation.json` declares `"adminVersion": 4`
- [ ] Every `routes.json` component exists in `react/`
- [ ] Page root wrapped in `ThemeProvider` (+ `ToastProvider`, `I18nProvider` as needed)
- [ ] Labels declared in `messages/` with the `admin/` prefix
- [ ] Root `package.json` on a verified toolchain set (no root `@types/node`; on engine errors, walk versions back; lint stack: eslint ^8 exactly, prettier ^3 + explicit eslint-plugin-prettier ^5, root typescript ^4)
- [ ] `react/` respects the builder caps: typescript 3.9.7 (builder-enforced), `graphql` ≤ ^14.6.0, `@types/node` ^13, template tsconfig verbatim
- [ ] `node/` respects the builder caps (7.x): typescript ≤ 5.5.3, `@types/node` ≤ ^20.0.0
- [ ] Base dotfiles present: `.eslintrc`, `.eslintignore`, `.prettierrc`, `.gitignore`, `.vtexignore`, `.editorconfig`
