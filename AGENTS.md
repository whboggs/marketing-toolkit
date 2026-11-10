# Meta Lead tags live in two places — keep them in sync

Each `fbq('track', 'Lead', …)` snippet exists in BOTH its own folder and the
GTM Starter Kit (`gtm-essentials/gtm-starter-kit/gtm-starter-kit.json`, the
tag's `html` parameter, JSON-escaped):

| Snippet | Folder file | Starter Kit tag |
|---|---|---|
| Basic | `meta/pixel/lead/basic/chtml-lead-config.html` | **Meta - Event - Lead - Form Submit** |
| Advanced Matching | `meta/pixel/lead/advanced-matching/chtml-lead-advanced-matching.html` | **Meta - Event - Lead - Form Submit (Advanced Matching)** |

Each pair must always match. Whenever one is updated, apply the same change to
the other in the same commit — and update both folders' READMEs to match.
