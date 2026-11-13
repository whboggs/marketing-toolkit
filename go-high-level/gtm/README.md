# GoHighLevel – Google Tag Manager

GTM tags and variables for tracking GoHighLevel forms, calendars, and chat
widgets embedded on a client website.

---

## Folders

### [`gtm-tags/`](gtm-tags/)

- **[GHL Tracking Template](gtm-tags/ghl-tracking-template.json)** — complete
  GTM container export with every tag, trigger, and variable needed for a full
  GHL setup. Anonymized IDs.
- **[Form Submit Bridge](gtm-tags/form-submit-bridge/)** — listens for
  GoHighLevel iframe form submits and pushes `ghl_form_submit` to the
  dataLayer.
- **[Chat Widget Bridge](gtm-tags/chat-widget-bridge/)** — pushes GHL chat
  widget open and lead-submit events (`ghl_chat_open`, `ghl_chat_submit`) to
  the dataLayer.
- **[Meta Event – Lead](gtm-tags/meta/meta-event-lead/)** — Meta Pixel Lead
  tag for GHL submissions.

### [`gtm-variables/`](gtm-variables/)

- **[GHL GTM Variables](gtm-variables/)** — container export with only the
  variables from the tracking template (no tags or triggers), plus a table of
  every variable and its configuration.

---

## Which file to import

| You need | Import |
|---|---|
| A full GHL setup in a new or empty container | `gtm-tags/ghl-tracking-template.json` |
| Just the GHL variables (your own tags, or repairing missing variables) | `gtm-variables/ghl-gtm-variables.json` |
