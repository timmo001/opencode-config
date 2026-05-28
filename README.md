# OpenCode Config

Shared [OpenCode](https://opencode.ai) skills, agents, plugins, and commands.

Published from [`timmo001/dotfiles`](https://github.com/timmo001/dotfiles) — source at [`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.config/opencode).

## Installation

Clone the repo and copy what you need into your OpenCode config directory:

```bash
git clone https://github.com/timmo001/opencode-config.git
cd opencode-config

# Copy individual items
cp -r skills/diagnose ~/.agents/skills/
cp commands/inject-context.md ~/.config/opencode/commands/
cp plugins/env-protection.js ~/.config/opencode/plugins/
cp agents/reviewer.md ~/.config/opencode/agents/

# Or copy everything
cp -r skills ~/.agents/
cp -r agents commands plugins ~/.config/opencode/
```

> **Stow users:** If your OpenCode config is managed by [GNU Stow](https://www.gnu.org/software/stow/) or a similar symlink manager, the `cp` commands above will not work — they copy into the live path rather than your stow source directory. Either follow the [dotfiles setup](https://github.com/timmo001/dotfiles) this repo is published from, or ask an agent to adapt the files into your own stow structure.

Some skills and commands depend on plugins to function. Check the tables below for required plugins and install them alongside the skill or command.

### Importing Skills

Once you have the `import-external-skill` skill installed, you can use it to import skills from this or any public GitHub skills repo. Point it at a skill directory URL and it handles fetching, frontmatter conversion, and origin tracking:

```
# origin: https://github.com/timmo001/opencode-config/tree/main/skills/<skill-name>
```

It also supports a review mode: give it a repo URL and it will list all available skills, compare them against your local library, and recommend which to import, adapt, or skip.

Agents, commands, and plugins are not managed by `import-external-skill` — copy them manually as shown above.

### Minimum Configuration

This repo provides skills, agents, commands, and plugins but not an `opencode.json` config file. You need one to load them. Here is a minimal starting point:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  // Choose your provider and model
  "model": "anthropic/claude-sonnet-4-20250514",
  // Agents defined in agents/ are loaded automatically from ~/.config/opencode/agents/
  // MCP servers, tool overrides, and provider options go here as needed
}
```

Place it at `~/.config/opencode/opencode.json` (or `opencode.jsonc` for comments). See the [OpenCode docs](https://opencode.ai/docs/config) for the full configuration reference.

## How It Fits Together

The config is built around a few patterns:

- **Branch context injection** — The `branch-context` plugin pre-computes git and PR state once per command and injects it as structured XML. Commands that need current-branch context declare a dependency on this plugin instead of running their own `git`/`gh` calls.
- **Graduated agent permissions** — Agents range from fully read-only (`reviewer`, `ask`) through ask-gated (`build-ask`) to edit-capable (`refactorer`, `build-locked`). A guard plugin prevents read-only agents from escalating via subagent delegation.
- **Scoped cleanup commands** — Commands like `/types-enforce-ts`, `/cleanup-unnecessary-variables`, and `/remove-single-use-functions` combine branch-context work-scope with a matching skill and route through the `refactorer` agent, keeping changes within the current git diff.
- **Skill-based routing** — Commands are thin wrappers that name an agent, declare required skills, and state whether branch context is needed. The workflow logic lives in skills and plugins, not in the command itself.
- **Secret protection** — The `env-protection` plugin blocks reads of `.env` files (except `.env.example`) across all agents.

## Skills

| Skill | Description | Requires | Works with |
|---|---|---|---|

### From External Sources

These skills were imported from other repos. Some are used as-is; others have been adapted for local workflows and conventions.

| Skill | Origin | Local Changes | Requires | Works with |
|---|---|---|---|---|

## Agents

| Agent | Description |
|---|---|
| `ask` | Ask clarifying questions before taking action |
| `build-ask` | Build agent that executes clear tasks and relies on permissions for write actions |
| `build-locked` | Build agent that can edit files but cannot run shell commands |
| `general-readonly` | General-style parallel subagent that researches and runs commands but cannot modify workspace files via file tools (for delegation from read-only primaries). |
| `refactorer` | Refactor code while preserving behavior and following local command and skill workflows |
| `reviewer` | Reviews code for quality, bugs, security, and best practices |

## Commands

| Command | Description | Agent | Requires | Works with |
|---|---|---|---|---|
| `/all-lit-skills` | Apply all Lit rendering skills in current git scope | default | `branch-context` plugin |  |
| `/all-ts-skills` | Apply all TypeScript-specific skills in current git scope | default | `branch-context` plugin |  |
| `/check-skill-updates` | Check imported skills for upstream updates | default |  |  |
| `/debug-frontend` | Debug browser-specific UI issues with Chrome DevTools tools first | ask |  |  |
| `/explore-codebase` | Explore a codebase topic with the task explore subagent | ask |  |  |
| `/fallow-audit` | Audit changed JavaScript or TypeScript code with Fallow | ask |  |  |
| `/fallow-project-analyse` | Analyse a JavaScript or TypeScript project with Fallow | ask |  |  |
| `/handoff` | Write a handoff document for the next agent session | default |  |  |
| `/handoffs-list` | List handoff notes for the current repository | default |  |  |
| `/home-assistant/all-frontend-skills` | Apply all Home Assistant frontend skills in current git scope | default | `branch-context` plugin |  |
| `/home-assistant/lazy-context` | Review and fix Home Assistant frontend lazy-context and memoization usage in current git scope | default | `branch-context` plugin |  |
| `/home-assistant/list-components` | Migrate Home Assistant list components from MWC to new primitives in current git scope | default | `branch-context` plugin |  |
| `/home-assistant/lit-rendering` | Review and fix Home Assistant Lit rendering and picker callback-shape patterns in current git scope | default | `branch-context` plugin |  |
| `/home-assistant/migrate-dialog` | Migrate dialog(s) to ha-wa-dialog (path or name targets) | default |  |  |
| `/home-assistant/replace-spacing` | Replace hardcoded spacing values with ha-space tokens from core.globals.ts | default |  |  |
| `/import-external-skill` | Import or review external skills for the local skill library | default |  |  |
| `/improve-codebase-architecture` | Review a codebase area for architectural friction and focused structural improvements | plan |  |  |
| `/inject-context` | Inject branch context and optionally execute an instruction | default |  |  |
| `/investigate` | Investigate a topic, issue, or area without editing by default | ask |  |  |
| `/note-append` | Append new notes to an existing note file for the current repository | default |  |  |
| `/note-create` | Create a new note for the current repository in your Obsidian notes vault | default |  |  |
| `/note-reference` | Load one or more notes for the current repository into context | default |  |  |
| `/notes-list` | List notes for the current repository, optionally filtered by tag | default |  |  |
| `/notes-search` | Search notes for the current repository by topic, keyword, or tag | default |  |  |
| `/plan` | Manual entrypoint to native plan mode from the current conversation context | plan |  |  |
| `/refactor-cleanup-variables` | Refactor - inline and remove unnecessary variables from current git scope | refactorer | `branch-context` plugin |  |
| `/refactor-current-work` | Refactor current branch work while preserving behaviour | refactorer | `branch-context` plugin |  |
| `/refactor-enforce-types` | Refactor - enforce TypeScript type safety in current git scope | refactorer | `branch-context` plugin |  |
| `/refactor-remove-single-use` | Refactor - inline and remove safe single-use functions from current git scope | refactorer | `branch-context` plugin |  |
| `/reset-branch-reapply` | Reset branch to default and reapply current diff staged | build | `branch-context` plugin |  |
| `/review-current-work` | Review current branch work with BranchContextPlugin context | reviewer | `branch-context` plugin |  |

## Plugins

| Plugin | Description |
|---|---|
| `branch-context` | Injects branch-context blocks into command prompts before execution |
| `env-protection` | Blocks reads of .env files to prevent leaking secrets |
| `notes-guard` | Blocks direct LLM file tool access to the notes vault |
| `notification` | Plays a desktop notification sound when agent tasks complete |
| `readonly-subagent-task-guard` | Forces read-only primary agents to delegate only to non-modifying subagents |
| `repo-notes` | Injects repo-note context blocks into note commands |

## Publishing

This repo is published automatically via GitHub Actions when the source
[`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.config/opencode) changes.
