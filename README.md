# Birch plugins

Agent plugins for [Birch](https://www.getbirchcode.dev/), the desktop developer tool for managing multiple coding agents and integrating them into the development workflow. They let Birch show, for each workspace, what the agent running there is doing: ready, working, waiting for you, done, or stopped with an error.

| Plugin | Agent | Directory | Marketplace manifest |
| --- | --- | --- | --- |
| `birch-status` | [Claude Code](https://code.claude.com/) | [`claude-code/`](claude-code/) | [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) |
| `birch-status` | [Codex](https://developers.openai.com/codex/) | [`codex/`](codex/) | [`.agents/plugins/marketplace.json`](.agents/plugins/marketplace.json) |

Both marketplaces are named `birch`, so the plugin id is `birch-status@birch` in either agent.

## What the plugins do

Each plugin registers hooks for the agent's lifecycle events (session start, prompt submitted, tool use, waiting for permission or input, turn finished, session end, sub-agents starting and stopping).
Each hook invokes the birch command-line tool that ships with Birch. The CLI passes the event to the running Birch app. The Claude Code plugin also adds a `/birch` command that continues the current session in a Birch app. The per-plugin READMEs list every hook and what Birch shows for it.

**The plugins make no network requests.** The hooks pass the agent's payload — including the session ID, working directory, and event details - to the Birch app running on the same computer over a local named pipe. If Birch isn't running, the hooks exit silently within a second.

The `birch` command must be on your `PATH`; Birch can install it from **Settings -> Status
integrations**.

## Installing

**You normally don't need to.** Birch installs and updates these plugins itself from **Settings -> Status integrations**, and offers them when you first use each agent. Birch bundles a pinned copy of this repository, writes it to its data directory, and registers that copy with each agent as a local marketplace. The installed plugin therefore always matches the Birch version you run, and installation requires no network access.

To install a plugin by hand instead:

```bash
# Claude Code
claude plugin marketplace add BirchHQ/birch-plugins
claude plugin install birch-status@birch

# Codex
codex plugin marketplace add BirchHQ/birch-plugins
codex plugin add birch-status@birch
```

Start a new agent session afterwards. Codex also asks you to review and trust the new hooks: open `/hooks` in the new session.

## Reporting a security issue

Use [private vulnerability reporting](https://github.com/BirchHQ/birch-code/security/advisories/new). Do not open a public issue for a suspected vulnerability. See [SECURITY.md](SECURITY.md).
