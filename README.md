# Weekly Update

Autonomously drafts your team's Friday status update. Every Friday morning, it pulls the last 7 days from your connected Claude connectors — **Google Drive**, **Gmail**, **Google Calendar**, and **Slack** (if connected) — consolidates it into a structured update, and drops it where you want to review it: a Slack channel, a Gmail draft to yourself, or a local markdown file.

## Zero-config MCP

This plugin rides entirely on **Claude's native connectors**. There is no bundled MCP server, no token paste, no Slack app to register. Whatever connectors you have enabled in **Settings → Connectors** is what the plugin can use. Connect a new one later and re-run setup — the plugin will pick it up.

## What you get

- A **setup** skill that detects your connectors, asks what to pull from, and creates a Friday scheduled trigger.
- A **run** skill that consolidates the week and delivers the draft.
- A **Friday 9am scheduled trigger** (Claude Code) that fires the run skill automatically.
- Auto-update is enabled during setup, so you'll always be on the latest version without lifting a finger.

## Prerequisites

At least one of these connectors enabled in **Claude → Settings → Connectors**:

| Connector | Role |
|---|---|
| **Google Drive** | Source — project plans, weekly docs |
| **Gmail** | Source — customer signal, external mail. Also acts as draft destination (draft-to-self). |
| **Google Calendar** | Source — meetings held + commitments for next week |
| **Slack** | Optional source — channel activity. Also preferred draft destination if connected. |

No credentials live in the plugin. No secrets live on disk except whatever Claude's connector layer already stores.

## Install

```
/plugin marketplace add SYMBaiEX/weekly-update
/plugin install weekly-update@weekly-update
```

## Use

**First time:**

```
set up weekly update
```

Interactive setup detects your connectors, asks which to use, picks a draft destination, and creates the Friday trigger. Writes `~/.weekly-update/config.json`.

**Automatic (Claude Code):** fires every Friday at the time you picked. Nothing to do.

**Manual (any surface):**

```
run weekly update
```

**Change settings:**

```
change weekly update sources
```

or

```
move weekly update to Monday
```

## Draft destinations

The setup skill picks the best option based on what you have connected:

1. **Slack channel** — if Slack is connected. Usually a private channel or DM-to-self.
2. **Gmail draft to yourself** — unsent draft in your inbox. Open Friday morning, review, forward.
3. **Local markdown file** — `~/.weekly-update/drafts/YYYY-MM-DD.md`. Fallback, always works.

You can override the default during setup.

## Where things live

- `~/.weekly-update/config.json` — your personal config (chmod 600). Survives plugin updates.
- `~/.weekly-update/drafts/` — local drafts, if you chose the file destination.

## Platforms

- **Claude Code** — full support including autonomous Friday scheduling.
- **Claude Desktop** — skills work; scheduling is manual (Desktop has no cron yet). Just say "run weekly update" Friday morning.

## Customizing the template

Edit `skills/weekly-update-run/references/update-template.md`. The run skill reads this file each invocation — edits take effect immediately, no reload.

## Privacy

- No data is stored outside Claude's connectors and `~/.weekly-update/config.json` (identifiers + search queries only, no message bodies).
- The draft is delivered only to the destination you configured. The plugin never @-mentions anyone or posts elsewhere.

## Sharing with teammates

Teammates run the same two commands, answer the setup prompts, and it works — using their own connectors and their own permissions. No shared tokens, no service accounts, no workspace-admin coordination unless they choose to use the Slack source.
