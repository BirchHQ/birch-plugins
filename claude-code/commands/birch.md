---
description: Continue the current Claude Code session in Birch
disable-model-invocation: true
---

Continue this Claude Code session in the Birch desktop app. Birch comes to the foreground and opens a terminal in the current directory, resuming this exact session:

!`birch open --session-id "${CLAUDE_SESSION_ID}" --cwd "$(pwd)"`