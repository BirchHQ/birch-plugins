# birch-status for Claude Code

Reports the live status of each Claude Code session to the running [Birch](https://www.getbirchcode.dev/) desktop app and adds a `/birch` command for continuing the current session in Birch.

## `/birch`: continue this session in Birch

Run `/birch` in any Claude Code session to continue it in Birch. Birch comes to the foreground, selects the repository containing the current directory, and opens a terminal in that directory with the same Claude Code session resumed.

If Birch isn't running, the command starts it first.

Under the hood, `/birch` runs:

```bash
birch open --session-id "${CLAUDE_SESSION_ID}" --cwd "$(pwd)"
```

## Status reporting

Each hook invokes `birch agent-event`, which reads the hook's JSON payload from standard input and passes the event to Birch. Birch shows the resulting status on the workspace containing the session's `cwd`:

| Claude Code hook | `birch agent-event` | What Birch shows |
| --- | --- | --- |
| `SessionStart` | `--event started` | ready for your input |
| `UserPromptSubmit` | `--event working` | working |
| `PreToolUse` / `PostToolUse` | `--event working` | working |
| `Notification` | `--event waiting` | waiting for you, with a desktop notification |
| `SubagentStart` | `--event subagent-started` | nothing (sub-agent bookkeeping) |
| `SubagentStop` | `--event subagent-stopped` | nothing (sub-agent bookkeeping) |
| `Stop` | `--event idle` | done |
| `StopFailure` | `--event idle` | stopped with an error |
| `SessionEnd` | `--event ended` | the session is over |

- **Waiting.** Claude Code sends `Notification` for several reasons, with `notification_type` indicating which one. Birch ignores notifications that don't require your attention, such as an idle-prompt reminder or a background agent finishing.

- **Back to working.** `PreToolUse` and `PostToolUse` clear the waiting state. After you answer a permission prompt or question, Claude continues without a new user prompt, so the next tool call moves the workspace back to working.

- **Errors.** Claude Code fires `StopFailure` instead of `Stop` when a turn ends because of an API error, such as a rate limit, overload, authentication failure, or billing failure. Its payload contains `error` and `last_assistant_message`, including the error text shown by Claude. Birch marks the workspace red, shows the error, and adds an agent-error row to its Inbox. Versions of this plugin before 0.6.0 had no `StopFailure` hook, so API errors could leave the workspace showing as working.

- **Sub-agents.** While background sub-agents are running, the main agent may stop and Claude Code can fire `Stop` before the overall task is complete. Birch checks the `background_tasks` list included with each `Stop`: if sub-agents are still running, the workspace remains working; only a stop after the final background task finishes is treated as done. `SubagentStart` and `SubagentStop` maintain a fallback count for Claude Code versions whose `Stop` payload does not include this list. 

If Birch isn't running, `birch agent-event` exits silently in well under a second and never interrupts Claude. Every hook also specifies `"timeout": 5` seconds. Claude Code waits for hooks to exit before continuing, while its default hook timeout is ten minutes.

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

Birch normally installs and updates this plugin for you. It offers the plugin when you first use Claude Code, and **Settings -> Status integrations -> Claude Code** provides Install, Update, and Uninstall actions.

Birch writes the bundled marketplace to `agent-plugins/claude-code` in its data directory:

- macOS: `~/Library/Application Support/birch`
- Windows: `%APPDATA%\birch`

It then registers that directory with Claude Code as a local marketplace. This keeps the installed plugin aligned with the Birch version you are running and requires no network access. The installation applies to every Claude profile managed by Birch.

To install the plugin manually:

```bash
claude plugin marketplace add BirchHQ/birch-plugins
claude plugin install birch-status@birch
```

Or from inside Claude Code:

```text
/plugin marketplace add BirchHQ/birch-plugins
/plugin install birch-status@birch
```

The hooks apply to Claude Code sessions started after installation, across all projects.

## Uninstall

```bash
claude plugin uninstall birch-status@birch
```

## Usage statusline (optional)

Birch can also show live per-session usage information — including the model, context fill, and five-hour and weekly rate-limit windows — in the workspace properties panel.

This data is only available through Claude Code's status line payload, so it requires the user-level `statusLine` setting, which a plugin cannot configure:

```json
{
  "statusLine": {
    "type": "command",
    "command": "birch statusline",
    "refreshInterval": 60
  }
}
```

Enable it from **Settings**. Birch never overwrites an existing status line configuration.

You can also add the setting manually to `~/.claude/settings.json`.

The `refreshInterval` keeps idle sessions reporting periodically, allowing Birch to distinguish a live but idle session from one that has ended.

If you already use another status line command, keep it and pass the same standard input to `birch statusline` as well. For example, your command can start with:

```bash
tee >(birch statusline >/dev/null)
```

`birch statusline` prints a compact status string for Claude Code, for example:

```text
Opus 5.5 · ctx 71% · 5h 34%
```

It also forwards the status payload to a running Birch instance. If Birch isn't running, the command exits silently with status code 0.