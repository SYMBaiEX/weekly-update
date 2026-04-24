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

**Install in any one of your agent runtimes — setup auto-propagates the plugin to the others on your machine.** That means you can install via Claude Code, then switch to Codex or Cursor later and it just works. No re-cloning, no re-configuring.

### Easiest — via Claude Code

```
/plugin marketplace add SYMBaiEX/weekly-update
/plugin install weekly-update@weekly-update
set up weekly update
```

### Via any AGENTS.md-aware runtime (Codex, Cursor, Cline, Continue)

Clone once into the runtime's global scan path, then run setup. Setup will relocate the plugin to a canonical path (`~/.weekly-update/plugin/`) and symlink it into every other agent runtime on your machine, so this is a one-time action even if you use multiple runtimes.

```bash
# Example for Codex — adjust the target path for your runtime
git clone https://github.com/SYMBaiEX/weekly-update.git ~/.codex/weekly-update
```

Then in that runtime say `set up weekly update`.

### How auto-propagation works

The setup skill maintains one canonical copy at `~/.weekly-update/plugin/` and symlinks it into each installed runtime's scan path (`~/.codex/`, `~/.cursor/`, `~/.cline/`, `~/.continue/` — Claude Code/Desktop manage their own cache and are left alone). On each setup run, it `git pull`s the canonical copy so every runtime sees the latest version at once. Windows symlinks fall back to file copies; re-run setup to refresh after updates.

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

| Runtime | Skills | Autonomous Friday run |
|---|---|---|
| Claude Code | ✅ | ✅ via built-in scheduled triggers |
| Claude Desktop | ✅ | ❌ manual — say "run weekly update" Friday morning |
| Codex CLI / Desktop | ✅ via AGENTS.md | ❌ manual — or wire a shell cron (example in `AGENTS.md`) |
| Cursor / Cline / Continue | ✅ via AGENTS.md | ❌ manual |
| Any MCP-capable agent | ✅ — follow the SKILL.md files directly | depends on runtime |

## Customizing the template

Edit `skills/weekly-update-run/references/update-template.md`. The run skill reads this file each invocation — edits take effect immediately, no reload.

## Privacy

- No data is stored outside Claude's connectors and `~/.weekly-update/config.json` (identifiers + search queries only, no message bodies).
- The draft is delivered only to the destination you configured. The plugin never @-mentions anyone or posts elsewhere.

## Sharing with teammates

Teammates run the same two commands, answer the setup prompts, and it works — using their own connectors and their own permissions. No shared tokens, no service accounts, no workspace-admin coordination unless they choose to use the Slack source.
