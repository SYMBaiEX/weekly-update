# Weekly Update

Autonomously drafts your team's Friday status update. Pulls the last 7 days from **Slack**, **Google Drive**, and **Gmail**, consolidates it into a structured update, and drops it as a draft in a Slack channel you choose — ready for you to review and forward.

## What you get

- A **setup** skill that walks you through picking sources (Slack channels, Drive folders, Gmail queries) and the destination Slack channel.
- A **run** skill that consolidates the week and posts the draft.
- A **Friday scheduled trigger** (Claude Code) that fires the run skill automatically. No secrets stored — the plugin uses whichever Slack / Google Drive / Gmail connectors you already have in Claude.

## Prerequisites

Before running setup, connect these in **Claude → Settings → Connectors**:

- **Google Drive** (only if you want Drive as a source)
- **Gmail** (only if you want email as a source)

Slack is handled by the plugin's bundled MCP server (`@modelcontextprotocol/server-slack`). The setup skill walks you through a **~90-second one-time Slack app install** — it copies the app manifest to your clipboard, opens the Slack app creation page, and prompts you to paste your resulting User OAuth Token. The token is stored locally in `~/.weekly-update/config.json` (chmod 600). Your token inherits your own Slack permissions, so the plugin automatically has access to every channel you can see — nothing extra to invite.

The plugin itself contains **no credentials, tokens, or secrets**. Each teammate creates their own Slack token and connects their own Google/Gmail — which is why this plugin is safe to share internally and publish publicly.

## Install

In Claude Code:

```
/plugin install weekly-update
```

(or drop the `.plugin` file into Claude Desktop to install there.)

## Use

**First time:**

```
set up weekly update
```

This launches an interactive setup that writes `~/.weekly-update/config.json` and creates a Friday 9am scheduled trigger.

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

## Where things live

- `~/.weekly-update/config.json` — your personal config (sources, draft channel, schedule). Survives plugin updates.
- Plugin files — read-only reference; safe to reinstall or upgrade without losing config.

## Platforms

- **Claude Code** — full support including autonomous Friday scheduling.
- **Claude Desktop** — skills work; scheduling is manual (Desktop has no cron yet). Just say "run weekly update" Friday morning.
- **ChatGPT Desktop** — not targeted in v1.

## Customizing the template

Edit `skills/weekly-update-run/references/update-template.md`. The run skill reads this file each time, so template edits take effect immediately.

## Privacy

- No data is stored outside your Claude connectors and the local `~/.weekly-update/config.json` (which only contains identifiers and your search queries — no message bodies).
- The draft is posted only to the channel you configured. The plugin never @-mentions anyone or posts to other channels.

## Sharing with teammates

Because there are no secrets and the plugin relies on each user's own connectors, teammates can install this and immediately use it with their own Slack / Drive / Gmail. No shared tokens, no service accounts.
