---
description: Continue the current session in Birch Code
disable-model-invocation: true
---

Hand this Claude Code session off to the Birch desktop app — it comes to the foreground and opens a
terminal at this directory that resumes this exact conversation (`claude --resume`):

!`birch open --session-id "${CLAUDE_SESSION_ID}" --cwd "$(pwd)"`
