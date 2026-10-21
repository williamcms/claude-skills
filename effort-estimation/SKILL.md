---
name: effort-estimation
description: Apply when asked to estimate the time/effort/budget for a dev task, bug fix, or feature, especially for client-facing or handoff-to-another-dev estimates. Covers the mandatory pre-work checks, a fixed output template, scoping against a real design reference, presenting multiple implementation paths without duplicating shared steps, a calibration table of real dev-hour anchors, always asking the user whether to include a platform-specific-unknowns line, and arithmetic discipline. Use whenever the user asks "how long would this take", "can you estimate this", or wants a budget/orçamento written up for a ticket.
---

# Effort estimation

Writing a time estimate for a development task, read by a client, a PM, or another developer who will pick up the work.

This skill is self-contained: everything needed to produce an estimate is here. The only external sources to consult are other installed skills and the web (for a platform's official documentation). The estimate itself is written in the language of the request, usually Brazilian Portuguese. In this skill, everything is in English except the strings that must appear verbatim in the delivered estimate; those are shown in Portuguese, in backticks, and must be reproduced exactly.

The skill has three parts: general rules (how to work, independent of the document's shape), the output template (the single authority on how the document is laid out), and anchor sections (detail for individual template items). Nothing outside the template defines layout.

# General rules

## Before writing a single number

These are mandatory checks, not optional. Do not skip one and estimate blind; ask the user for what is missing.

- **Pull the task from Tarefy when given its link or id.** When the user provides a Tarefy demand link or task id, fetch the task's details and comments through the Tarefy MCP and use them as the base briefing instead of asking the user to paste it. Providing the link or id is itself the authorization to read that task's data, so pull it without asking first, a deliberate, scoped exception to presenting a call before running it; only when the MCP is not connected, proactively offer to add it and help the user connect it, the same way as humanizer (see "Auxiliary tools and MCPs").
- **Ask for a design link if the task touches UI and none was given, unless the task is purely data/registration work (cadastro de itens, dados) with no UI to design.** Prompt directly, in Portuguese: "tem um Figma pra isso?". Pull it before estimating, because mockups routinely contain requirements the text brief omits (extra buttons, states, or fields). If the user confirms there is no design reference, proceed and note that the estimate assumes reuse of existing UI patterns. If a design reference exists and covers a visual detail (icon, color, exact look, placement), treat that detail as settled even if you did not inspect the design in depth, and even if the user called the design "irrelevant" for estimation purposes — that phrase means "don't spend time analyzing it," not "it doesn't define the look"; only list it as an open question later (`Pontos em aberto:`) when no design reference exists at all or it is genuinely ambiguous. When you are not sure whether the design actually covers a given aspect (placement, exact visual, a state) rather than the brief, do not guess either way — guessing it's covered can hide real scope, guessing it's open can pad the estimate with a question the design already answers. Ask the user directly with a concrete yes/no question naming the aspect, in Portuguese: "o Figma define onde o componente será posicionado?", "o Figma define o visual do componente (ícone, cores)?". Only fall back to your own judgment, and list it as a genuine open question, when the user says they don't know either.
- **Cross-check the design against the current live product for features the mockup does not show.** A redesign mockup shows the new screens, not every existing feature and state. Features present today but absent from the design (an order-note field, a login prompt, loading, error, empty or out-of-stock states, quantity limits, installments, secondary buttons) are easy to miss and get dropped, then resurface as unplanned work or a regression. Open the live version next to the design and record what exists today that the design omits. This feeds the template item "Itens que não aparecem no Figma:".
- **Cross-check a detailed example against the rest of a list the same way.** When a briefing gives one concrete example (an existing URL, existing content, a migration need) alongside a list of other items presented as equal priorities, do not assume the example is the exceptional case and the rest need less work. Ask whether the same conditions generalize to every item in the list before sizing them differently.
- **Get the code before estimating.** Do not estimate from the brief alone; you need the repo to verify the diagnosis and to find code you can reuse. When a repo path or clone URL is given, first look for the repo already present under the root where Claude is running (unless another location was specified beforehand); if it is not there, ask permission before cloning it into that same root. Only proceed without the code if the user says it is unavailable.
- **Verify the diagnosis in the actual code or logs before describing it.** Do not restate a bug report's claimed cause as fact. Read the files, search for the route, config, or constant in question, and reproduce if possible. If someone already diagnosed it, confirm it yourself and phrase it as confirmed, in Portuguese: "Confirmado no código: ...".
- **Verify claims about what is live — rendered output, or whether a file/route/template exists and matches something — against a live source, not against the repo or its own docs alone.** Code in the repo often does not show the final reality: CMS-managed content (Site Editor, a headless CMS) renders markup nobody versioned, a README or OVERVIEW.md can be stale and contradict a directory listing you already have, and a block declared in code can have no live route at all. Before asserting what is rendered, what is absent, or that a given template is "the closest match" to something in the briefing, check it against a live fetch or the live route table, not just guesswork or the repo's own structure and docs. Only fall back to asking the user, or to leaving it as an open question, when no live URL or route table exists yet to check.
- **Cite the source inline when an open point rests on a specific, counterintuitive platform-documentation claim.** A claim the reader might doubt (a documented default that contradicts the observed behavior, a parameter's actual scope) needs its source right next to it, not only in the trailing `Links:` list — the reader has to be able to check it without hunting.
- **Look for existing code to reuse before assuming a build-from-scratch cost.** A feature that reuses an existing component, client, or route convention is materially faster than one with no precedent. Search for it before sizing. Do not cite a file as "reuse" just because it contains a trivial one-line call to a native/stdlib API (`window.scrollTo`, `fetch`, `Array.map`) that is also used elsewhere — that is not a precedent that changes effort, and listing it reads as padding, not diligence. Only cite a file when it is a genuine integration point (where the new code plugs in) or non-trivial reusable logic/structure. This includes a pattern already shipped in a sibling repo (another brand's theme on the same platform) when the brief names it as the template to follow. Size with the ponytail skill in full mode: the laziest solution that actually works is what you cost, reuse, stdlib, and native platform features before any new code (see "Auxiliary tools and MCPs").
- **When the same change is priced across more than one store, theme, or repo, verify shared-code claims independently in each one.** Reading one repo and assuming the others match "because it's the same theme" is a real gap, not a shortcut: confirm the specific files the estimate cites, and any codebase-quality-point measurement, in every repo before stating they are identical. See "Multi-store / multi-repo estimates" for how this feeds the budget.
- **When that search turns up an existing config toggle instead of code to write, check its exact granularity before estimating: it may be split per breakpoint (separate mobile and desktop switches for the same outcome) or some other dimension the brief never names.** Cost enabling it and confirming it works as a single step, not the flip alone. A ticket framed as something broken can turn out to be a feature that already works and was simply never turned on, but that's only a real answer once it's been enabled and verified, not something to assume from reading the code alone.
- **Use an installed platform skill for platform-specific detail.** Do not estimate a platform constraint from memory when a skill documents it. If no relevant skill is installed, ask the user for one or for documentation; failing that, search the web for the platform's official documentation before guessing at limits, quotas, or required patterns. A web-fetched summary can misstate or invert a claim (it summarizes through a smaller model); for a counterintuitive or load-bearing claim, ask for a verbatim quote of the source line, or fetch the raw page directly, before treating it as confirmed.
- **Do not default to the most sophisticated architecture.** Question whether a database entity, a new service, or a new app is justified for the data volume and change frequency described. A handful of rarely-changing records driven by a config value is usually cheaper to build and maintain than a backing store, until the requirements say otherwise. The same restraint applies to gating or scoping content: check whether the platform's own admin/CMS already does it natively (for example, VTEX Site Editor content Versions applied per URL) before proposing a custom component. It also applies to a value that looks hardcoded in a theme's block declaration (a fixed ID, a fixed query filter): before costing work to parameterize it in code, check whether the platform's Site Editor (or equivalent CMS) already lets a merchant override that same prop per URL — the same blockId powering several different live pages with different content, as institutional pages commonly do, is the tell that it already does.
- **Do not guess who owns or has access to external infrastructure the fix touches** (a CDN/WAF zone, a DNS panel, a third-party account outside this repo). State it as an open point for the team or client to confirm, never as settled fact. The same restraint applies to a complexity rating: if it cannot actually be backed by verified facts, write `Complexidade: n/a.` instead of asserting a level. See "External-ownership handoff variant".

## Scope to what actually changed, not the briefing's definition of done

When the user gives their own list of what changed, estimate that list. A client briefing's definition of done often describes the entire redesigned page (typography, colors, grid, every section), while the real difference against the current site is a handful of presentation tweaks; costing the full definition of done can inflate the estimate two to three times. Use the design to confirm what changed, not as a checklist of work. If a definition-of-done item seems required but is not in the user's list, ask instead of silently adding it.

The same discipline applies in the other direction: when your own investigation turns up other plausible-looking targets the briefing never named (other page templates, other routes, other content types), do not fold them into scope on your own judgment. Map what you found back to exactly what the briefing or reference doc named, and ask before adding a template, page type, or surface nobody asked for.

## Keep it lean

The estimate is a single, direct, final document read by both the developer who will implement it and the client who pays for it. Be direct and cut anything that serves neither.

- State the conclusion, not the investigation trail. No file lists, no search trails, no stack minutiae. Write "é o site institucional", not "procurei por X e Y e não achei Z".
- Do not assert specifics you could not validate in code. Generalize when unconfirmed: write `as combinações`, not `as 21 combinações`, if you could not confirm the count.
- Never include the local absolute path used to reach the repo (for example `C:\www\project`, `/Users/name/code/project`) anywhere in the delivered estimate. It is specific to whoever investigated the code, not to the reader, and it leaks local machine structure into a client-facing document. Name the codebase by its repo/platform name only (`tema project`, not `C:\www\project`); cite individual files with repo-relative paths only, as in template item 4.

## Calibrate to who will do the work

Always calibrate for a junior or generalist pace unless the user says the work goes to someone more senior; do not ask who will execute, assume junior by default. A senior specialist who knows the platform's edge cases is not the same number as a junior who discovers them live.

Calibration anchors from real estimates, at a junior or generalist pace on VTEX. Adjust proportionally for other platforms or more senior teams.

| Task shape | Stack | Hours |
|------|------|------|
| Presentation-only tweak to an existing component (icon-only button, simplified button style) | Any | ~30min |
| Placing an already-registered block or component on another page | VTEX IO | ~30min |
| Repositioning and restyling a block already plugged into the page | VTEX IO | ~30min |
| Spacing or alignment pass across one page's components | Any | ~1h30 |
| SCSS variants for a small, fixed set of layout options on an existing component (e.g. left/center/right alignment), including a mobile fallback | FastStore | ~45min |
| Restyling an existing mid-size (about 100-line) component to a new design | Any | ~1h |
| Wiring a component to respect new CMS toggle/enum props (conditional render, forward a value to CSS) | FastStore | ~45min |
| Porting an existing calculated badge/tag pattern from one page type to another (e.g. a vitrine/PLP badge added to the PDP), same toggle convention | FastStore | ~45min |
| Adapting an existing component from a sibling theme/repo to the current theme's design | FastStore | ~1h30 |
| Responsive validation and functional regression pass on one existing page | Any | ~1h |
| Responsive and functional test of a toggle plus a small set of discrete variants (e.g. 3 alignments), desktop and mobile | FastStore | ~1h |
| Enabling an already-built admin/CMS toggle that implements the exact ask, plus confirming it works per breakpoint (mobile/desktop) | Shopify | ~30min |
| Adjusting a single value on an already-active native checkout lock via the platform's Checkout API (e.g. VTEX's recaptchaMinScore) | VTEX (Checkout API) | ~20min |
| Functional test of the checkout flow after a native anti-fraud lock change (a legitimate-purchase pass plus one simulated-risk scenario) | VTEX (Checkout API) | ~45min |
| Opening a support ticket with the platform requesting an account-level security hardening change already scoped and agreed (e.g. VTEX's blockRecaptchaV2) | VTEX | ~30min-1h (higher end when the ticket needs to reference a specific finding, not just request the action) |
| Responsive/functional QA on each additional page reusing an already-validated component, per page after the first | Any | ~10min |
| Content change only, no code | Any | 1h-1h30 |
| Editing a hotsite's content in the admin's rich-text editor (Hotsites > hotsite > content), admin path already mapped, including transcription from the client's doc and a responsive check | Wake | ~40min |
| Researching and documenting a security-mechanism comparison/recommendation for an estimate's own deliverable (e.g. reCAPTCHA v2 vs v3) | Any | ~1h |
| Migrating existing content (banner, title, description, SEO text) from an old custom page to a native platform page via built-in CMS versioning, per item | VTEX IO | ~30min |
| Transcribing ready-made content (e.g. Q&A pairs) from a source doc into an already-existing content slot, per list item | Any | ~10min |
| New data entity, few fields, manual data entry (includes a CMS-managed list with a few fields, e.g. tags with text/color/border per collection id) | Any | ~1h |
| Add a couple of new toggle/dropdown fields to an existing block's schema (contentSchemas.json only, no separate publish round trip) | VTEX IO | ~30min |
| Add a couple of new toggle/dropdown fields to an existing section's Headless CMS schema, then generate and publish it | FastStore | ~45min |
| Simple event listener or webhook handler, single purpose | Any | ~2h |
| URL-param detection and apply via an API with no native support | Any | ~3h30 |
| Custom admin interface for non-technical users | Any | ~8h |
| Order-broadcast automation populating a data store | Any | ~11h |
| Full small app: background job, data store, and frontend logic | Any | 16h-22h depending on whether it includes a user-facing listing screen |

Patterns worth carrying forward:

- A custom admin interface is nearly as expensive as the automation behind it. Always cost it as its own line, never assume it is "just a form."
- "No code" stays cheap only when it is pure content. Touching checkout, order data, or requiring a new app raises the complexity floor regardless of scope size — see "Complexity level" for the exact rating.
- User-facing screens (a new listing page, an account section) cost proportionally more than the backend logic feeding them.
- Integrating the same feature into the PDP typically costs more than into a PLP/vitrine list view (more layout and state to fit into); cost each surface as its own line, never assume PDP integration equals list-view integration.

## Codebase quality points

When you have repo access, do a quick code-quality scan before sizing, because legacy patterns slow down every task in that repo. Each confirmed point below adds time to the estimate: use the point's own fixed cost when it states one (for example the plain-CSS point is 30min), otherwise about 10-15% of the coding total, rounded to a clean step. Every applied point must appear as its own named line in the budget so the reader sees why the number grew.

- **Plain JS versus TypeScript ratio.** Count component files (`.js` and `.jsx` versus `.ts` and `.tsx`, excluding config and build files). More than 40% plain JS is one point.
- **Plain CSS instead of SASS/SCSS.** Plain CSS only (no `.scss`) is one point, costed as a flat 30min line, because the lack of mixins and shared color variables makes styling work slower. This applies only when the platform lets you precompile SCSS (for example VTEX). Do not apply it where the theme system is plain CSS by nature (for example Shopify); there it is not a debt.
- **Other friction signals** (no linter or formatter config, very large components, deeply nested overrides, an outdated framework major, an untyped data layer, or a large amount of dead code): flag each one you find and ask the user whether it should count as a point before applying it.
- **In a multi-store estimate, measure every point per repo**, not just the first one checked; fold them into a single shared budget line only after confirming the same pattern holds in each repo.

## Arithmetic and rounding discipline

- **Never price the same work twice.** Before sending, reread every line and confirm that no step's time is also embedded in another line's time. The classic case is a variant line whose description overlaps a shared step. This is a writing-attention problem, not a calibration one: the numbers were right individually, the composition double-counted.
- **Sum the breakdown and check it against the headline total before sending.** A total that does not match the sum of its own lines is the most common mistake, usually after editing one line and forgetting to propagate the change.
- **The lines must sum exactly to the headline total, even after rounding.** If rounding the raw sum up to a clean 15- or 30-minute increment opens a gap, fold that gap into one existing line (normally the platform-unknowns line), rather than presenting a total the breakdown does not support. The reader should be able to add up every line and land exactly on the headline number.
- **Round up, never below the raw sum of the listed parts.** If two variants share a base and differ by a set delta, keep that delta after rounding (if variant B does 1h30 more raw work than variant A, the rounded totals still differ by 1h30).
- When a correction changes one number, re-verify every total that depends on it, not just the line that changed.

## Ask for the time spent producing the estimate

Once the coding budget and headline total are otherwise settled, ask the user how much time was spent producing the estimate itself, in Portuguese: "quanto tempo você gastou pra montar esse orçamento?". This is separate from the estimated coding hours: it is the researching/writing effort behind the estimate, not part of the work being scoped. See "Time spent producing the estimate" for where it goes in the template, when to skip the ask, and how it folds into the total.

## Humanize the final text

Once the estimate is otherwise settled, before the file handoff below, run it through the `humanizer` skill so it reads like the writer instead of a template dump. It only rewords: never let it touch a number, a fact, or one of this skill's literal Portuguese template strings.

If `humanizer` is not installed, see "Auxiliary tools and MCPs" for how to add it.

## Deliver the final estimate as a file

Once the estimate text is genuinely final (confirmed by the user, or ready for first hand-off), save it verbatim as a `.md` file and reply with just the file path, not the full text pasted again in chat. Chat copy-paste can silently corrupt or truncate a long estimate (a garbled line, a cut-off link); a file survives copy/paste intact.

- Default to the session scratchpad without asking. Only use a different location (a docs folder in the project being estimated, or elsewhere) when the user specifies one for that estimate.
- The saved file's content is exactly the template output, nothing added or reformatted for the file.
- While drafting or iterating on the estimate with the user, keep pasting it inline as normal; only switch to the file-and-path handoff once the text is genuinely final.
- Name the file after the estimate's title, kebab-case, not a generic `estimate.md`.

## Clean up what you downloaded

Investigating an estimate often means cloning repos or downloading images that were not already in the workspace. Keep a running note of what you added; anything that existed before you started does not count. The estimate `.md` file saved to the scratchpad (see "Deliver the final estimate as a file") counts as one of these items too. After delivering the estimate, ask the user whether to delete what you added. Do not delete anything unprompted, and do not ask a blanket "clean up?":

- Group by category (cloned repos, downloaded images, the scratchpad estimate file), not one long list, unless there are only one or two items total.
- Within a category, let the user pick all, none, or specific items.
- Only offer categories that actually apply to this session.
- If a cloned repo is likely useful for the follow-up work (it is the project being estimated and more work is coming), say so and suggest keeping it before defaulting to delete.

# The output template

This is the only place that defines the estimate's layout. The anchor sections below add detail for individual items but never change the structure. Use the same order and the same literal Portuguese headings every time.

Format: plain text, no markdown headings, no bold. Lists as "- " bullets. Times as `Xh`, `Xh30`, or `XXmin` (for values under an hour, e.g. `30min`). Commas, not em dashes. The Portuguese strings in backticks are literal and must be reproduced verbatim.

Omit any section whose content you cannot state; do not fill it with speculation. Put the uncertainty in `Pontos em aberto:` instead.

1. `Estimativa: <título curto>`, on one line.
2. Diagnosis line or short paragraph, always present. State concisely what is happening and what will be done to address it. Keep it readable for a non-technical reader, though technical terms are allowed where they help. See "Diagnosis".
3. `Plataforma: <plataforma>, <ferramentas externas>.` Lists the platform and the external tools, when there are any (for example `Plataforma: VTEX, Sizebay.`). Never a description of the work, scope lives in the file list, the open questions, and the budget bullets.
4. `Arquivos já existentes que serão a base do trabalho:` followed by a bullet list of `caminho/do/arquivo: descrição`. Include only when the repo was validated; a short line may follow for extra existing behaviors. Omit the whole section when there is no repo access. What qualifies as a citation here follows the "look for existing code to reuse" check above, not a separate rule.
5. `Tempo de orçamento: Xh.`, on its own line. States the time spent producing the estimate itself. See "Time spent producing the estimate" for when to ask, when to omit it, and how it folds into the total.
6. `Pontos em aberto:` followed by a bullet list, one open question per line. See "Open questions".
7. `Itens que não aparecem no Figma:` followed by one posture sentence and a bullet list. Include only when applicable. See "Items not in the Figma".
8. `Tempo total estimado: Xh.` then, on its own separate line below it, `Complexidade: <nível>.` (never on the same line as the total). Use one of the standard levels defined in "Complexity level". When there are variants, replace the single total with one total line per variant (for example `Variante A, <descrição>: Xh` and `Variante B, <descrição>: Xh`), followed by a single `Complexidade: <nível>.`; see "The budget" for how the breakdown is shared. For an investigation-first task, replace `Xh` with the literal `A ser estimado após investigação` (see "Investigation-first tasks"). For a multi-store/multi-repo estimate, add a `Tempo por loja: Xh cada.` line (or one line per store, if they differ) directly after `Complexidade:`; see "Multi-store / multi-repo estimates".
9. `Sobre o orçamento:` followed by the bullet breakdown, each line `- <descrição>: Xh`. No separate sum line: item 8 already states the total, and the bullets plus the item 5 `Tempo de orçamento` line must sum to it exactly. See "The budget". Omit this section entirely for an investigation-first task.
10. `Bloco opcional, a confirmar (<o que>)` followed by a bullet list, then the closing line `Se incluídos, somam Xh, levando o total a Xh.`. Include only when there is optional or out-of-briefing scope.
11. `Links:` followed by a bullet list of reference URLs worth keeping for later: the live site, a dev/staging/workspace environment, the repo/GitHub URL, and official platform documentation found while researching the task. May be used in any estimate. Never search the web just to fill this section. Do not repeat links or images already in the briefing (task links, attached screenshots): the reader already has the briefing, restating them here is noise, not a citation. If there is nothing worth keeping, omit the whole section.

# Anchor sections

## Diagnosis

Always include a short context or diagnosis (template item 2). State concisely what is happening and what will be done to address it, so both the client and the developer understand the work; keep it readable for a non-technical reader, though technical terms are allowed where they genuinely help. Ground it in what was verified in the code (see "Verify the diagnosis in the actual code or logs"). State only what is not obvious (the mechanism and the reason); do not restate what the briefing already says. For a straightforward redesign or restyle, keep it to one or two sentences naming what changes and why.

## Time spent producing the estimate

Template item 5, `Tempo de orçamento: Xh.`, is the time spent researching and writing the estimate itself, not part of the scoped work. Ask the user for it once the coding budget and headline total are otherwise settled: "quanto tempo você gastou pra montar esse orçamento?". It folds into the headline total (item 8): that total equals the coding budget (sum of item 9's bullets) plus this line.

Skip the ask, and omit the line, whenever the estimate carries no numeric hours total and no real work has happened yet to produce one: investigation-first tasks, the external-ownership handoff variant, and the external-vendor visibility variant (see all three below). There is nothing to aggregate it into. The client-decision variant is the exception: keep the line standalone, uncombined with any total, because it reflects investigation already performed even though the forward scope is still undecided. When there are multiple variants (item 8 lists one total per variant), ask once and add the same value to every variant's total, since it is the same document regardless of which variant gets picked.

## Open questions

State each ambiguity (template item 6) as its own line, never buried in prose. Name the ambiguity and the assumption the estimate is making. Keep it terse, one clear sentence per open question, not a hedging paragraph. Phrase an unconfirmed hypothesis as the concrete next step that would verify it, not as a disclaimer about its own certainty (write "investigar se X", not "isso não é confirmado, deve ser tratado como hipótese"). When the point assigns a follow-up action and "who will execute" is already known, name that party directly instead of a vague clause like "quem tiver acesso". List only what is genuinely undecided and changes scope, not something the budget already resolves. Never manufacture a boilerplate open question out of a missing input (that there is no Figma, no repo, or that someone must grant access): a genuinely missing input is raised with the user up front, not parked here, unless the user explicitly asks for it to be listed as an open question. Tone, in Portuguese: "Não foi informado X. A estimativa abaixo assume Y; Z mudaria o escopo."

Whether a design detail is settled or genuinely open is decided during the "ask for a design link" check above, not here — do not redecide or restate it; only reflect its outcome as a line if that check left something genuinely undecided. Likewise, do not list an implementation default (a scroll threshold, a debounce value, a z-index choice) as an open question just because the brief didn't state it; if a reasonable default resolves it without changing scope, it belongs in the assumption baked into the budget, not in this section.

If there is truly no genuine open question, write the single line `N/a` under this heading rather than manufacturing one to fill space.

When a single open question blocks the entire scope, not just one line item, lead the delivered message with that question restated plainly and addressed to whoever must decide, before the estimate body — don't leave it sitting as one bullet among others where it can be skimmed past. The same sentence still belongs inside `Pontos em aberto:` too, so the document stays self-contained on its own.

## Items not in the Figma

List (template item 7) the features that exist in the current product but are absent from the design, found during the cross-check in the general rules. Default posture for each: the item may or may not stay; the developer does not remove the existing implementation, only hides or drops the declaration that renders it, which is low effort and stays inside the total. Redesigning any of them (not just hiding it) needs a new estimate. Intro sentence, in Portuguese: "A estimativa assume ocultar ou retirar a exibição desses itens sem mexer na implementação; redesenhá-los exige nova estimativa."

## Complexity level

Template item 8 ends with `Complexidade: <nível>.`. Use one of these five standard levels. The word agrees with "Complexidade", so it is feminine. Complexity reflects risk, unknowns, and how sensitive the touched area is, not only the number of hours: a short task on a payment or checkout flow can still be `Alta`.

- `Baixa`: presentation, config, or content changes that reuse existing patterns; no new integrations; predictable outcome; little or no platform risk.
- `Média-Baixa`: mostly straightforward, with a few moving parts or one minor unknown; still low risk.
- `Média`: several components or a new integration that follows a known pattern; moderate scope; some platform behavior to handle; manageable unknowns.
- `Média-Alta`: broad or cross-cutting scope, a genuine platform unknown, or work near sensitive flows; meaningful risk that a platform-unknowns line, if included, should cover.
- `Alta`: security-sensitive work, or anything touching checkout, order, or payment; new architecture; significant unknowns; or large cross-cutting work where a regression is costly.

State the level plainly, with no parenthetical explanation and no exceptions to this format; the reasoning behind the level belongs in the diagnosis paragraph or `Pontos em aberto:`, not attached to the `Complexidade:` line itself.

## The budget

The heading `Sobre o orçamento:` introduces what the budget covers. One line per task with its time; the lines plus the item 5 `Tempo de orçamento` line must sum exactly to the headline total (see "Arithmetic and rounding discipline").

- **Always ask the user whether to include a line for platform-specific unknowns** (build or link errors, framework quirks, an undocumented config limit that only surfaces at runtime) before finalizing the budget — never decide unilaterally either way, regardless of complexity level, since your own risk read can be wrong in either direction. Ask in Portuguese and state your own assessment as part of the question, naming the specific reasons behind it: "acho que vale incluir uma linha de imprevistos de plataforma por causa de X, Y; quer que eu inclua?" or, when you see no real risk, "não vejo um risco de plataforma real aqui (motivo A, B); quer que eu inclua uma linha mesmo assim?". Do not fold it invisibly into other lines when included. Size it at about 10-15% of the coding total, rounded to a clean increment. The heading is always `Imprevistos de plataforma`, and the parenthetical adapts to the actual unknowns of this estimate (for example `(aplicação de cupom fora do checkout)`). A technical term is allowed in the parenthetical, but write it with enough context for a non-technical reader, the same way PII is written as "informações de identificação pessoal (PII)". Never use the word "buffer" and never internal jargon. Do not manufacture a justification just to fill the parenthetical: the unknown must be a genuine, nameable risk, never the mechanism itself when it is already confirmed mature and documented. If nothing genuine survives scrutiny, drop the line rather than force a reason.
- **Multiple viable implementation paths:** present each as a named variant. List the steps that are identical in scope and time across every variant once, under `Sobre o orçamento:`. Then list each variant's extra or different steps under `Específico da variante X`. Never restate a shared step inside a variant's list.
- **Variant lines price only the increment.** If a variant is described as "place and restyle" and placing is already a shared step, the variant line covers restyling only; its time must not re-include the shared step. This is the same rule as "never price the same work twice".
- **Leave genuinely undecided scope as its own bullet marked `a confirmar`,** with no number, rather than inventing a figure for it.

## Multi-store / multi-repo estimates

Applies whenever the same change is being priced across more than one store, theme, or repo (sibling brand sites sharing a white-label codebase, for example).

- **Default to the same base time per target** for the same modification, unless the user says otherwise or investigation shows one target's change is genuinely different (a divergent component, a missing file the others have, a different entity name). Confirming this equivalence is the multi-repo check in "Before writing a single number", not a separate step.
- **Combine `Sobre o orçamento:` into totals that already sum every target**, one line per task, not one block per target. State the convention once, for example `Sobre o orçamento (cada linha já soma as N lojas):`, rather than repeating "nas N lojas" on every line.
- **Add a `Tempo por loja: Xh cada.` line** (or one line per target, if the times genuinely differ) directly under `Complexidade:`. This answers "how much per target" without making the reader divide the total themselves.
- The `Tempo de orçamento` line (item 5) and any codebase-quality-point line are shared across the whole document; only the task lines that scale with the number of targets get multiplied.

## Investigation-first tasks

Applies whenever the estimate cannot carry an hour total yet, either because the core of the task is investigating an unknown cause or behavior (a bug with no confirmed diagnosis, an integration divergence, an unexplained platform behavior) and the fix cannot be scoped until the cause is found, or because the cause and fix are already fully confirmed but something else still blocks a number: execution belongs to another party, the durable fix belongs to a platform vendor, or the client still has to pick a direction (see the variants below). In every case, do not produce hour totals at all; the deliverable is the framing of what's blocking the estimate, and the real number comes after it.

What changes against the normal template:

- No hours anywhere in the document. Item 8 becomes the literal line `Tempo total estimado: A ser estimado após investigação.`, still followed by `Complexidade: <nível>.`.
- Omit `Sobre o orçamento:` entirely; there are no lines to sum.
- Omit `Tempo de orçamento:` too (item 5); there is no total to fold it into.
- Keep the diagnosis paragraph (item 2), phrased as unconfirmed hypotheses, naming the typical candidate causes for the reported pattern and stating that identifying the real cause is part of the task. Tone, in Portuguese: "Não foi possível confirmar a causa preliminar sem iniciar a investigação; os candidatos típicos nesse padrão são X, Y ou Z. Identificar a causa real é parte da tarefa."
- `Pontos em aberto:` keeps only points that change the scope or the deliverable (for example, the fix moving to the client's side turns the deliverable into a documented diagnosis). Drop any point that only qualified a budget line, since there is no budget. Drop a point that only restates something already in the briefing's own investigation checklist or DoD; list only what the briefing does not already cover. When the team cannot reproduce the issue and only the client holds the missing context (access to an external system, how an integration is actually implemented), phrase exactly those questions as the open points — they define what the assigned dev needs before starting.
- End with `Links:` (item 11).
- **Platform/vendor-tooling variant.** When the confirmed cause traces to a
  gap in the platform's own migration tooling or an official pattern the
  platform provides, rather than something specific to this site's
  application code, the investigation-first framing still applies even
  after the cause and a candidate fix are fully identified. The reasoning:
  the real, durable fix may belong upstream (the platform vendor's tooling
  or support), and a local one-off patch could be premature or incomplete
  if other sites share the same gap. In this case: item 8 reads
  `Tempo total estimado: Escopo de investigação, com
  possível abertura de chamado com o fornecedor da plataforma.`; the
  diagnosis paragraph states the confirmed cause plainly, not hedged, then
  adds one line flagging the vendor-escalation option; and any ready-made
  fix sketch (files, diffs, code) moves out of the estimate's body into a
  separate trailing note attached after the estimate (a comment alongside
  the `Links:` entry, not inside the section itself), kept apart from the
  narrative.

- **External-ownership handoff variant.** When the cause and the fix are both fully confirmed, but the entity that must execute it is a different team or organization (the client's own internal team, another agency, a third party) whose access to the relevant system was not verified, item 8 does not use `A ser estimado após investigação`, it states who must execute it: `Tempo total estimado: Ação a ser realizada por <time>, fora do escopo de <nosso time>.` Omit `Sobre o orçamento:` and `Tempo de orçamento:` (item 5) as well; there is no numeric total to fold either into. Do not give it a `Complexidade` rating unless one can actually be backed by verified facts; use `Complexidade: n/a.` otherwise. Keep the diagnosis paragraph confirmed and plain, this variant assumes the cause is already known, unlike the general investigation-first case. `Pontos em aberto:` must name who holds admin access to the relevant system as an open question directed at that team, plus any decision left for them to make (for example choosing between two valid fix approaches). A ready-made technical fix sketch stays out of the estimate's body, in its own `Nota técnica para quem for implementar:` section, the same placement rule as the platform/vendor variant's trailing note.

- **Client-decision variant.** When investigation confirms every applicable native/recommended option is already implemented at the value the platform's own documentation recommends, and no further scope exists until the client picks a direction (adjust one of those values, or build something different from what is already active), omit item 8 (`Tempo total estimado:` / `Complexidade:`) and item 9 (`Sobre o orçamento:`) entirely, there is nothing yet to total or rate. Unlike the two variants above, keep item 5 (`Tempo de orçamento:`) standalone with no total to fold into: it reflects real investigation already performed (for example, pulling the live configuration and checking it against documentation), not a placeholder for work that hasn't started. The diagnosis paragraph states the confirmed current state plainly, not hedged. `Pontos em aberto:` leads with the blocking decision itself, phrased as a direct question to the client, not a hedged ambiguity, in Portuguese: "já está tudo no valor recomendado pela documentação, o que se espera a partir daqui: ajustar algum desses valores, ou implementar uma proteção diferente das já ativas?"

- **External-vendor visibility variant.** When the reported behavior touches one or more externally-owned systems (a platform's own native connector or gateway, a marketplace's own seller platform) and the team holds admin access to none of them, do not scope internal investigation hours even though the cause itself was never confirmed. The reasoning: without visibility into the vendor's side, an internal investigation would just restate the symptom already in the briefing. Item 8 reads `Tempo total estimado: Ação a ser realizada via abertura de chamado com <fornecedor(es)>, fora do escopo de <nosso time>.` Omit `Sobre o orçamento:` and `Tempo de orçamento:` (item 5) as well, and use `Complexidade: n/a.`, since there is no confirmed cause to back a rating. Unlike the external-ownership handoff variant, the diagnosis paragraph stays unconfirmed and hedged (the cause genuinely is not known); it states plainly that the access gap, not the cause, is why the team is recommending escalation instead of an internal investigation. `Pontos em aberto:` must name who (client or internal team) opens each ticket, and what evidence it needs (an example affected item, a suspected date of change). Once a ticket is opened, have the assigned dev share its number back for internal tracking, and add any stakeholder who needs visibility as a watcher on it.

Reference example (VTEX, price divergence between ERP and commercial policy):

```
Pelo briefing, a VTEX recebe o preço do ERP, mas a política comercial mantém o valor anterior. Não foi possível confirmar a causa preliminar sem iniciar a investigação; os candidatos típicos nesse padrão são preço fixo cadastrado na política comercial sobrepondo o preço-base, regra de arredondamento ou markup da política, ou atraso de indexação. Identificar a causa real é parte da tarefa.

Pontos em aberto:
- Os SKUs de exemplo podem já estar corrigidos quando a tarefa iniciar. O dev vai solicitar ao cliente um exemplo real e ativo da divergência.
- Não foi informado se o ERP envia apenas o preço-base ou também preços fixos por política comercial. Isso muda onde está a causa; a investigação cobre os dois cenários.
- Não foi informado quantas políticas comerciais a loja possui.
- Se a causa estiver do lado do ERP (integração mantida pelo cliente ou parceiro), a correção sai deste escopo e a entrega passa a ser o diagnóstico documentado, com o ajuste a ser feito do lado do ERP.

Tempo total estimado: A ser estimado após investigação.
Complexidade: Média-Alta.

Links:
https://help.vtex.com/pt/docs/tutorials/configurar-regra-de-preco-para-politica-comercial
https://help.vtex.com/pt/docs/tutorials/arredondamento-de-precos
https://help.vtex.com/pt/docs/tutorials/configuracoes-de-precos
```

Reference example (Shopify, drawer search finding products the /search page does not):

```
Não foi possível confirmar a causa preliminar sem iniciar a investigação. Os candidatos possíveis para esse padrão (busca preditiva encontra produtos que a página de resultados não encontra, mesmo com os mesmos parâmetros) são: alguma configuração ou app de busca ativo na loja que sobrepõe o comportamento padrão documentado pela Shopify, um problema na busca da loja desatualizado ou divergente entre os dois mecanismos, ou uma diferença de comportamento entre a Predictive Search API e o Search nativo não coberta pela documentação pública.

Plataforma: Shopify.

Arquivos já existentes que serão a base do trabalho:
- assets/predictive-search.js: monta a chamada à Predictive Search API e a URL do botão "Ver todos os resultados"
- sections/predictive-search.liquid: template do drawer de busca
- sections/main-search.liquid: página de resultados, usa o objeto nativo search do Liquid/Shopify

Pontos em aberto:
- Eu não tenho acesso ao admin da loja para verificar se há algum app de busca (por exemplo Search & Discovery, ou outro) instalado ou configurado, o que pode estar alterando o comportamento padrão documentado pela Shopify. Isso precisa ser validado pelo dev que vai executar a tarefa.
- A documentação oficial (https://shopify.dev/docs/storefronts/themes/navigation-search/search.md) diz que options[prefix]=last já é o padrão tanto na busca preditiva quanto no /search, o que deveria fazer "harl" encontrar "harley" nos dois lugares; isso não bate com o observado e pode exigir abertura de chamado com o suporte Shopify caso a causa não seja encontrada no lado da loja.
- O caso da busca por "HD" retornando um produto pode ser um sinal de relevância/histórico de busca; investigar se há algo cadastrado nesse produto diferente dos demais.

Tempo total estimado: A ser estimado após investigação.
Complexidade: Média-Alta.

Links:
- https://shopify.dev/docs/storefronts/themes/navigation-search/search.md
- https://apps.shopify.com/search-and-discovery?locale=pt-BR
```

# Auxiliary tools and MCPs

This skill leans on a few external tools and connectors. When one is needed but not available, show the user how to add it and ask before installing; never assume it is present. This section holds the technical how-to; the places that use each tool only reference it.

- **ponytail** (skill) — apply it in full mode while sizing and hunting for reuse (see "Look for existing code to reuse before assuming a build-from-scratch cost"): the laziest solution that actually works is the baseline you cost. If it is not installed, offer to install it: in Claude Code, `/plugin marketplace add DietrichGebert/ponytail` then `/plugin install ponytail@ponytail` (two separate prompts); in Codex, `codex plugin marketplace add DietrichGebert/ponytail` then `codex plugin add ponytail@ponytail`. Repo: `https://github.com/DietrichGebert/ponytail`.
- **humanizer** (skill) — rewords the final estimate so it reads like the writer (see "Humanize the final text"). If not installed, offer to install it: `/plugin marketplace add blader/humanizer` then `/plugin install humanizer@humanizer`.
- **Tarefy** (MCP connector) — pulls a task's details and comments to use as the base briefing (see "Pull the task from Tarefy when given its link or id"). When it is not connected, take the same posture as humanizer: proactively suggest adding it and guide the user through the process, do not just point at the settings. Start the OAuth flow with `mcp__claude_ai_Tarefy__authenticate`, share the authorization URL it returns, then finish with `mcp__claude_ai_Tarefy__complete_authentication` once the user authorizes. Connector address: `https://mcp.tarefy.com/mcp`. To connect manually in Claude: open Settings > Connectors > Browse, find "Tarefy", connect, then choose the company and the access level; the read-only level (`Ler`) is enough for this skill. Requires an active Tarefy collaborator account and the `Conectar o Tarefy a assistentes de IA` permission enabled; authentication is OAuth 2.1 + PKCE. References: the Claude directory `https://claude.ai/directory/tarefy` and the help page `https://tarefy.com/ajuda/conector-ia/`.

# Maintaining this skill

This file gets edited over time as real estimates surface gaps. Before writing a change to it:

- Read the whole file first, not just the section being touched, to check whether an existing rule already covers the same ground.
- Match the surrounding section's own style (terseness, whether it uses a Portuguese-quote example, table row length) instead of dropping the new text in verbatim.
- Place a new rule or table row next to its closest existing relative, not appended at the end, unless there is genuinely no natural neighbor.
- Reread the new text for redundancy, using this checklist:
  - Grep the file for the new text's key terms before writing; read every existing hit in full, not just skim for the word.
  - Check whether an anchor section already elaborates the same point in more detail; if so, point to it instead of restating it in the general rules.
  - Compare the new bullet against its immediate neighbors (the ones right before and after) for an overlapping "when to apply" condition, not just overlapping wording.
  - If two bullets end up teaching the same underlying lesson from different angles (e.g. "don't trust the repo alone, check what's live" applied to two different kinds of claims), decide explicitly whether they're distinct enough to stay separate or should merge — don't leave both by default.
- Present the exact diff and wait for confirmation before writing, same as any other state-changing edit.
