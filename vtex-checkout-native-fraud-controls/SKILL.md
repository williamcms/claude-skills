---
name: vtex-checkout-native-fraud-controls
description: "Apply when analyzing, configuring, or estimating work on VTEX's native checkout anti-fraud/anti-bot controls (reCAPTCHA v2/v3, blockRecaptchaV2, minimumPurchaseDowntimeSeconds, cartAgeToUseNewCardSeconds, requiresLoginToPlaceOrder, requiresLoginToUseNewCard), or diagnosing bot/card-testing attacks on VTEX checkout. Covers what each control actually does, its real scope (per-buyer/session vs account-wide), which toggles require a VTEX support ticket, the discrete recaptchaMinScore values, and known side effects. Use for fraud-mitigation tickets, checkout security estimates, or explaining why a native lock isn't working."
---

# VTEX Checkout Native Fraud Controls

## When this skill applies

- Diagnosing "bot attack" / "card testing" complaints on VTEX checkout
- Deciding which reCAPTCHA version (v2/v3) or account-level toggle to recommend
- Estimating or scoping work to adjust orderForm anti-fraud fields
- Explaining why a native lock "isn't working" or "doesn't help"

Do not use this skill for:
- Payment gateway/acquirer antifraude rules (3DS, BIN blocking, velocity rules) — that lives in the payment provider's own console, not the orderForm
- WAF/CDN-level IP rate limiting — that's edge infra (Cloudflare and similar), separate from the orderForm

## Recognizing the attack pattern (card testing / bot attack)

Typical signature, confirmed in a real VTEX support case: many orders cancelled within minutes of each other, recurring data (same delivery ZIP/CEP) with varying order values, random and different customer names/emails per attempt, and traffic that may hit the public Checkout API directly rather than only the storefront UI (so front-end-only mitigations don't fully cover it).

## The orderForm fields (reference)

| Field | What it does | Scope | Notes |
|---|---|---|---|
| `requiresLoginToPlaceOrder` | Requires the buyer to be authenticated, with the same email used when the cart was created, to place an order | Account-wide | Disables SmartCheckout for **every** customer, the single highest-conversion-impact lock in this list |
| `requiresLoginToUseNewCard` | Requires authentication only to add a new (not-yet-saved) card | Account-wide | Lower conversion impact than the field above; targets exactly the "new card" step bots rely on |
| `minimumPurchaseDowntimeSeconds` | Minimum interval between purchases | **Per buyer/session** | Confirmed by VTEX support: a bot rotating emails/sessions per attempt is not slowed by this at all, it only throttles a single persistent identity retrying repeatedly |
| `cartAgeToUseNewCardSeconds` | Minimum time a cart must exist before a new card can be used on it | Per cart | |
| `recaptchaValidation` | `"always"` validates 100% of card-paid orders (score-based, v3); `"vtexCriteria"` is VTEX's own recommended criteria for when to challenge (mixes in the v2 checkbox for flagged/suspicious purchases); `"never"` disables it | Account-wide | `"always"` is the documented choice when using score-based validation |
| `recaptchaMinScore` | Threshold VTEX compares the per-request Google score against | Per key > per account > VTEX default | **Configurable values are discrete: 0.1, 0.3, 0.7, 0.9** (VTEX default 0.7). Confirmed by VTEX's own Field Engineering team; don't assume a continuous 0.0-1.0 range is settable, and treat any live config showing an out-of-list value (e.g. `1`) as worth re-verifying live rather than taking at face value |
| `recaptchaKeys` | Custom reCAPTCHA site key | — | `null` means VTEX's own default implementation is used. Extra implementation (a custom key) is only required for **headless/native-app** integrations ("reCAPTCHA V3 Full Headless"); the standard native checkout already gets v3 with no extra work |
| `blockRecaptchaV2` | Forces every validation through v3 only, ignoring any v2 key/flow | Account-wide | Not a self-service API/admin field as of mid-2026, requested via VTEX support ticket. **v2 and v3 can be simultaneously active on a native checkout even when `recaptchaValidation` is `"always"`**, confirmed in a real case where both showed as independently enabled; v2 stays a real, exploitable fallback path until this is explicitly blocked |

## What the reCAPTCHA score actually measures

The score (0.0-1.0, continuous, computed per request by Google) measures bot-likelihood, not payment or card legitimacy, per Google's own reCAPTCHA v3 documentation (https://developers.google.com/recaptcha/docs/v3): "1.0 is very likely a good interaction, 0.0 is very likely a bot." It does not by itself stop manually-operated fraud by a real human using stolen card data, since that traffic behaves like a genuine human to Google's model. It IS effective against automated/scripted card testing: in a documented real case, enabling `blockRecaptchaV2` alongside the already-active v3 stopped a recurring, clearly bot-driven card-testing attack (confirmed by the client within days of the change).

When the score falls below the threshold, VTEX support describes the effect as "pode exigir desafio adicional ou bloquear a finalização" (may require an additional challenge or block completion), while VTEX's own recaptcha guide states flatly that the order cannot be placed. Treat the harder wording as the documented baseline, but verify live if the exact behavior matters for a specific case.

## Known caveats / gotchas

- **Per-buyer/session scope.** Several of these locks reset with every new synthetic identity a bot uses. They slow down one persistent attacker, not one that rotates emails/sessions per attempt, which is the common pattern in card testing.
- **`blockRecaptchaV2` is ticket-only, not instant.** Expect at least one confirmation round-trip (support asks you to confirm no flow/customization depends on v2, since those break once it's blocked) plus VTEX's own turnaround time — observed ~10 days end-to-end in one real case.
- **Unconfirmed but reported side effect.** In one account, a "can't add product to cart" issue (fixed only by the affected user clearing cache) surfaced about 6 days after `blockRecaptchaV2` was enabled. VTEX support denied causation without a deep investigation (the timing didn't line up with when the issue was first reported). Worth a monitoring note after applying this change, not a proven risk, and not a reason to avoid it.
- **VTEX Shield is a generic security upsell, not a fraud tool by itself.** It's a real, paid VTEX add-on (Security Monitor, WAF, Data Protection Plus, custom SSL, mTLS). VTEX support routinely pitches it at the close of fraud/bot tickets, unprompted, even when the actual resolution had nothing to do with any of its five modules, none of which specifically target checkout/order/card fraud. Verify fit before including it in a fraud-mitigation scope.
- **Some official doc pages referenced by VTEX support itself have gone 404.** A "boas práticas contra ataques virtuais" tutorial was confirmed broken by VTEX support as of mid-2026. Don't assume a given help.vtex.com URL is still live without checking.

## How to check/change the current config

```
GET/POST https://{accountName}.vtexcommercestable.com.br/api/checkout/pvt/configuration/orderForm
-H "X-VTEX-API-AppKey: {appKey}"
-H "X-VTEX-API-AppToken: {appToken}"
```

Requires an AppKey/AppToken with Checkout permission. For a POST, start from the GET response and change only the fields you mean to touch. Validate in the Beta (`{account}.myvtex.com`) environment before applying to production.

**Faster alternative for a one-off check**: `vtex login {accountName}` then `vtex local token` generates a user token (copied to the clipboard, valid 24h) without going through License Manager to create an AppKey/AppToken. Send it via the `VtexIdclientAutCookie` header instead of `X-VTEX-API-AppKey`/`AppToken`. It inherits the logged-in user's own License Manager permissions rather than a dedicated key policy, so confirm that user actually has the Checkout resource permission (e.g. Shopping Cart Full Access) before relying on it for the orderForm config route.

## Reference

- reCAPTCHA guide: https://developers.vtex.com/docs/guides/recaptcha
- Checkout API — OrderForm Configuration: https://developers.vtex.com/docs/api-reference/checkout-api
- reCAPTCHA validation now follows orderForm config for all requests (2023): https://developers.vtex.com/docs/guides/2023-07-05-recaptcha-validation-will-now-follow-orderform-configuration-for-all-requests
- reCAPTCHA v3 support for integrations (2023): https://developers.vtex.com/docs/guides/2023-06-28-recaptcha-v3-support-available-for-integrations
- Implementing reCAPTCHA in integrations (headless/mobile key): https://developers.vtex.com/docs/guides/implementing-recaptcha-in-integrations
- Google reCAPTCHA v3 overview: https://developers.google.com/recaptcha/docs/v3
- SmartCheckout security: https://help.vtex.com/pt/tutorial/smartcheckout-security--3SrJuuhrqwePUg1rp1exfB
- VTEX Shield: https://help.vtex.com/pt/docs/tutorials/vtex-shield
- API authentication using user tokens (`vtex local token`): https://developers.vtex.com/docs/guides/api-authentication-using-user-tokens

## Sourcing note

Field mechanics, the discrete `recaptchaMinScore` values, and the caveats above were confirmed in a real VTEX support ticket (VTEX Field Software Engineering Team, tickets on a bot/card-testing attack, mid-2026), not only from public docs. Some public doc pages that VTEX support itself linked were reported broken at the time; re-check any URL here before relying on it if enough time has passed.
