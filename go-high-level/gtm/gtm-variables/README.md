# GHL GTM Variables

GTM Container Export (JSON)
Last updated: November 2026

---

## Overview

`ghl-gtm-variables.json` is a Google Tag Manager **container export** that
creates every variable from the
[GHL Tracking Template](../gtm-tags/ghl-tracking-template.json) in one import —
24 user-defined variables plus the 6 Form built-in variables — without any of
the template's tags or triggers. Use it when a container already has its own
tags and you only need the GHL variables, or to repair variables that were
deleted or renamed.

The variables are copied unchanged from the template, so the GHL tags in
[`../gtm-tags/`](../gtm-tags/) resolve against them by name.

---

## How to Import

1. GTM → **Admin → Import Container**.
2. Choose `ghl-gtm-variables.json` and select your workspace.
3. Pick **Merge** and **Rename conflicting** so nothing existing is
   overwritten.
4. Preview, then Confirm.
5. Replace the two constants (`GA4 Measurement ID`, `Meta Pixel ID`) with the
   client's real IDs.

Placeholder account/container IDs are remapped into whatever container you
import into, so the file works in any account.

---

## Variables

Data Layer Variables all use **Data Layer Version 2** with no default value.
Most `ghl_*` keys are pushed by the
[Form Submit Bridge](../gtm-tags/form-submit-bridge/) (`ghl_email` and
`ghl_phone` also by the [Chat Widget Bridge](../gtm-tags/chat-widget-bridge/)).
`ghl_form_id`, `ghl_calendar_id`, `ghl_appointment_id`, and
`ghl_booking_data` are not pushed by either bridge — they stay empty unless
another push on the site sets them.

| Variable Type | Variable Name | Configuration |
|---|---|---|
| Constant | `GA4 Measurement ID` | Value: `REPLACE_WITH_GA4_MEASUREMENT_ID` |
| Constant | `Meta Pixel ID` | Value: `REPLACE_WITH_META_PIXEL_ID` |
| Data Layer Variable | `DLV - GHL Form ID` | Data Layer Variable Name: `ghl_form_id` |
| Data Layer Variable | `DLV - GHL Form Name` | Data Layer Variable Name: `ghl_form_name` |
| Data Layer Variable | `DLV - GHL Submission ID` | Data Layer Variable Name: `ghl_submission_id` |
| Data Layer Variable | `DLV - GHL Event ID` | Data Layer Variable Name: `ghl_event_id` |
| Data Layer Variable | `DLV - GHL Email` | Data Layer Variable Name: `ghl_email` · Format Value → Convert undefined to: `""` |
| Data Layer Variable | `DLV - GHL Phone` | Data Layer Variable Name: `ghl_phone` |
| Data Layer Variable | `DLV - GHL First Name` | Data Layer Variable Name: `ghl_first_name` |
| Data Layer Variable | `DLV - GHL Last Name` | Data Layer Variable Name: `ghl_last_name` |
| Data Layer Variable | `DLV - GHL Country` | Data Layer Variable Name: `ghl_country` |
| Data Layer Variable | `DLV - GHL Location ID` | Data Layer Variable Name: `ghl_location_id` |
| Data Layer Variable | `DLV - GHL Calendar ID` | Data Layer Variable Name: `ghl_calendar_id` |
| Data Layer Variable | `DLV - GHL Appointment ID` | Data Layer Variable Name: `ghl_appointment_id` |
| Data Layer Variable | `DLV - GHL Form Data` | Data Layer Variable Name: `ghl_form_data` |
| Data Layer Variable | `DLV - GHL Booking Data` | Data Layer Variable Name: `ghl_booking_data` |
| 1st-Party Cookie | `1PC - _fbp` | Cookie Name: `_fbp` · URI-decode cookie: off |
| 1st-Party Cookie | `1PC - _fbc` | Cookie Name: `_fbc` · URI-decode cookie: off |
| Custom JavaScript | `cJS - GHL Email (Normalized)` | Trims and lowercases `{{DLV - GHL Email}}` for Meta / Google matching: `function() { var email = {{DLV - GHL Email}}; if (!email \|\| typeof email !== 'string') return undefined; return email.trim().toLowerCase(); }` |
| Custom JavaScript | `cJS - GHL Phone (E.164)` | Strips non-digits from `{{DLV - GHL Phone}}` and prepends US country code `1` to 10-digit numbers: `function() { var phone = {{DLV - GHL Phone}}; if (!phone \|\| typeof phone !== 'string') return undefined; var digits = phone.replace(/\D/g, ''); if (digits.length === 0) return undefined; if (digits.length === 10) digits = '1' + digits; return digits; }` |
| Custom JavaScript | `cJS - Event Source URL` | Page URL sent as Meta CAPI `event_source_url`: `function() { return window.location.href; }` |
| Custom JavaScript | `cJS - User Agent` | Browser user agent sent as Meta CAPI `client_user_agent`: `function() { return navigator.userAgent; }` |
| Custom JavaScript | `cJS - Traffic Source` | Last-touch traffic source. Long script — identical to `cJS - Traffic Source` in [GTM Essentials](../../../gtm-essentials/variables/cjs-traffic-source/) |
| Custom JavaScript | `cJS - Placement` | `utm_placement`, persisted per session. Long script — identical to `cJS - Ad Placement` in [GTM Essentials](../../../gtm-essentials/variables/cjs-ad-placement/) |

> The one-line scripts above are compacted for the table. The commented
> versions are in `ghl-gtm-variables.json`. `cJS - GHL Phone (E.164)` assumes
> US numbers — change the default country code for non-US clients.

### Built-in variables enabled

Form Element, Form Classes, Form ID, Form Target, Form URL, Form Text.

---

## Keeping It in Sync

This export is a copy of the variables in
[`ghl-tracking-template.json`](../gtm-tags/ghl-tracking-template.json). If you
add, rename, or change a variable in the template, regenerate this file and
update the table above.
