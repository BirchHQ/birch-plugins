# Birch plugins

Agent plugins for [Birch](https://www.getbirchcode.dev/), the desktop git client for working with
coding agents. They let Birch show, on each workspace, what the agent running there is doing: ready,
working, waiting for you, done, or stopped with an error.

| Plugin | Agent | Directory | Marketplace manifest |
| --- | --- | --- | --- |
| `birch-status` | [Claude Code](https://code.claude.com/) | [`claude-code/`](claude-code/) | [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) |
| `birch-status` | [Codex](https://developers.openai.com/codex/) | [`codex/`](codex/) | [`.agents/plugins/marketplace.json`](.agents/plugins/marketplace.json) |

Both marketplaces are named `birch`, so the plugin id is `birch-status@birch` in either agent.

## What the plugins do

Each plugin registers hooks for the agent's lifecycle events (session start, prompt submitted, tool
use, waiting for permission or input, turn finished, session end, sub-agents starting and stopping).
Every hook runs the `birch` command-line tool that ships with Birch, which passes the event to the Birch
app. The Claude Code plugin also adds a `/birch` command that continues the current session in a Birch
terminal. The per-plugin READMEs list every hook and what Birch shows for it.

**Nothing leaves your machine.** The hooks hand the agent's hook payload (the session id, working
directory and event details) to a Birch app running on the same computer, over a local named pipe. They
make no network requests, and when Birch isn't running they exit silently within a second.

The `birch` command must be on your `PATH`; Birch can install it from **Settings → Status
integrations**.

## Installing

**You normally don't need to.** Birch installs and updates these plugins itself from **Settings →
Status integrations**, and offers them when you first use each agent. Birch carries a pinned copy of
this repository, writes it to its data directory, and registers that copy with each agent as a local
marketplace. The installed plugin therefore always matches the Birch version you run, and installing
needs no network access.

To install a plugin by hand instead:

```bash
# Claude Code
claude plugin marketplace add BirchHQ/birch-plugins
claude plugin install birch-status@birch

# Codex
codex plugin marketplace add BirchHQ/birch-plugins
codex plugin add birch-status@birch
```

Start a new agent session afterwards. Codex also asks you to review and trust the new hooks: open
`/hooks` in the new session.

## How Birch uses this repository

This repository is the single source of the plugins. Birch's own repository includes it as a git
submodule pinned to a commit and embeds these files in the app, so a change made here reaches Birch
users with the Birch release that moves the submodule to it.

When you change a plugin:

- Bump `version` in its manifest (`claude-code/.claude-plugin/plugin.json` or
  `codex/.codex-plugin/plugin.json`). Both agents cache installed plugins by version, and Birch offers
  the update when the version it ships is newer than the installed one.
- Treat `hooks.json` changes with care. Codex trusts each hook entry by a hash of its definition, so
  changing an entry asks every user to review and trust it again.
- Hooks call a bare `birch`, so a newer plugin can meet an older `birch` command. The command ignores
  options it doesn't recognize, but it rejects an `--event` value it doesn't know, so a new event kind
  needs a Birch release that understands it first.
- Keep files in LF line endings (`.gitattributes` enforces it); Birch compares the files it ships byte
  for byte.
