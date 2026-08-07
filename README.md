# indate-tools

Claude Code plugin marketplace.

```
/plugin marketplace add InDate/indate-tools
/plugin install devharness@indate-tools
/plugin install chsum@indate-tools
```

## Plugins

**[devharness](https://github.com/InDate/devharness)** — run and debug a live app
from inside Claude Code. Breakpoints and variable inspection in Chrome and
Node.js, managed dev servers, and replayable call history so setup is never
re-driven by hand. The MCP server installs separately: `npx devharness@latest`.

**[chsum](https://github.com/InDate/chsum)** — recover past Claude Code sessions
as verbatim context. Lists a project's sessions, digests one into your prompts in
order, files changed, commands run, and where it left off. Nothing is
model-generated. Command installs separately: `pipx install chsum`.

## How this repo works

Each plugin lives in its own repository. This one holds only
`.claude-plugin/marketplace.json`, which points at them by URL and commit sha.

Pinning is deliberate: a release in a tool repo doesn't reach anyone until the sha
here is bumped. One extra commit per release, in exchange for controlling exactly
what ships and being able to roll it back.
