# birch-status for Codex

Reports the live status of a Codex session to the matching workspace in a running
[Birch](https://www.getbirchcode.dev/) desktop app. The plugin uses Codex's native lifecycle hooks;
nothing polls Codex or watches its processes.

## Status reporting

Every hook runs `birch agent-event`, which reads the hook's JSON payload from standard input, finds the
workspace that contains the session's `cwd`, and relays the event to Birch:

| Codex hook | `birch agent-event` | What Birch shows |
| --- | --- | --- |
| `SessionStart` | `--event started` | ready for your input |
| `UserPromptSubmit` | `--event working` | working |
| `PermissionRequest` | `--event waiting` | waiting for a permission decision |
| `PreToolUse` matching `^request_user_input$` | `--event waiting` | waiting for your answer |
| `PostToolUse` | `--event working` | working |
| `Stop` | `--event idle` | done |
| `SessionEnd` | `--event ended` | the session is over |
| `SubagentStart` | `--event subagent-started` | nothing (counts running sub-agents) |
| `SubagentStop` | `--event subagent-stopped` | nothing (counts running sub-agents) |

`PermissionRequest` and `request_user_input` are separate, explicit waiting signals; the `PostToolUse`
that follows your answer switches the workspace back to working. The sub-agent events let Birch avoid
showing a finished turn while background agents are still running.

Every command passes `--agent codex`, so Birch knows which agent reported without relying on
Codex-specific payload fields. Each hook has a three-second timeout. If Birch isn't running, the command
exits silently and does not interrupt Codex. It talks to Birch over a local named pipe only.

## Requirements

The `birch` command-line tool that ships with Birch must be on your `PATH`. Birch can set that up:
**Settings → Status integrations → Install birch CLI to PATH**. Check with:

```bash
birch --help
```

## Install

Birch installs and updates this plugin itself: **Settings → Status integrations → Codex** has Install,
Update and Uninstall, and Birch offers the plugin the first time you start a Codex workspace. Birch
writes this marketplace from its own bundle to `agent-plugins/codex` in its data directory
(`~/Library/Application Support/birch` on macOS, `%APPDATA%\birch` on Windows) and registers that
folder with Codex as a local marketplace, so the plugin always matches the Birch version you run and
installing needs no network.

To install it by hand instead:

```bash
codex plugin marketplace add BirchHQ/birch-plugins
codex plugin add birch-status@birch
```

Then start a new Codex session so the hooks load, open `/hooks`, review the `birch-status` commands and
trust them. Codex does not run a plugin's hooks until you approve them, and it asks again whenever a
hook's command changes.

## Uninstall

```bash
codex plugin remove birch-status@birch
```
