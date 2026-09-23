# Buildertrend Form Submit Listener

Pushes a `buildertrend_form_submit` event to the dataLayer when a visitor
submits a **Buildertrend Lead Capture Form** embedded on your site, so you can
fire GA4, Google Ads, and Meta tags from a GTM Custom Event trigger.

## How it works (and the trade-off)

The Buildertrend form lives in a cross-origin `buildertrend.net` iframe, so
GTM's built-in Form Submission trigger can't see it. The form doesn't send the
parent page a "submitted" message either. It only posts resize messages
(`{height: N}`) so the host page can size the iframe.

The listener uses those resize messages:

- While the form is on screen, `N` is the form's height (hundreds of px).
- When no form is rendered, `N` is Buildertrend's bare padding value, `50`.
  After the form has loaded, that only happens on the success screen ("Your
  information was successfully submitted.").

So once the full form has been seen, a later `{height: 50}` from the same
iframe counts as a successful submit. Validation errors and failed submits
leave the form on screen, so they don't fire.

**The trade-off:** this reads Buildertrend's undocumented resize behavior.
If they change it, the listener goes quiet until it's re-tuned (usually just
the `EMPTY_HEIGHT` value at the top of the script). Re-verify in GTM Preview
after Buildertrend updates.

## Before you install: check the form's redirect setting

If the Lead Capture Form is set to **redirect to a thank-you page** after
submit, Buildertrend sends the whole browser to that URL straight away and
never shows its success screen, so **this listener won't fire**. In that case,
skip the listener and use a Page View trigger on the thank-you page URL.

## dataLayer output

| dataLayer event | dataLayer variables |
|---|---|
| `buildertrend_form_submit` | `bt_event_id`, `bt_builder_id`, `bt_form_origin` |

- `bt_event_id`: unique per submit. Use it as the Meta Pixel `eventID` for CAPI deduplication.
- `bt_builder_id`: the `builderID` from the iframe's `src` (if found).
- `bt_form_origin`: the iframe origin that sent the message (for debugging).

**No field values are available** (no email, phone, or name). The iframe is
cross-origin and Buildertrend doesn't send them to the parent page.

## Installation

1. GTM → **Tags** → **New** → **Custom HTML**.
2. Paste the contents of `buildertrend-listener.html`.
3. **Triggering:** All Pages (Page View). It only reacts to messages from
   `buildertrend.net`, so it's safe to run everywhere. You can limit it to
   pages that have the form if you'd like.
4. Name it `cHTML - Buildertrend Form Listener` and save.
5. Create a **Custom Event** trigger with the Event Name `buildertrend_form_submit`.
6. Create Data Layer Variables for any of the `bt_*` keys you need
   (e.g. `DLV - bt_event_id`).
7. Attach your conversion tags (GA4 `generate_lead`, Google Ads conversion,
   Meta Lead with `eventID` = `{{DLV - bt_event_id}}`) to the trigger from step 5.

## Verify

1. GTM **Preview** → submit a real test lead through the form.
2. Confirm `buildertrend_form_submit` appears once, after the "successfully
   submitted" message shows.
3. Delete the test lead in Buildertrend.

If nothing fires, open DevTools → Console and paste:

```js
window.addEventListener('message', function (e) { console.log(e.origin, e.data); });
```

Then reload and submit again. You should see `{height: …}` messages from
`https://buildertrend.net`, ending in `{height: 50}` after the submit. If the
final value isn't `50`, update `EMPTY_HEIGHT` in the script to match.

## Known limitations

- **Relies on undocumented behavior.** A Buildertrend update can silently stop events.
- **Redirect-after-submit forms aren't detected.** Use a thank-you page trigger instead (see above).
- **No field values.** The payload has no email, phone, or name, so Meta CAPI matching is limited to what your other tags collect.
- **Custom CSS edge case.** If custom CSS in the form hides the whole form element, its height reads as zero and the listener could fire early. Standard styling is fine.

## Disclaimer

Provided as-is under the MIT License. Test thoroughly before relying on it for
revenue attribution.

---

**Packaged by W.H. Boggs** — [whboggs.com](https://whboggs.com)
