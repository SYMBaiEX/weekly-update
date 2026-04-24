---
name: weekly-update-setup
description: Interactive first-run setup for the weekly team update consolidator. Walks the user through picking which sources to pull from (Slack channels, Google Drive folders, Gmail labels/queries), choosing the destination Slack channel to drop the draft in, confirming the Friday schedule, and creating the scheduled trigger. Use this skill when the user says things like "set up weekly update", "configure weekly update", "install weekly update", "onboard the weekly update plugin", or whenever the plugin is newly installed and no config exists at ~/.weekly-update/config.json yet. Also re-run this when the user wants to change sources or the draft channel.
---

# Weekly Update — Setup

This skill performs one-time configuration for the weekly update consolidator. The resulting config lives at `~/.weekly-update/config.json` so it survives plugin updates.

## Why this lives in its own skill

The consolidator itself runs headless on a Friday schedule — there is no human in the loop at run time. All choices that require human judgment (which Slack channels matter, which Drive folder has the current project plans, which Gmail query surfaces customer email, where to drop the draft) must be captured up front. Treat this skill as the "installer."

## Flow

Perform these steps in order. Use `AskUserQuestion` when available; otherwise ask in chat. Do not skip the confirmation at the end.

### 1. Check for existing config

Read `~/.weekly-update/config.json`. If it exists, show the current settings and ask whether the user wants to *edit* (change specific fields), *replace* (start fresh), or *cancel*. If editing, only re-ask the fields they want to change.

### 2. Verify connectors

The plugin relies on the user's existing Claude connectors — it does NOT ship its own MCP credentials. Before configuring sources, confirm with the user that these connectors are connected in their Claude settings:

- **Slack** — required (both as a source and as the draft destination)
- **Google Drive** — required only if the user wants Drive as a source
- **Gmail** — required only if the user wants email as a source

If a required connector is missing, pause setup and point the user to Settings → Connectors. Do not proceed with a source the user cannot authenticate.

### 3. Pick sources

Ask the user which of the following sources to include (multi-select — at least one required):

- **Slack** — channels to scan for wins, blockers, and notable threads
- **Google Drive** — folders containing project plans or weekly docs
- **Gmail** — saved searches or labels that indicate customer signal / external commitments

For each selected source, collect the specific scope:

- **Slack sources**: ask for channel names. Resolve each to a channel ID by calling the Slack connector's channel-list tool so we store stable IDs, not names. Store both `{ id, name }` so the draft is readable.
- **Google Drive sources**: ask for folder names or URLs. For URLs, extract the folder ID. For names, search Drive and let the user disambiguate if multiple match. Store `{ id, name }`.
- **Gmail sources**: ask for one or more Gmail search queries (e.g. `label:customer-signal newer_than:7d`, `from:@bigcustomer.com newer_than:7d`). Store the raw query strings — they are evaluated at run time with a sliding 7-day window.

### 4. Pick the draft destination

Ask which Slack channel the weekly draft should be posted to. This is almost always a private channel the user owns (e.g. `#me-drafts`, DM to self, or a team lead channel). Confirm explicitly — the consolidator runs unattended, so a wrong channel means the draft goes to the wrong audience.

Store the destination as `{ id, name }`.

### 5. Pick the schedule

Default: **Friday 09:00 in the user's local timezone**. Ask whether to keep the default or change the day/time. Store as a cron expression and a human-readable label.

### 6. Preview and confirm

Before writing the config, print a summary:

```
Weekly update will run:        Fridays at 9:00 AM America/Chicago
Sources:
  Slack:         #product, #eng-standup, #customers
  Google Drive:  "2026 Project Plans" folder
  Gmail:         label:customer-signal newer_than:7d
Draft posted to:               #my-drafts (DM)
```

Ask for explicit "yes, create it" confirmation.

### 7. Write config

Create `~/.weekly-update/` (700 perms) and write `config.json`:

```json
{
  "version": 1,
  "created_at": "ISO-8601 timestamp",
  "sources": {
    "slack": [{ "id": "C0123", "name": "product" }],
    "drive": [{ "id": "1abc...", "name": "2026 Project Plans" }],
    "gmail": ["label:customer-signal newer_than:7d"]
  },
  "draft": {
    "channel_id": "D0456",
    "channel_name": "my-drafts"
  },
  "schedule": {
    "cron": "0 9 * * 5",
    "timezone": "America/Chicago",
    "label": "Fridays at 9:00 AM CT"
  }
}
```

Do not log any tokens, channel contents, or email bodies during setup — only store identifiers and user-supplied queries.

### 8. Create the scheduled trigger

On **Claude Code**, create a scheduled trigger via the `schedule` skill / CronCreate tool. The trigger's prompt should be exactly:

```
/weekly-update-run
```

Use the cron and timezone from the config. Save the returned trigger ID into `config.json` under `schedule.trigger_id` so a later re-run of setup can update it in place instead of creating duplicates.

On **Claude Desktop** (no native scheduler), skip this step and instead tell the user: "Claude Desktop doesn't support scheduled triggers yet, so on Friday mornings just say `run weekly update` and the plugin will do the rest." This is documented behavior, not a bug — surface it clearly.

### 9. Offer a dry run

Ask: "Want me to run it once right now to sanity-check the output?" If yes, invoke the `weekly-update-run` skill immediately with a flag indicating dry-run (it posts to the draft channel the same way, so the user can see the real output in its real destination).

## Editing later

If the user ever says "change the weekly update sources" / "add a channel to the weekly update" / "move the weekly update to Monday", re-enter this skill in edit mode (step 1), preserve unchanged fields, and re-create the scheduled trigger if the schedule changed.
