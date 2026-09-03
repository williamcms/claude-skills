---
name: vtex-io-performance-debugging
description: "Apply when investigating slow storefront performance (TBT, LCP, FCP, CLS) on a VTEX IO Store Framework theme. Covers the official VTEX debugging checklist (SSR test, __disablePixels, bundle reports, SpeedCurve, admin performance settings, critical CSS, Lighthouse Scoring Calculator) plus a forensic method for attributing a slow script to theme code, an installed account-level app, or a script injected at runtime by a tag manager container. Use before proposing any code fix for a performance ticket, and before estimating one."
---

# VTEX IO performance debugging

## When this skill applies

Use this skill when investigating why a VTEX IO Store Framework storefront is slow (high TBT, LCP, FCP, or CLS), before proposing a fix or writing an estimate for a performance ticket.

- A Lighthouse/PageSpeed report lists third-party scripts, unused JavaScript, or long tasks
- A client asks to "defer third-party scripts" or "reduce unused JS"
- Deciding whether a slow script is something this repo's code can actually fix

Do not use this skill for:
- Backend/Node service latency, caching, VBase, resolver chains, use vtex-io-application-performance
- Block registration/composition mechanics, use vtex-io-render-runtime-and-blocks
- Writing the estimate itself once the diagnosis is done, use effort-estimation

## Official VTEX checklist first

Full doc: https://developers.vtex.com/docs/guides/vtex-io-documentation-debugging-performance-issues

Always run Lighthouse against `https://{account}.myvtex.com/?workspace={workspace}` for the workspace you're testing, or the real end-user URL for production. The plain `https://{workspace}--{account}.myvtex.com/` pattern will not reflect that workspace's real performance. Analyze Home, Search, Category, and Product pages separately, they don't share the same profile.

1. **SSR test.** Disable JavaScript (Chrome DevTools) and reload. Expected: simpler design, no animations or interactive menus, no below-the-fold content. Undesired: blank page, full page refresh on navigation instead of partial update, broken layout. This confirms the SSR HTML itself is functional before any JS-driven improvement is layered on top.
2. **Isolate pixel apps.** Re-run Lighthouse with `https://www.{account}.com.br/?workspace={workspace}&__disablePixels&v={randomvalue}` (the `v` param is a cache-buster, use a random value each time). This tells you how much of the problem is third-party Pixel apps versus the theme's own code, do this before any static-analysis guesswork.
3. **Bundle size.** For a custom React component, check:
   - Dev: `https://{workspace}--{account}.myvtex.com/_v/private/assets/v1/linked/{app}@{version}/public/react/devReport.html`
   - Production: `https://{workspace}--{account}.myvtex.com/_v/public/assets/v1/published/{app}@{version}/public/react/prodReport.html`
   The report breaks the bundle down by module, so a heavy dependency (the doc's own example: `react-icons` dominating a `vendors~Pickup.js` bundle) is visible directly. Fixes: code-splitting, fetching the bundle asynchronously, or swapping the dependency for something more minimal.
4. **Heavy script identification.** SpeedCurve or similar, to name specific offending scripts: remove, replace with something lighter, or change how it loads.
5. **Admin performance settings.** Store Settings > Storefront > Store > Advanced. Enable one at a time and measure the impact with Lighthouse before enabling the next. See "best practices for optimizing performance" and "safely enabling performance settings" in Reference below, the second one specifically covers rollout risk.
6. **Critical CSS.** Requires SSR and CSS concatenation enabled. Verify it actually took effect by checking for `<style id="critical" type="text/css">` inside `<head>`, not just that the setting is on. It rolls out gradually per page/viewport combination (a PDP with a SKU selector and one without won't share the same critical CSS, mobile and desktop won't either), so absence right after enabling doesn't mean it failed.
7. **Lighthouse Scoring Calculator** (https://googlechrome.github.io/lighthouse/scorecalc/#version=6). Plug in your scores to see which metric has the most headroom, then target it:
   - High TBT: use the browser's performance tab and React DevTools to find components running JS for long stretches and blocking the main thread; break long tasks into smaller ones, under 50ms each.
   - High FCP: prioritize above-the-fold assets, don't ship what the first paint doesn't need.
   - High LCP: concentrate the page around one dominant element (a banner) and load it with priority (fetchpriority, preload).

## PageSpeed Insights: lab data vs field data

Source: https://developers.google.com/speed/docs/insights/v5/about

- **Lab data** (Lighthouse) is a single simulated run on a fixed device and network profile (a mid-tier mobile device on a mobile network for mobile, an emulated desktop with a wired connection for desktop). Good for debugging in a controlled environment, not representative of every real visitor.
- **Field data** is powered by the Chrome UX Report (CrUX): real users' First Contentful Paint (FCP), Interaction to Next Paint (INP), Largest Contentful Paint (LCP), and Cumulative Layout Shift (CLS), aggregated over the trailing 28-day collection period. Time to First Byte (TTFB) is also reported as an experimental metric.
- **Field data can silently fall back to origin-level.** A page needs enough real-user samples in that 28-day window to get its own CrUX entry. A recently published page, or one with low traffic, won't have enough, PSI then falls back to origin-level data (aggregated across every page of the site) instead of that page's own numbers, without making the switch obvious in the UI. If the origin itself lacks enough data too, no field data shows at all. Before comparing a specific page's field data before and after a fix, confirm you're actually looking at page-level data, not the whole origin's average.
- **Good / Needs Improvement / Poor thresholds** (lab scores and field data use the same bands):

| Metric | Good | Needs Improvement | Poor |
|---|---|---|---|
| FCP | 0-1.8s | 1.8-3s | >3s |
| LCP | 0-2.5s | 2.5-4s | >4s |
| CLS | 0-0.1 | 0.1-0.25 | >0.25 |
| INP | 0-200ms | 200-500ms | >500ms |
| TTFB (experimental) | 0-800ms | 800-1800ms | >1800ms |

- **Core Web Vitals are specifically LCP, INP, and CLS.** A page "passes" when the 75th percentile of all three is Good. TBT is a lab-only proxy metric (Lighthouse uses it to approximate responsiveness in a single run), it is not itself one of the three official Core Web Vitals, don't conflate the two in a DoD or a ticket.

## What the official checklist doesn't cover

Everything below came from actually running this checklist on a real ticket and hitting gaps it doesn't mention.

**WebFetch strips `<script>` tags.** It converts pages to markdown for readability, so it's useless for auditing what scripts a page actually loads. Use `curl` with a realistic `User-Agent` header to get the raw HTML, then grep for `<script`.

**Read the Lighthouse JSON report directly, not just the rendered HTML.** Save/export the raw JSON (or the one a user sends you) and parse specific audits, they carry exact numbers the rendered report only shows as rounded bars:
- `total-blocking-time`, `largest-contentful-paint`, `cumulative-layout-shift`, the headline metrics.
- `mainthread-work-breakdown`, time per category (Script Evaluation, Parsing & Compilation, Style & Layout, Garbage Collection...).
- `unused-javascript`, bytes downloaded but never executed, per script URL.
- `bootup-time`, execution time per script URL. This is a different measurement than `unused-javascript`, a script can be 100% used and still be the most expensive one to evaluate.
- `long-tasks`, individual tasks over 50ms with their attributed URL.
- `unsized-images` and `layout-shift-elements`, for CLS root-causing.
- `network-requests`, full request list with transfer size, useful for counting duplicate or redundant loads.

**A cited "TBT" number needs to be verified, not assumed.** Check whether it matches `audits['total-blocking-time']` or whether it's actually a manual sum of the `mainthread-work-breakdown` categories, a materially larger, different number that gets mislabeled as TBT in client reports. Also confirm whether it's lab data or field data (see "PageSpeed Insights: lab data vs field data" above), the two are not directly comparable.

**Local Lighthouse runs may carry extension noise.** Check `bootup-time`/`unused-javascript` for `chrome-extension://` entries. If present, treat absolute numbers as unreliable and ask for (or produce) a clean incognito/no-extension run before comparing before/after.

## Attributing a script: theme, account app, or tag manager container

Ticket briefings that say "defer third-party scripts" often assume every listed script is theme code. Most of the time it is not. Confirm ownership before scoping any fix:

1. Check `manifest.json`. If the script's app (`vtex.google-tag-manager`, `vtex.facebook-fbe`, `vtex.hotjar`, `vtexbr.tiktok-tbp`, etc.) is not a declared `dependency` or `peerDependency`, it is installed at the account level, not shipped by this theme. Its loading order and async/defer behavior are not editable in this repo.
2. Check the server-rendered HTML for `<!--pixel:start:APP_NAME--> ... <!--pixel:end:APP_NAME-->` markers. `vtex.pixel-manager` wraps every registered pixel app's rendered output in these comments. Whatever HTML sits between them is that app's own output, not the theme's.
3. Search the repo for components that inject their own script tag directly (`createElement('script')` / `createElement("script")`), separate from any pixel app. This catches theme-owned third-party integrations that pixel-manager markers won't reveal, they're real, in-repo code even though they call out to an external domain.
4. Whatever appears in the HAR or Lighthouse network capture but not in the static server-rendered HTML (curl the page, compare) is being injected client-side, almost always fired by a tag manager container after its own container script runs. This is a strong signal, not absolute proof, when possible confirm via the request's initiator chain in the HAR rather than absence alone. Editing its load timing means editing the container configuration, not this repo.
5. Only after ruling out 1 through 4 does a script belong to the theme's own code.

## bootup-time attributes a whole bundle, not one component

VTEX's react builder bundles multiple unrelated components together into one asset chunk, by page composition, not by feature. The `bootup-time` audit (see "What the official checklist doesn't cover" above) reports cost per bundle URL, not per component inside it.

- Before blaming one component for a bundle's cost, check every component the bundle's `files=` querystring lists, and grep each one's own imports for a heavy dependency (a carousel library, a rich date picker, a charting library). That's usually the real driver, not the small component you first suspected.
- Don't recommend lazy-loading a component just because it isn't gated behind a click. Confirm it's actually heavy first (line count, imports, whether it shares a dependency like `axios` that's already loaded unconditionally elsewhere on the same page, splitting it out wouldn't remove that shared weight from the page at all).

## Validating "this block or component is unused" across a VTEX theme

Before proposing to delete a component as dead code, confirm it end to end, this is easy to get wrong:

- Search the whole repository, not just one `store/blocks/*.jsonc` file or folder. Block composition can live several folders deep.
- VTEX allows `"blockId#instanceId"` for multiple instances of the same block. A search for the exact `"blockId"` string alone will miss every instanced usage and produce false "unused" results.
- Always run a control test before trusting a negative result: search for something you already know is used, with the exact same method and tool, and confirm it's found. This applies beyond block IDs too, a shell recursive glob (`**/*.jsonc` without `globstar` enabled) can silently fail to recurse and return an empty, misleading result with no error.

## Related skills

- vtex-io-application-performance, backend and Node performance, VBase, caching
- vtex-io-render-runtime-and-blocks, block registration and composition mechanics
- effort-estimation, once the diagnosis above is done, use this to size and write the ticket

## Reference

- https://developers.vtex.com/docs/guides/vtex-io-documentation-debugging-performance-issues, main checklist
- https://developers.vtex.com/docs/guides/vtex-io-documentation-best-practices-for-optimizing-performance, performance settings and critical CSS detail
- https://developers.vtex.com/docs/guides/vtex-io-documentation-safely-enabling-performance-settings, rollout risk for admin toggles
- https://developers.vtex.com/docs/guides/vtex-io-documentation-getting-started-with-lighthouse
- https://developers.vtex.com/docs/guides/vtex-io-documentation-tracking-changes-in-lighthouse-performance-score
- https://googlechrome.github.io/lighthouse/scorecalc/#version=6, Lighthouse Scoring Calculator
- https://developers.vtex.com/docs/guides/vtex-io-documentation-pixel-app, what a Pixel app is
- https://developers.google.com/speed/docs/insights/v5/about, PageSpeed Insights lab vs field data, thresholds, Core Web Vitals definition
