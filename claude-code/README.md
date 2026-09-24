# birch-status for Claude Code

Reports the live status of each Claude Code session to a running [Birch](https://www.getbirchcode.dev/)
desktop app, and adds a `/birch` command that continues the current session inside Birch.

## `/birch`: continue this session in Birch

Run `/birch` in any Claude Code session to hand it over to Birch. Birch comes to the foreground, selects
the repository this directory belongs to, and opens a terminal **in this directory** that resumes this
exact conversation with `claude --resume`. If Birch isn't running, the command starts it first. The
command runs `birch open --session-id "${CLAUDE_SESSION_ID}" --cwd "$(pwd)"`.

## Status reporting

Every hook runs `birch agent-event`, which reads the hook's JSON payload from standard input and relays
it to Birch. Birch shows the result on the workspace the session's `cwd` belongs to:

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

- **Waiting.** Claude Code sends `Notification` for many reasons, and the payload's `notification_type`
  says which. Birch ignores the ones that don't need you, such as the reminder that fires after the
  prompt has been idle for a while, or a background agent finishing.
- **Back to working.** `PreToolUse` and `PostToolUse` clear the waiting state: after you answer a
  permission prompt or a question, Claude continues without a new prompt, so the next tool call is
  what flips the workspace back to working.
- **Errors.** Claude Code fires `StopFailure` *instead of* `Stop` when a turn ends on an API error (a
  rate limit, an overload, an authentication or billing failure). Its payload carries `error` and
  `last_assistant_message`, the error text Claude showed you; Birch marks the workspace red, shows the
  text, and adds an agent-error row to its Inbox. Versions of this plugin before 0.6.0 have no
  `StopFailure` hook, so an API error left the workspace spinning.
- **Sub-agents.** While background sub-agents run, the main agent stops and Claude Code fires `Stop`
  mid-task. Birch reads the `background_tasks` list that Claude Code sends with every `Stop`: a stop
  with sub-agents still running shows as working, and only the stop after the last one finishes counts
  as done. `SubagentStart` / `SubagentStop` keep a fallback count for Claude Code versions whose `Stop`
  payload has no such list. (Versions before 0.5.0 matched the `Task` tool by name, which broke when it
  was renamed `Agent`; the sub-agent events don't depend on a tool name.)

The session's `cwd` is matched to the workspace it lives in, so the right workspace lights up whichever
folder Claude runs in, including sessions started outside Birch. Events from a `cwd` that no Birch
workspace contains are ignored.

If Birch isn't running, `birch agent-event` exits silently in well under a second and never interrupts
Claude. Every hook also carries `"timeout": 5` (seconds): Claude Code waits for a hook to exit before it
continues, and its default hook timeout is ten minutes. The command talks to Birch over a local named
pipe only; it never opens Birch's database.

## Requirements

The `birch` command-line tool that ships with Birch must be on your `PATH`. Birch can set that up:
**Settings → Status integrations → Install birch CLI to PATH** (a symlink in `/usr/local/bin` on
macOS; the CLI's folder is added to your user `PATH` on Windows). Check with:

```bash
birch --help
```

## Install

Birch installs and updates this plugin itself. It offers the plugin on first launch, and
**Settings → Status integrations → Claude Code** has Install, Update and Uninstall. Birch writes this
marketplace from its own bundle to `agent-plugins/claude-code` in its data directory
(`~/Library/Application Support/birch` on macOS, `%APPDATA%\birch` on Windows) and registers that
folder with Claude Code as a local marketplace, so the plugin always matches the Birch version you run
and installing needs no network. It covers every Claude profile Birch manages.

To install it by hand instead:

```bash
claude plugin marketplace add BirchHQ/birch-plugins
claude plugin install birch-status@birch
```

Or, inside Claude Code: `/plugin marketplace add BirchHQ/birch-plugins`, then
`/plugin install birch-status@birch`. The hooks apply to Claude Code sessions started afterwards, in
every project.

## Uninstall

```bash
claude plugin uninstall birch-status@birch
```

## Usage statusline (optional)

Birch can also show live per-session usage (model, context fill, and the five-hour and weekly rate-limit
windows) in its workspace properties panel. That data is only available through Claude Code's status
line payload, so it needs the user-level `statusLine` setting, which a plugin cannot provide:

```json
{
  "statusLine": { "type": "command", "command": "birch statusline", "refreshInterval": 60 }
}
```

Enable it from Birch (**Settings → Status integrations → Claude Code → Enable usage statusline**; Birch
never overwrites a status line you already have) or add the snippet to `~/.claude/settings.json`
yourself. The `refreshInterval` keeps an idle session reporting, so Birch can tell a live session from
one that has ended. If you already have a status line command, keep it and also send the same standard
input to `birch statusline`, for example by starting your command with `tee >(birch statusline >/dev/null)`.

`birch statusline` prints a compact status text (`Opus 4.8 · ctx 71% · 5h 34%`) for Claude Code and
forwards the payload to a running Birch. It is silent and exits with 0 when Birch isn't running.
