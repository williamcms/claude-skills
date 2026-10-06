# claude-skills

Personal Claude Code skills. Some are original, others are my edited versions
of the public ones with production learnings folded in.

## Skills

| Skill                                                  | Description                                                                        |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| [effort-estimation](effort-estimation)                 | Writes calibrated time/effort estimates for dev tasks and tickets                  |
| [vtex-io-admin-ui](vtex-io-admin-ui)                   | VTEX IO Admin UI (@vtex/admin-ui) design system: components, tokens, theming       |
| [vtex-io-app-contract](vtex-io-app-contract)           | VTEX IO app manifest/contract: builders, dependencies, billing, identity           |
| [vtex-io-graphql-api](vtex-io-graphql-api)             | VTEX IO GraphQL builder: schema, resolvers, `@auth`/`@cacheControl`, 2.x migration |
| [vtex-io-masterdata](vtex-io-masterdata)               | VTEX IO Master Data v2: entities, schemas, typings, CRUD patterns                  |
| [masterdata-storage-strategy](masterdata-storage-strategy) | When Master Data is the right storage, schema design, indexing, billing        |
| [vtex-io-messages-and-i18n](vtex-io-messages-and-i18n) | VTEX IO localization via the `messages` builder and `react-intl`                   |
| [vtex-io-react-apps](vtex-io-react-apps)               | VTEX IO React storefront/admin components, blocks, css-handles                     |
| [vtex-checkout-native-fraud-controls](vtex-checkout-native-fraud-controls) | VTEX native checkout anti-fraud/anti-bot controls: reCAPTCHA v2/v3, blockRecaptchaV2, orderForm locks |
| [vtex-io-performance-debugging](vtex-io-performance-debugging) | VTEX IO storefront performance debugging: official checklist, script ownership, bundle attribution |
| [vtex-io-site-editor-content-schema](vtex-io-site-editor-content-schema) | VTEX Site Editor content schemas: JSON Schema, widgets, conditional fields, admin-pages GraphQL |
| [commit-guidelines](commit-guidelines)                 | Git commit message rules: format, approval workflow, co-author attribution         |

## Install

Clone the repo, then link the skill(s) you want into your Claude Code skills folder.

```bash
git clone https://github.com/williamcms/claude-skills.git
```

**Windows** (junction, no admin needed):

```
mklink /J "%USERPROFILE%\.claude\skills\<skill-name>" "C:\path\to\claude-skills\<skill-name>"
```

**macOS/Linux** (symlink):

```bash
ln -s /path/to/claude-skills/<skill-name> ~/.claude/skills/<skill-name>
```

Linking (instead of copying) keeps the skill live: pulling repo updates
applies immediately, no re-copy needed.
