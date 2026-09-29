# birch-status for Codex

Reports the live status of each Codex session to the running [Birch](https://www.getbirchcode.dev/) desktop app.

The plugin uses Codex's native lifecycle hooks; nothing polls Codex or watches its processes.

## Status reporting

Each hook invokes `birch agent-event`, which reads the hook's JSON payload from standard input and passes the event to Birch. Birch shows the resulting status on the workspace containing the session's `cwd`:

| Codex hook | `birch agent-event` | What Birch shows |
| --- | --- | --- |
| `SessionStart` | `--event started` | ready for your input |
| `UserPromptSubmit` | `--event working` | working |
| `PermissionRequest` | `--event waiting` | waiting for a permission decision |
| `PreToolUse` matching `^request_user_input$` | `--event waiting` | waiting for your answer |
| `PostToolUse` | `--event working` | working |
| `Stop` | `--event idle` | done |
| `SessionEnd` | `--event ended` | the session is over |
| `SubagentStart` | `--event subagent-started` | nothing (sub-agent bookkeeping) |
| `SubagentStop` | `--event subagent-stopped` | nothing (sub-agent bookkeeping) |

- **Waiting.** `PermissionRequest` and `request_user_input` are separate, explicit signals that Codex is waiting for you. The `PostToolUse` that follows your answer moves the workspace back to working.

- **Sub-agents.** `SubagentStart` and `SubagentStop` let Birch track running background agents, so it does not show a turn as done while sub-agents are still working.

Every hook passes `--agent codex`, so Birch knows which agent reported the event without relying on Codex-specific payload fields.

If Birch isn't running, `birch agent-event` exits silently and never interrupts Codex. Every hook also specifies a three-second timeout.

The command communicates with Birch only through a local named pipe. It never opens Birch's database and makes no network requests.

## Requirements

The `birch` command-line tool that ships with Birch must be on your `PATH`. Birch can set this up from:

**Settings -> Status integrations -> Install birch CLI to PATH**

On macOS, Birch creates a symlink in `/usr/local/bin`. On Windows, it adds the CLI directory to your user `PATH`.

Check the installation with:

```bash
birch --help
```

## Install

Birch normally installs and updates this plugin for you. It offers the plugin when you first use Codex, and **Settings -> Status integrations -> Codex** provides Install, Update, and Uninstall actions.

Birch writes the bundled marketplace to `agent-plugins/codex` in its data directory:

- macOS: `~/Library/Application Support/birch`
- Windows: `%APPDATA%\birch`

It then registers that directory with Codex as a local marketplace. This keeps the installed plugin aligned with the Birch version you are running and requires no network access.

To install the plugin manually:

```bash
codex plugin marketplace add BirchHQ/birch-plugins
codex plugin add birch-status@birch
```

Then start a new Codex session so the hooks load. Open `/hooks`, review the `birch-status` commands, and trust them.

Codex does not run a plugin's hooks until you approve them, and asks again whenever a hook command changes.

## Uninstall

```bash
codex plugin remove birch-status@birch
```