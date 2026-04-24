---
name: weekly-update-run
description: Consolidates the past 7 days of activity from the user's configured sources (Slack channels, Google Drive project-plan folders, Gmail saved searches) into a structured weekly team update and posts it as a draft in the configured Slack channel for the user to review and send. Use this skill when a scheduled Friday trigger fires, when the user says "run the weekly update", "generate this week's update", "draft my weekly update", "weekly status report", "Friday recap", or any close variant. Also use it when the user asks for a status summary over the last week even if they don't say the word "weekly". Do not use this skill before the user has run weekly-update-setup — check for ~/.weekly-update/config.json first, and if it's missing, defer to weekly-update-setup.
---

# Weekly Update — Run

Consolidate the last 7 days of activity into a posted draft. This skill is designed to run unattended on a Friday schedule — assume no human is watching. Fail loud by posting a visible error in the draft channel rather than silently.

## Preflight

1. Read `~/.weekly-update/config.json`. If missing or `version` != 1, stop and post a Slack message to the user's DM saying "Weekly update not configured — run the setup skill to initialize."
2. Resolve the reporting window: `end = now`, `start = now - 7 days`, in the config's timezone.
3. Verify each configured connector is still authenticated. If any is not, post an error draft listing which connectors failed and skip that source — do not abort the whole run.

## Pull from each source

Pull these in parallel where possible.

### Slack

For each channel in `config.sources.slack`:
- Fetch messages in the window.
- Keep: messages with 3+ reactions, messages that start threads with 5+ replies, messages from the user or their direct reports, messages containing keywords that signal decisions or blockers (`decided`, `blocker`, `shipped`, `launched`, `postponed`, `at risk`, `p0`, `p1`).
- Drop: bot noise, reactions without content, pure emoji replies.
- Capture permalinks for the top ~5 items per channel so the final draft can cite them.

### Google Drive

For each folder in `config.sources.drive`:
- List files modified within the window.
- For each modified doc, read the first ~2000 tokens plus any section headings that contain the word "status", "update", "this week", or "blockers".
- Capture the doc title and URL.

### Gmail

For each saved query in `config.sources.gmail`:
- Execute the query. Pull subject, from, date, and the first ~500 tokens of the body.
- De-dup threads — one entry per thread, use the most recent message.
- Capture the thread permalink.

## Consolidate

Synthesize pulled material into the template in `references/update-template.md`. Do not invent content — every bullet must be grounded in a real source. Prefer the user's own words where they exist. Keep each bullet to one line where possible.

When a bullet is grounded in a specific artifact (Slack thread, Drive doc, email), include a compact Slack-flavored link at the end of the bullet so the reader can click through. Example: `• Closed the pricing redesign decision <https://...|[thread]>`

If a section has no content for the week, write `_Nothing this week_` rather than omitting the heading — readers should be able to tell "there were no blockers" apart from "the blockers section was dropped."

## Post the draft

Post the final markdown to the Slack channel in `config.draft.channel_id`. Prefix the first line with `:memo: *Weekly update draft — <date range>*` and add a second line of plain text: `_Review, edit, and forward when ready. Reply in thread with "regenerate" to run this again._`

Do not post to any other channel. Do not @-mention anyone. The draft is a private artifact for the user until they choose to send it.

## On error

If the run fails after the preflight passes, post a short error message to the draft channel with: the failing step, the error, and what the user can do (e.g. "reconnect Gmail in Settings → Connectors"). Silent failures are worse than noisy ones for an autonomous job.
