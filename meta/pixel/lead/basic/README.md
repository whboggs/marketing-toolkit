# Meta Pixel — Lead Event

Fires a Meta Pixel `Lead` event with custom parameters sourced from GTM variables. It's a self-contained Custom HTML tag: paste it into GTM, and re-paste this file to pick up updates.

> **Want a higher Event Match Quality?** Use the [Advanced Matching Lead event](../advanced-matching/) instead — same event, plus the lead's email, phone, name, and a persistent `external_id`.

## What it does

1. Reads payload values from GTM variables (traffic source, page path, optional form name)
2. Calls `fbq('track', 'Lead', payload, { eventID })`

## Prerequisites

- The **base Meta Pixel** must be installed and firing on All Pages — Page View, before this tag fires. Without it, `fbq` is undefined and the tag throws an error without sending the event.
- A GTM variable named `cJS - Traffic Source` returning last-touch traffic source (Custom JavaScript variable).
- A GTM variable named `cJS - Ad Placement` returning the ad placement (Custom JavaScript variable).
- A GTM variable named [`cJS - Custom Event ID`](../../../../gtm-essentials/variables/cjs-custom-event-id/) returning a globally unique event ID (Custom JavaScript variable, plus its `DLV - gtm.uniqueEventId` companion) — sent as `eventID` so Meta can dedupe browser Pixel and Conversions API events.
- Built-in GTM variable `Page Path` enabled.
- *(Optional)* A Lookup Table variable named `LUT - Convert Form ID to Text` mapping form IDs to human-readable form names — uncomment the line in `chtml-lead-config.html` to send it.

## Installation

### Step 1: Create the Custom HTML tag

In GTM, go to **Tags** → **New**:

1. Click **Tag Configuration** → choose **Custom HTML**
2. Paste the contents of `chtml-lead-config.html` into the HTML field
3. *(Optional)* Uncomment the `form_name` line if you have the `LUT - Convert Form ID to Text` variable set up
4. Click **Triggering** → choose your form-success trigger (e.g., `Custom Event - GF Form Success` from the Gravity Forms listener, or any other trigger that signals a lead)
5. Name the tag: `Meta - Event - Lead - Form Submit`
6. Save

### Step 2: Preview and publish

1. Click **Preview** to test in Tag Assistant
2. Submit a form on your site — verify the Lead event fires by:
   - Checking the Network tab in DevTools for a request to `facebook.com/tr` with `ev=Lead`
   - Using the Meta Pixel Helper Chrome extension (should show a `Lead` event with your custom parameters)
3. Once verified, publish your container

## Payload structure

The event is sent with the following parameters:

| Parameter | Source | Notes |
|---|---|---|
| `lead_type` | Hardcoded `Form Submit` | Custom param — change in `chtml-lead-config.html` if this tag fires for a different lead type (e.g., `Phone Call`) |
| `traffic_source` | `{{cJS - Traffic Source}}` | Custom param — last-touch traffic source |
| `content_name` | `{{Page Path}}` | Meta standard param — page where the lead occurred |
| `ad_placement` | `{{cJS - Ad Placement}}` | Custom param — ad placement |
| `form_name` | `{{LUT - Convert Form ID to Text}}` | *(Optional)* Custom param — human-readable form name |
| `eventID` | [`{{cJS - Custom Event ID}}`](../../../../gtm-essentials/variables/cjs-custom-event-id/) | Meta's browser/server dedupe key — passed in the event options (the second object passed to `fbq`), not the custom-data payload |

## Known limitations

- **One lead type per tag**: `lead_type` is hardcoded in the tag. Duplicate the tag for different lead types (Phone Call, Demo Request, etc.) and edit the value.
- **Event deduplication needs both sides**: the tag sends `eventID` from [`cJS - Custom Event ID`](../../../../gtm-essentials/variables/cjs-custom-event-id/), but Meta only dedupes if your server-side CAPI integration sends the **same** value as `event_id` for the same event.
- **No consent mode integration**: Handle in your CMP separately.

## Disclaimer

This script packages Meta's standard Pixel snippet for distribution via the Marketing Toolkit. The underlying Pixel code is provided by Meta Platforms, Inc. and subject to their terms.

Provided as-is under the MIT License. Test thoroughly before relying on it for revenue attribution.

---

**Packaged by W.H. Boggs** — [whboggs.com](https://whboggs.com)
[→ Free Meta Ads audit](https://whboggs.com/audit)
