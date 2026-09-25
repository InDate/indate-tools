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
re-driven by hand. The plugin starts its MCP server with `npx`, at the version
pinned beside it, so it needs Node.js and nothing installed by hand.

**[chsum](https://github.com/InDate/chsum)** — recover past Claude Code and Codex
CLI sessions as verbatim context. Lists a project's sessions, digests one into
your prompts in order, files changed, commands run, and where it left off.
Nothing is model-generated. The plugin carries the code and needs Python 3.10+;
its hooks run it with nothing on `PATH`. A `chsum` command of your own is
optional — see chsum's README.

## How this repo works

Each plugin lives in its own repository. This one holds
`.claude-plugin/marketplace.json`, which points at them by URL and commit sha,
and `.github/workflows/pin-plugin.yml`, which updates those pins.

Prefer pinning a subdirectory (`git-subdir` with a real `path`) over the whole
repository. The installer runs `npm install` in the plugin directory, so a repo
root containing `package.json` costs ~175MB of `node_modules` per installed
version — devharness's root measured 175MB, 90MB of it dev dependencies; a
subtree holding just the manifest and skill costs nothing.

`path: "."` is the one shape to avoid — it checks out root-level files without
recursing, so `skills/` never arrives and the plugin installs as a shell that
does nothing. The pin workflow rejects it.

Pinning is deliberate: a release in a tool repo doesn't reach anyone until the sha
here is bumped. One extra commit per release, in exchange for controlling exactly
what ships and being able to roll it back.

## Releasing

Tagging a tool repo opens a pull request here.

```
tag v0.8.1 in InDate/devharness
  → notify-marketplace.yml dispatches to this repo
  → pin-plugin.yml validates, repins marketplace.json, opens a PR
  → you merge; the release reaches installed users
```

The PR step is the point. Tagging publishes to npm or PyPI, which only affects
people who go and install it. Merging the pin is what pushes an update at
everyone who already has the plugin — a separate decision, so it gets a separate
approval.

Validation before the PR opens: the sha must be 40 hex characters and must
actually exist in the tool's repository, the plugin must already be listed here,
and the version must be semver. The payload arrives from another repository, so
none of it is taken on trust.

The PR then waits for npm. A plugin whose `.mcp.json` runs `npx -y <pkg>@<version>`
fails to start until npm serves that version, and `npm publish` returns minutes
before it does. The workflow reads `.mcp.json` at the pinned sha and opens the PR
only once every pinned package resolves; after 15 minutes it fails instead. A
plugin with no `.mcp.json` skips the wait.

It also refuses a sha that moves under an unchanged version. Installed plugins
are cached per version, so that pin would merge cleanly and reach nobody — the
existing installs see the same version and never refetch. Bump the version and
re-tag instead.

Repinning by hand, if a dispatch is ever missed:

```sh
gh workflow run pin-plugin.yml -R InDate/indate-tools \
  -f name=devharness -f version=0.8.1 -f ref=v0.8.1 -f sha=<40-char-sha>
```

### One-time setup

Each tool repo needs a `MARKETPLACE_DISPATCH_TOKEN` secret: a fine-grained PAT
scoped to `InDate/indate-tools` with **contents: write** and
**pull requests: write**. A repo's own `GITHUB_TOKEN` can't reach another
repository, which is why this exists.
