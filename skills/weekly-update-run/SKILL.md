---
name: weekly-update-run
description: Consolidates the past 7 days of activity from the user's configured Claude connector sources (Google Drive project-plan folders, Gmail saved searches, Google Calendar meetings, Slack channels if connected) into a structured weekly team update and posts it to the configured draft destination (Slack channel, Gmail draft to self, or local markdown file) for the user to review and send. Use this skill when a scheduled Friday trigger fires, or when the user says "run the weekly update", "generate this week's update", "draft my weekly update", "weekly status report", "Friday recap", or any close variant. Also use it when the user asks for a status summary over the last week even if they don't say the word "weekly". Do not use this skill before the user has run weekly-update-setup — check for ~/.weekly-update/config.json first, and if it's missing, defer to weekly-update-setup.
---

# Weekly Update — Run

Consolidate the last 7 days into a drafted update, delivered to the configured destination. Designed to run unattended on a Friday schedule — assume no human is watching. Fail loud (visible error in the destination) rather than silently.

## Preflight

1. Read `~/.weekly-update/config.json`. If missing or `version` < 2, stop and tell the user to run the setup skill. If the destination is Slack, also DM the user a message; if Gmail, save an error note as a draft; if file, write it to the drafts dir.
2. Resolve the reporting window: `end = now`, `start = now - 7 days`, in the config's timezone.
3. For each source, verify the required connector is still enabled. If not, log the failure into an `errors[]` array and skip that source — don't abort the whole run. The final draft will include a footer listing what was skipped and why.

## Pull from each source (run in parallel where possible)

### Google Drive

For each folder in `config.sources.drive`:
- List files modified within the window.
- For each modified doc, read the first ~2000 tokens plus any section headings containing "status", "update", "this week", or "blockers".
- Capture title + URL + the owner if it's not the user.

If no folders were configured and the user picked the "just my modified docs" default, scan all docs in `My Drive` modified in the window (cap at ~20).

### Gmail

For each saved query in `config.sources.gmail`:
- Execute the query. Pull subject, from, date, thread ID, and the first ~500 tokens of the most recent message body.
- De-duplicate by thread — one entry per thread.
- Capture the thread permalink.

### Google Calendar

If `config.sources.calendar` is present:

- **include_past = true**: list events on the selected calendars within the past 7 days. Filter out declined events and events where the user was just an attendee on a large (10+) meeting with no notable outcome signal. Keep: 1:1s, smaller meetings, anything marked important, and any event where the user was the organizer. Capture title, attendees (if <8), and any notes/description that look like decisions or action items.
- **include_future = true**: list events in the next 7 days. Capture title + attendees. These surface as commitments / upcoming work, not outcomes.

### Slack

Only if `config.sources.slack` exists and the Slack connector is active. For each channel:
- Fetch messages in the window.
- Keep: messages with 3+ reactions, messages that start threads with 5+ replies, messages from the user or their direct reports, messages containing decision/blocker keywords (`decided`, `blocker`, `shipped`, `launched`, `postponed`, `at risk`, `p0`, `p1`, `rolled back`).
- Drop: bot noise, pure emoji replies, reactions without content.
- Capture permalinks for the top ~5 items per channel.

## Consolidate

Synthesize pulled material into the template at `references/update-template.md`. Rules:

- **Grounded bullets only.** Every bullet must trace to a real artifact from the pulled material. Do not invent, infer, or extrapolate beyond the data.
- **Prefer the user's own words** where they exist in the source material.
- **One line per bullet** where possible. Trim ruthlessly — if a section exceeds 5 bullets, pick the top 5 by impact.
- **Link in context**. When a bullet cites a specific artifact, append a compact link:
  - Slack: `<url|[thread]>`
  - Drive: `<url|[doc]>`
  - Gmail: `<url|[email]>`
  - Calendar: `<url|[meeting]>`
- **Include a "Meetings" section** only if Calendar was a source — skip the header entirely otherwise. Same for "Customer signal" (Gmail), "Channel activity" (Slack), etc. Missing sources are dropped, not padded with "N/A."
- **Empty but-relevant sections say so** — if Blockers was promised (because a source was configured) but nothing landed, write `_Nothing this week_` under the heading. Readers need to distinguish "there were no blockers" from "the blockers section was dropped because no source was configured for it."

## Deliver to the configured destination

Branch on `config.draft.type`:

### `slack`

Post the final markdown to `config.draft.id`. Prefix first line:
```
:memo: *Weekly update draft — {{start_date}} → {{end_date}}*
_Review, edit, and forward when ready. Reply "regenerate" to run again._
```
Do not post to any other channel. Do not @-mention anyone.

### `gmail_draft`

Create an unsent draft in the user's Gmail, `To:` themselves:
- Subject: `Weekly update — {{start_date}} → {{end_date}}`
- Body: the final markdown rendered as HTML (convert headings to `<h3>`, bullets to `<ul><li>`, preserve links). Include a top banner: "This is a draft. Review and forward when ready."

Do not send. Leave it as a draft so the user controls delivery.

### `file`

Write to `{{config.draft.dir}}/{{YYYY-MM-DD}}.md`. Create the dir if missing. Overwrite any existing file for the same date (idempotent re-runs). After writing, print the absolute path so the user can open it.

## On error

If the run fails after preflight, write a short error report to the same destination (Slack message / Gmail draft / markdown file) with:
- the failing step
- the error message
- what the user can do (e.g. "reconnect Gmail in Settings → Connectors")

Silent failures are worse than noisy ones for an autonomous job.
