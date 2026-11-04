---
name: faststore-webops-cms-preview
description: "Apply when previewing an unpublished storefront change on a VTEX FastStore store before it is published — the code from a WebOps deploy build, the content from an unpublished branch/version in the new VTEX CMS (content-platform), or both combined. Covers triggering a WebOps rebuild, getting the deploy preview origin (buildHash--account.preview.vtex.app), reading documentId and versionId from the content-platform branch-form URL, and composing the /api/preview link with contentType, documentId, versionId and locale. Use when a PR's visual change must be validated against specific unpublished CMS content."
---

# WebOps Deploy + CMS Branch Preview (FastStore)

## When this skill applies

Use this skill to preview an unpublished storefront change on a VTEX FastStore store before anything reaches production — the *code* from a WebOps deploy that is not yet promoted, the *content* from an unpublished branch/version in the new VTEX CMS (`content-platform`), or both at once.

- Validating a storefront PR's visual change against a specific CMS content version
- Opening a WebOps deploy's preview build and pointing it at a chosen CMS document/version
- Reproducing what a page will look like once a CMS branch is published

Do not use this skill for:

- Legacy Store Framework (VTEX IO) theme preview — this is FastStore plus the new CMS. For the IO theme model use `vtex-io-storefront-theme-app`.
- General FastStore development conventions — use `faststore-storefront`.

## Key concepts

- **WebOps deploy preview** — every WebOps build (a rebuild of a branch or PR) publishes an isolated preview of that code at `https://{buildHash}--{account}.preview.vtex.app/`. This is the *code* under test, serving independently of production.
- **New VTEX CMS (`content-platform`)** — content is versioned as branches. An unpublished branch holds a draft version of a document that is not live yet. This is the *content* under test.
- **`/api/preview`** — the FastStore route that renders a given CMS document and version on top of whatever build serves it. Combining a deploy preview origin with this route lets you test unpublished code and unpublished content together.

## The preview URL

```text
{previewUrl}/api/preview?contentType={contentType}&documentId={documentId}&versionId={versionId}&locale={locale}
```

- `{previewUrl}` — the WebOps build preview origin, e.g. `https://sfj-034c3a6--loungerieqa.preview.vtex.app`
- `{contentType}` — the page/content type, e.g. `home`
- `{documentId}` — the CMS document (the entry)
- `{versionId}` — the CMS branch/version of that document
- `{locale}` — e.g. `pt-BR`

Example:

```text
https://sfj-034c3a6--loungerieqa.preview.vtex.app/api/preview?contentType=home&documentId=4af215ae-8f40-4cc2-9c5d-972a023ece89&versionId=bf9a3f21-7d4a-4838-8c22-a61c8215ce2b&locale=pt-BR
```

## Step by step

1. **Trigger the WebOps rebuild** for the branch/PR under test, in the admin:
   `/admin/webops/deployments/{projectId}/{deploymentId}`
2. **Grab the build's preview origin** from the finished deploy: `https://{buildHash}--{account}.preview.vtex.app/`.
3. **Get the CMS `documentId` and `versionId`** for the content you want to render (see below).
4. **Compose** the `/api/preview` URL from the template above and open it against the preview origin.

## Getting documentId and versionId

Two ways:

- **Eye icon (transient).** Clicking the eye/preview icon on the CMS entry produces the link, but it is short-lived — copy it immediately or it expires.
- **Admin branch-form URL (stable, preferred).** Open the entry's branch form in the new CMS:
  ```text
  /admin/content-platform/branches/{versionId}/form/{account}/{contentType}/entries/{documentId}
  ```
  Read the IDs straight from the path:
  - the segment after `branches/` is the **`versionId`**
  - the segment after `entries/` is the **`documentId`**

  Then plug them into the preview URL template.

## Common failure modes

- **Swapping `versionId` and `documentId`.** `branches/...` is the `versionId`, `entries/...` is the `documentId`. Backwards renders the wrong content or none.
- **Relying on the eye-icon link.** It expires fast; prefer building the URL by hand from the admin branch-form path.
- **Using the production origin instead of the build preview origin.** The point is to serve unpublished code, so the URL must start with `{buildHash}--{account}.preview.vtex.app`, not `{account}.myvtex.com`.
- **Omitting `locale`.** It can fall back to a default locale and hide the change you are validating.

## Related skills

- [`faststore-storefront`](../faststore-storefront/SKILL.md) — FastStore development conventions, section overrides, and BFF.
- [`vtex-io-storefront-theme-app`](../vtex-io-storefront-theme-app/SKILL.md) — the legacy VTEX IO theme equivalent, which has a different preview model.
