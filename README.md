# OpenCode Config

Shared [OpenCode](https://opencode.ai) skills, agents, plugins, and commands.

Published from [`timmo001/dotfiles`](https://github.com/timmo001/dotfiles) — OpenCode config at [`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.config/opencode) and shared skills at [`agents/.agents/skills/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.agents/skills).

## Installation

Clone the repo and copy what you need into your OpenCode config directory:

```bash
git clone https://github.com/timmo001/opencode-config.git
cd opencode-config

# Copy individual items
cp -r skills/diagnose ~/.agents/skills/
cp commands/inject-context.md ~/.config/opencode/commands/
cp plugins/env-protection.ts ~/.config/opencode/plugins/
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
| `branch-context-consumer` | Consume BranchContextPlugin injections in commands. Use when a command depends on an injected <branch-context> block for its scope. | `branch-context` plugin |  |
| `check-skill-updates` | Check imported skills for upstream changes and apply updates. Use when reviewing whether externally imported skills have new upstream content, or when `dot skill-updates` reports available changes. |  | `import-external-skill` skill |
| `cleanup-unnecessary-variables` | Safe unnecessary-variable cleanup guidance for code review and refactoring. |  |  |
| `dotfiles-stow` | REQUIRED when changing configs managed by ~/.config/dotfiles or ~/.config/dotfiles-private. Enforces editing stow source paths (not ad-hoc live paths) and using the dot command for stow/update/validation workflows. |  |  |
| `git-context` | Patterns for working with git branches, remotes, diffs against the default branch, and rebases. Use when resolving rebase conflicts, continuing interactive rebases, amending commits, or any git operation that would open an interactive editor. | `branch-context` plugin |  |
| `grill-questions` | Run an extended one-question-at-a-time planning interview to stress-test a proposed change before implementation. Use when the user says grill, grill me, stress-test this plan, ask more questions, or wants to expand the planning/question window instead of moving straight to a plan. |  | `ask-questions-if-underspecified` skill |
| `handoff` | Compact the current conversation into a handoff document for another agent to pick up. |  |  |
| `home-assistant-frontend` | Home Assistant frontend development with Lit Web Components and TypeScript. Use when working in the Home Assistant frontend repo, editing ha-* components, reviewing HA PRs, or applying HA-specific conventions (localization, theming, dialogs, panels, cards). |  | `cleanup-unnecessary-variables` skill,`home-assistant-lazy-context` skill,`home-assistant-list-components` skill,`home-assistant-lit-rendering` skill,`lit-rendering` skill,`remove-single-use-functions` skill,`types-enforce-ts` skill |
| `home-assistant-lazy-context` | Home Assistant frontend lazy-context, memoization, and `hass` removal guidance. Use when migrating Lit components from `hass!: HomeAssistant`, `.hass=${...}`, or broad `hass` access to context slices. |  |  |
| `home-assistant-list-components` | Home Assistant list component migration and usage guidance. Use when editing ha-list, ha-list-item, ha-md-list, or migrating to ha-list-nav, ha-list-selectable, ha-list-item-button, ha-list-item-option, or ha-list-item-base. |  |  |
| `home-assistant-lit-rendering` | Home Assistant Lit rendering extensions for HA components and context-aware picker callback shape. |  | `lit-rendering` skill |
| `import-external-skill` | Import skills from external repos into the local dotfiles skill library. Use when pulling in a skill from a public repo, reviewing an external skill set for useful additions, or adapting external skill content into existing local skills. |  |  |
| `lit-rendering` | Lit rendering and picker callback-shape guidance for editing and reviewing Lit components. |  |  |
| `pkexec-root` | Use pkexec first for commands that need root directly or indirectly. |  |  |
| `pr-review` | Guidelines for reviewing pull requests - what to analyze, review etiquette, and output formatting |  |  |
| `remove-single-use-functions` | Safe single-use function removal guidance for code review and refactoring. |  |  |
| `safe-process-signals` | Safe process killing and signal handling for agent/subprocess contexts. Use when running pkill, killall, kill, or any process termination command from a shell subprocess, automated script, or coding agent. |  |  |
| `types-enforce-ts` | TypeScript type-safety guidance for editing and reviewing `.ts`, `.tsx`, `.mts`, and `.cts` files. |  | `fallow` skill |

### From External Sources

These skills were imported from other repos. Some are used as-is; others have been adapted for local workflows and conventions.

| Skill | Origin | Local Changes | Requires | Works with |
|---|---|---|---|---|
| `ask-questions-if-underspecified` | [trailofbits/skills](https://github.com/trailofbits/skills/tree/main/plugins/ask-questions-if-underspecified/skills/ask-questions-if-underspecified) | Yes |  | `grill-questions` skill |
| `css-motion-systems` | [stolinski/s-stack](https://github.com/stolinski/s-stack/tree/main/skills/css-motion-systems) | Yes |  |  |
| `diagnose` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/diagnose) | Yes |  |  |
| `effect` | [anomalyco/opencode](https://github.com/anomalyco/opencode/tree/dev/.opencode/skills/effect) | No |  |  |
| `fallow` | [fallow-rs/fallow-skills](https://github.com/fallow-rs/fallow-skills/tree/main/fallow/skills/fallow) | No |  |  |
| `html` | [stolinski/s-stack](https://github.com/stolinski/s-stack/tree/main/skills/html) | Yes |  |  |
| `improve-codebase-architecture` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/improve-codebase-architecture) | Yes |  |  |
| `motion-choreography-patterns` | [stolinski/s-stack](https://github.com/stolinski/s-stack/tree/main/skills/motion-choreography-patterns) | No |  |  |
| `opentui` | [anomalyco/opentui](https://github.com/anomalyco/opentui/tree/main/packages/web/src/content) | Yes |  |  |
| `write-a-skill` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/write-a-skill) | Yes |  |  |

## Agents

| Agent | Description |
|---|---|
| `ask` | Ask clarifying questions before taking action |
| `build-ask` | Build agent that executes clear tasks and relies on permissions for write actions |
| `build-locked` | Build agent that can edit files but cannot run shell commands |
| `general-readonly` | General-style parallel subagent that researches with read-only tools and a narrow shell inspection allowlist (for delegation from read-only primaries). |
| `grill` | Extended read-only planning interview agent for one-question-at-a-time grilling |
| `refactorer` | Refactor code while preserving behavior and following local command and skill workflows |
| `reviewer` | Reviews code for quality, bugs, security, and best practices |

## Commands

| Command | Description | Agent | Requires | Works with |
|---|---|---|---|---|
| `/all-lit-skills` | Apply all Lit rendering skills in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill | `lit-rendering` skill |
| `/all-ts-skills` | Apply all TypeScript-specific skills in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill | `cleanup-unnecessary-variables` skill,`fallow` skill,`remove-single-use-functions` skill,`types-enforce-ts` skill |
| `/check-skill-updates` | Check imported skills for upstream updates | default |  | `import-external-skill` skill |
| `/debug-frontend` | Debug browser-specific UI issues with Chrome DevTools tools first | ask |  |  |
| `/explore-codebase` | Explore a codebase topic with the task explore subagent | ask |  |  |
| `/fallow-audit` | Audit changed JavaScript or TypeScript code with Fallow | ask | `fallow` skill |  |
| `/fallow-project-analyse` | Analyse a JavaScript or TypeScript project with Fallow | ask | `fallow` skill |  |
| `/grill` | Stress-test a proposed change with one-question-at-a-time planning questions | grill | `grill-questions` skill |  |
| `/handoff` | Write a handoff document for the next agent session | default |  |  |
| `/handoffs-list` | List handoff notes for the current repository | default |  | `handoff` skill |
| `/home-assistant/all-frontend-skills` | Apply all Home Assistant frontend skills in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill | `cleanup-unnecessary-variables` skill,`home-assistant-frontend` skill,`home-assistant-lazy-context` skill,`home-assistant-list-components` skill,`home-assistant-lit-rendering` skill,`lit-rendering` skill,`remove-single-use-functions` skill,`types-enforce-ts` skill |
| `/home-assistant/lazy-context` | Review and fix Home Assistant frontend lazy-context and memoization usage in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill,`home-assistant-frontend` skill,`home-assistant-lazy-context` skill | `home-assistant-lit-rendering` skill,`lit-rendering` skill |
| `/home-assistant/list-components` | Migrate Home Assistant list components from MWC to new primitives in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill,`home-assistant-frontend` skill,`home-assistant-list-components` skill,`lit-rendering` skill |  |
| `/home-assistant/lit-rendering` | Review and fix Home Assistant Lit rendering and picker callback-shape patterns in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill,`home-assistant-frontend` skill,`home-assistant-lit-rendering` skill |  |
| `/home-assistant/replace-spacing` | Replace hardcoded spacing values with ha-space tokens from core.globals.ts | default |  |  |
| `/import-external-skill` | Import or review external skills for the local skill library | default |  |  |
| `/improve-codebase-architecture` | Review a codebase area for architectural friction and focused structural improvements | plan |  |  |
| `/inject-context` | Inject branch context and optionally execute an instruction | default |  |  |
| `/investigate` | Investigate a topic, issue, or area without editing by default | ask |  | `diagnose` skill |
| `/note-append` | Append new notes to an existing note file for the current repository | default |  |  |
| `/note-create` | Create a new note for the current repository in your Obsidian notes vault | default |  |  |
| `/note-reference` | Load one or more notes, relevant skills, and next steps for the current repository | default |  |  |
| `/notes-list` | List notes for the current repository, optionally filtered by tag | default |  |  |
| `/notes-search` | Search notes for the current repository by topic, keyword, or tag | default |  |  |
| `/plan` | Manual entrypoint to native plan mode from the current conversation context | plan |  |  |
| `/refactor-cleanup-variables` | Refactor - inline and remove unnecessary variables from current git scope | refactorer | `branch-context` plugin,`branch-context-consumer` skill,`cleanup-unnecessary-variables` skill |  |
| `/refactor-current-work` | Refactor current branch work while preserving behaviour | refactorer | `branch-context` plugin,`branch-context-consumer` skill |  |
| `/refactor-enforce-types` | Refactor - enforce TypeScript type safety in current git scope | refactorer | `branch-context` plugin,`branch-context-consumer` skill,`types-enforce-ts` skill |  |
| `/refactor-remove-single-use` | Refactor - inline and remove safe single-use functions from current git scope | refactorer | `branch-context` plugin,`branch-context-consumer` skill,`remove-single-use-functions` skill |  |
| `/reset-branch-reapply` | Reset branch to default and reapply current diff staged | build | `branch-context` plugin,`branch-context-consumer` skill |  |
| `/review-current-work` | Review current branch work with BranchContextPlugin context | reviewer | `branch-context` plugin,`branch-context-consumer` skill,`pr-review` skill |  |

## Plugins

| Plugin | Description |
|---|---|
| `branch-context` | Injects branch-context blocks into command prompts before execution |
| `env-protection` | Blocks direct access to .env files to prevent leaking secrets |
| `notes-guard` |  |
| `notification` | Plays a desktop notification sound when agent tasks complete |
| `readonly-subagent-task-guard` | Forces read-only primary agents to delegate only to non-modifying subagents |
| `repo-notes` |  |
| `tui-dot-git-diff` |  |
| `tui-lazygit` |  |

## Publishing

This repo is published automatically via GitHub Actions when the OpenCode config
[`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.config/opencode) or shared skills
[`agents/.agents/skills/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.agents/skills) change.
