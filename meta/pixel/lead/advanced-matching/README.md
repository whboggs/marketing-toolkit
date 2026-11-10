# Meta Pixel — Lead Event (Advanced Matching)

The [basic Lead event](../basic/) plus **advanced matching**: the lead's email, phone, first and last name, and a persistent `external_id` ride along with the event so Meta can match it to a Facebook / Instagram account. Same custom parameters, same `eventID` dedupe key.

## Why use this over the basic Lead event

Meta can only use a matched event for attribution and ad optimization. The basic tag sends the Pixel's automatic signals (IP address, user agent, `_fbp`, `_fbc`) and nothing about the person. Email and phone are the strongest match keys Meta accepts, and this tag adds them.

If the Pixel has the **Conversions API Gateway** enabled, the gateway forwards the same hashed email, phone, name, and `external_id` with its server copy of the event. That copy is the one Event Match Quality scores, so the score improves without any server-side work.

## Why the user info goes in `init`, not the Lead event

`fbq('track', …)` has no slot for user info:

- Put in the event parameters (third argument), the values go out as **plain-text custom data** (`cd[em]=jane@example.com`). Meta doesn't match on custom data, and sending unhashed contact info can get the Pixel flagged and its events restricted.
- Put in the options (fourth argument, beside `eventID`), the Pixel **silently drops them**.

Calling `fbq('init', …)` again with the user data is Meta's advanced-matching method. It does **not** fire another PageView; the Pixel hashes the values and attaches them to the Lead event that follows.

## Prerequisites

- The **base Meta Pixel** must load before this tag fires (it calls `fbq`).
- A Constant variable **`Meta - Pixel ID`** set to the Pixel's ID. It **must** match the base Pixel's ID; a different ID starts a second Pixel.
- **`DLV - Email`**, **`DLV - Phone`**, **`DLV - First Name`**, **`DLV - Last Name`**: Data Layer Variables pointing at whatever your form listener pushes (see the next section).
- **`1PC - _ga`**: 1st-Party Cookie variable reading `_ga`, the Google Analytics client ID cookie. Requires GA4 on the site.
- `cJS - Traffic Source`, `cJS - Ad Placement`, [`cJS - Custom Event ID`](../../../../gtm-essentials/variables/cjs-custom-event-id/) (plus its `DLV - gtm.uniqueEventId` companion), and the built-in `Page Path`: the same requirements as the basic tag.

All of these except the four `DLV -` variables and `Meta - Pixel ID` come with [GTM Essentials](../../../../gtm-essentials/). The [GTM Starter Kit](../../../../gtm-essentials/gtm-starter-kit/) includes all of them.

## Point the DLV variables at your form

Each `DLV -` variable is a **Data Layer Variable**. Set its *Data Layer Variable Name* to the key your form listener pushes:

| Platform (listener) | `DLV - Email` | `DLV - Phone` | `DLV - First Name` | `DLV - Last Name` |
|---|---|---|---|---|
| GoHighLevel ([form-submit bridge](../../../../go-high-level/gtm-tags/form-submit-bridge/)) | `ghl_email` | `ghl_phone` | `ghl_first_name` | `ghl_last_name` |
| Wix ([form listener](../../../../wix/form-submission-listener/)) | `wix_email` | `wix_phone` ⚠️ | `wix_first_name` | `wix_last_name` |
| Anything else | the key your listener pushes | | | |

A variable that resolves empty or `undefined` is skipped, so a form without a last-name field still works.

**Gravity Forms, Elementor, and the native `gtm.formSubmit` trigger** don't push contact details into the dataLayer (the [Gravity Forms listener](../../../../wordpress/gravity-forms/listener/) only captures hidden fields). For those, turn on **Automatic Advanced Matching** in Events Manager → the Pixel → Settings, which reads email and phone from the form on submit. It can't see forms inside an iframe (GHL embeds, Jotform, Calendly).

## ⚠️ `DLV - Phone` must include the country code

Meta only matches phone numbers that include the country code. The Pixel strips symbols on its own (`+1 (801) 555-1212` → `18015551212`), but it **cannot add a missing country code**: `(801) 555-1212` is hashed as `8015551212` and won't match. Make sure the value `DLV - Phone` returns already includes it:

- **GoHighLevel:** already includes it (GHL stores `+18015551212`).
- **Wix:** the listener keeps whatever digits the visitor typed. A number typed without a country code reaches the tag without one.

If the source can't include it, add a Custom JavaScript variable that does (for example, a US/Canada-only site can prefix `1` to any 10-digit number) and use that as `DLV - Phone`.

## Installation

1. **Tags → New → Custom HTML.**
2. Paste the contents of `chtml-lead-advanced-matching.html`.
3. *(Optional)* Uncomment the `form_name` line if you use the `LUT - Convert Form ID to Text` variable.
4. **Triggering:** your form-success trigger (for example, the GHL bridge's `ghl_form_submit` Custom Event).
5. Name it `Meta - Event - Lead - Form Submit (Advanced Matching)` and save.
6. **Pause or delete the basic Lead tag** if it uses the same trigger. Otherwise every lead is counted twice.

## Verify

1. GTM **Preview**, then submit a test form with your own real email and phone.
2. DevTools → **Network** → filter `facebook.com/tr` → the `ev=Lead` request should carry `ud[em]`, `ud[ph]`, `ud[fn]`, `ud[ln]`, and `ud[external_id]` as 64-character hashes, plus `eid` (the event ID).
3. Events Manager → **Test events** shows the Lead with its customer information parameters.
4. After a few days, Events Manager → **Overview** → Lead → **Event match quality** shows coverage per match key. With low lead volume, compare 7-day windows rather than day-to-day; a small sample swings a lot.

## Payload

| Parameter | Source | Notes |
|---|---|---|
| `em` | `{{DLV - Email}}` | Advanced matching. Hashed by the Pixel. |
| `ph` | `{{DLV - Phone}}` | Advanced matching. Hashed by the Pixel. Must include the country code. |
| `fn` / `ln` | `{{DLV - First Name}}` / `{{DLV - Last Name}}` | Advanced matching. Hashed by the Pixel. |
| `external_id` | `{{1PC - _ga}}` | Advanced matching. The same visitor keeps the same ID across visits. |
| `lead_type` | Hardcoded `Form Submit` | Custom param. |
| `content_name` | `{{Page Path}}` | Meta standard param. |
| `traffic_source` | `{{cJS - Traffic Source}}` | Custom param. |
| `ad_placement` | `{{cJS - Ad Placement}}` | Custom param. |
| `eventID` | `{{cJS - Custom Event ID}}` | Browser/server dedupe key, passed in the event options. |

The contact variables are deliberately **unquoted** in the script, so an apostrophe in a value (O'Brien) can't break it.

## Using the same `external_id` server-side

If a CRM or Zapier integration sends Lead events to the Conversions API, it should send the same `external_id`. With MTK Attribution on the site, add a field mapped to `cookie:_ga` so the value lands on the lead record next to `mtk_fbc` / `mtk_fbp`.

## Known limitations

- **Safari caps script-set cookies at 7 days** of no visits, `_ga` included, so a returning Safari visitor may get a new `external_id`. Email and phone are unaffected.
- **No consent mode integration.** Gate the tag on marketing consent in your CMP.
- **One lead type per tag**, as with the basic tag.

## Disclaimer

This script packages Meta's standard Pixel calls for distribution via the Marketing Toolkit. The underlying Pixel code is provided by Meta Platforms, Inc. and subject to their terms.

Provided as-is under the MIT License. Test thoroughly before relying on it for revenue attribution.

---

**Packaged by W.H. Boggs** — [whboggs.com](https://whboggs.com)
[→ Free Meta Ads audit](https://whboggs.com/audit)
