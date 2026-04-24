# Weekly Update — Agent Instructions

This file is the cross-runtime entry point for the Weekly Update plugin. Runtimes that respect the `AGENTS.md` convention — **Codex CLI / Codex Desktop, Cursor, Cline, Continue, and any AGENTS.md-aware agent** — will pick this up automatically when the repo is added to their workspace / global agent path. Claude Code and Claude Desktop discover the same capabilities via the plugin manifest under `.claude-plugin/`.

The underlying logic lives in runtime-agnostic markdown files. This file just points the agent at them.

---

## What this plugin does

Consolidates the past 7 days of activity from the user's connected tools (Google Drive, Gmail, Google Calendar, and optionally Slack) into a structured weekly team update, delivered to a destination the user picks (Slack message, Gmail draft-to-self, or local markdown file). Intended to run on a Friday schedule unattended, but also triggerable by hand at any time.

The plugin rides on whatever connectors/tools the runtime already has. It does not bundle its own credentials, MCP servers, or secrets.

## Two skills, two triggers

### Setup — `skills/weekly-update-setup/SKILL.md`

Invoke when the user says anything like:
- "set up weekly update"
- "configure the weekly update"
- "install weekly update" (after the runtime has loaded this plugin)
- "onboard the weekly update plugin"
- "change weekly update sources" / "move weekly update to Monday" (edit mode)

Also invoke automatically the first time the `run` skill is triggered if `~/.weekly-update/config.json` does not exist.

**What it does:** detects connected tools, walks the user through picking sources + scope + draft destination + schedule, writes `~/.weekly-update/config.json`, and creates a recurring trigger if the runtime supports scheduling.

Full instructions: [`skills/weekly-update-setup/SKILL.md`](skills/weekly-update-setup/SKILL.md)

### Run — `skills/weekly-update-run/SKILL.md`

Invoke when the user says anything like:
- "run the weekly update"
- "generate this week's update"
- "draft my weekly update"
- "weekly status report"
- "Friday recap"
- any status-summary request over the last week, even without the word "weekly"

Also invoke when a scheduled trigger fires on Friday (if the runtime supports cron-style triggers).

**What it does:** reads the config, pulls the last 7 days from each configured source, consolidates using the template at `skills/weekly-update-run/references/update-template.md`, and delivers the draft to the configured destination.

Full instructions: [`skills/weekly-update-run/SKILL.md`](skills/weekly-update-run/SKILL.md)

## Configuration

Per-user config lives at `~/.weekly-update/config.json` (chmod 600). Schema and defaults are described in the setup skill. Never commit this file; it's ignored.

## Runtime-specific notes

- **Claude Code / Claude Desktop**: fully automatic. Install via `/plugin marketplace add SYMBaiEX/weekly-update` then `/plugin install weekly-update@weekly-update`. Setup creates a native Friday scheduled trigger.
- **Codex CLI / Codex Desktop**: place this repo under a path Codex scans for `AGENTS.md` (either workspace root or `~/.codex/`). Invoke by saying the trigger phrases above. Codex does not yet have a first-class cron, so schedule manually via `cron` or `launchd` pointing at a Codex one-shot invocation, or run by hand each Friday.
- **Cursor / Cline / Continue**: add the repo as a workspace and invoke by saying the trigger phrases. These runtimes generally don't have built-in scheduling — same workaround as Codex if you want autonomy.
- **Any agent with MCP + a shell**: read the two SKILL.md files and follow them directly. The instructions are runtime-agnostic.

## Scheduling without a native cron

For runtimes without built-in scheduling, the simplest pattern is a shell-level cron that invokes the runtime in one-shot mode with the prompt `run weekly update`. Example for Codex on macOS:

```cron
0 9 * * 5  /path/to/codex exec "run weekly update" >/dev/null 2>&1
```

The run skill handles everything from there — same code path as an interactive invocation.

## Do not

- Store tokens, API keys, or credentials anywhere in this repo or in the user's config file. The plugin exclusively uses whatever the runtime's connector layer provides.
- Send the draft to any destination other than the one configured in `~/.weekly-update/config.json`.
- @-mention anyone in the Slack draft — it's a private artifact for the user until they forward it.
- Invent or extrapolate bullets that aren't grounded in real artifacts pulled from the configured sources.
