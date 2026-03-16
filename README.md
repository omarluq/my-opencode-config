# OpenCode Config

This repo contains a personal `opencode.json` setup plus local plugins, context files, skills, and agent instructions for running OpenCode with a research-heavy, approval-gated, beads-aware workflow.

The intended model is to commit a version of this config into each project repo you care about. It is not just a static global dotfiles setup; it is meant to evolve alongside the codebase so the agent can learn the project's real stack, conventions, and workflows.

It mixes a few ecosystems:

- OpenCode as the main CLI/runtime
- `ocx` for package/profile management
- `beads` (`bd`) and `beads_viewer` (`bv`) for issue tracking and triage
- `opencode-workspace` ideas and components for plans, reviews, and worktrees
- OpenAgentsControl patterns for approval gates, context-first discovery, and staged execution

## What Is In This Repo

- Root config: `opencode.json`
- Local context and standards: `.opencode/context/`
- Local plugins: `.opencode/plugins/`
- Skills: `.opencode/skills/`
- Agent instructions: `AGENTS.md`

The current config uses `openai/gpt-5.4` as the default model and adds both npm-hosted plugins and MCP servers.

## How This Config Is Meant To Be Used

Treat this repo as a per-project OpenCode config starter.

- Commit it into a project repository, or adapt parts of it into that project's own OpenCode config
- Keep `core/`-style standards stable and reusable
- Let project-specific context evolve as the agent learns the real codebase
- Expect some context areas to begin as scaffolding rather than final documentation

In particular, `.opencode/context/development/` is meant to be morphed over time by your OpenCode agent to reflect the actual stack in the repository it is working in.

That means the `development/` tree may intentionally contain:

- placeholder navigation
- example topic layouts
- future-file references
- starter guidance that should later be replaced with project-specific knowledge

The goal is not for `development/` to stay generic forever. The goal is for it to gradually become a better map of your real backend, frontend, data, infra, and workflow patterns as the agent works with the repo.

## Prerequisites

Before using this config, make sure you have:

- Git
- Node.js and `npm`/`npx`
- Python plus `uvx` (used by the tree-sitter MCP)
- OpenCode itself installed and working

## Install Companion Tools

### 1) Install `ocx`

`ocx` is the easiest way to manage OpenCode packages and profiles.

```bash
curl -fsSL https://ocx.kdco.dev/install.sh | sh
```

Alternative:

```bash
npm install -g ocx
```

Useful follow-up commands from the upstream docs:

```bash
ocx init --global
ocx profile add ws --source tweak/p-1vp4xoqv --from https://tweakoc.com/r --global
ocx oc -p ws
```

### 2) Install `beads` (`bd`)

This repo uses beads for issue tracking.

```bash
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash
```

Alternatives mentioned upstream:

```bash
npm install -g @beads/bd
brew install beads
```

After installing, initialize beads inside a repo if needed:

```bash
bd init
```

### 3) Install `beads_viewer` (`bv`)

`bv` is the graph-aware triage companion for beads.

```bash
brew install dicklesworthstone/tap/bv
```

Important: do not run bare `bv` in agent workflows. Use robot mode, for example:

```bash
bv --robot-triage
bv --robot-next
```

### 4) Install `opencode-workspace`

If you want the upstream workspace bundle directly via `ocx`:

```bash
ocx add kdco/workspace --from https://registry.kdco.dev
```

This repo already contains local workspace-style pieces such as plan management, worktrees, review skills, and context files, so this package is most useful if you want the broader upstream package as well.

### 5) Install OpenAgentsControl

This config borrows heavily from the OpenAgentsControl workflow style.

```bash
curl -fsSL https://raw.githubusercontent.com/darrenhinde/OpenAgentsControl/main/install.sh | bash -s developer
```

## Quick Start

Clone the repo and launch OpenCode from the repo root so it picks up `opencode.json`.

```bash
git clone <your-fork-or-copy> my-opencode-config
cd my-opencode-config
opencode
```

If you use beads in this repo, the most common commands are:

```bash
bd ready --json
bd show <id> --json
bd update <id> --status in_progress --json
bd close <id> --reason "Completed" --json
```

## Included Plugins

### npm-hosted plugins from `opencode.json`

- `@tarquinen/opencode-dcp@1.2.7`
- `@franlol/opencode-md-table-formatter@0.0.3`
- `opencode-beads`
- `opencode-pty`
- `opencode-morph-plugin`
- `micode`
- `@nick-vi/opencode-type-inject`
- `@spoons-and-mirrors/subtask2@latest`

### Local plugins bundled in this repo

- `background-agents` - adds async delegation tools:
  - `delegate`
  - `delegation_read`
  - `delegation_list`
- `workspace-plugin` - adds plan persistence tools:
  - `plan_save`
  - `plan_read`
- `worktree` - adds isolated worktree tools:
  - `worktree_create`
  - `worktree_delete`
- `notify` - native desktop/terminal notifications

## Included MCP Servers

The current `opencode.json` config includes these MCP entries:

- `treesitter`
  - Local command: `uvx mcp-server-tree-sitter`
  - Purpose: code structure, AST queries, symbol and dependency analysis
- `firecrawl`
  - Local command: `npx -y firecrawl-mcp`
  - Purpose: scraping, mapping, search, browser automation
- `context7`
  - Remote endpoint: `https://mcp.context7.com/mcp`
  - Purpose: current library and framework documentation
- `exa`
  - Remote endpoint: `https://mcp.exa.ai/mcp`
  - Purpose: web search and research
- `gh_grep`
  - Remote endpoint: `https://mcp.grep.app`
  - Purpose: remote code search across public repositories
- `deepwiki`
  - Local command: `npx -y mcp-deepwiki@latest`
  - Purpose: fetch DeepWiki repo documentation as Markdown

## API Keys And Environment Variables

This repo does not centrally define all API keys in versioned config, so treat the list below as a practical setup guide: some are verified by common upstream usage, and some are likely requirements depending on your account and how you run the MCP.

### Usually needed outside the repo config

- `OPENAI_API_KEY`
  - Likely needed because the default model in `opencode.json` is `openai/gpt-5.4`
- `FIRECRAWL_API_KEY`
  - Commonly required for `firecrawl-mcp`
- `EXA_API_KEY`
  - Commonly required for Exa MCP access

### Sometimes needed or account-dependent

- `CONTEXT7_API_KEY`
  - Context7 often works without a key for basic usage, but higher limits or account-specific access may need one
- `GITHUB_TOKEN` or `GH_TOKEN`
  - Not explicitly configured in this repo for `gh_grep`, but GitHub auth is commonly useful elsewhere in OpenCode and GitHub tooling
- `MORPH_API_KEY`
  - Likely needed if you want the Morph fast-apply plugin to use its hosted editing backend

### Usually not required for this repo's MCP entries

- `treesitter`
  - Local MCP, no external API key expected
- `deepwiki`
  - Usually works without a user API key for public repos

## Related Upstream Projects

- `ocx`: <https://github.com/kdcokenny/ocx>
- `beads`: <https://github.com/steveyegge/beads>
- `beads_viewer`: <https://github.com/Dicklesworthstone/beads_viewer>
- `opencode-workspace`: <https://github.com/kdcokenny/opencode-workspace>
- `OpenAgentsControl`: <https://github.com/darrenhinde/OpenAgentsControl>
