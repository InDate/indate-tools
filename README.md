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

Sources are whole-repo (`source: "url"`). A path-scoped `git-subdir` source with
`path: "."` checks out root-level files and no subdirectories, so `skills/` never
arrives and the plugin installs as a working shell that does nothing. The pin
workflow now rejects a path-scoped source for that reason.

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
