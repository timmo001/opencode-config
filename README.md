# OpenCode Config

Shared [OpenCode](https://opencode.ai) skills, agents, plugins, and commands.

Generated and published from [`timmo001/dotfiles`](https://github.com/timmo001/dotfiles), with shared skills sourced from [`timmo001/skills`](https://github.com/timmo001/skills).

See the [OpenCode overview](https://dotfiles.timmo.dev/agents/opencode/overview/) for the overview, MCP notes, and generated reference pages.

## Installation

Clone the repo and copy what you need into your OpenCode config directory:

```bash
git clone https://github.com/timmo001/opencode-config.git
cd opencode-config

# Copy individual items
cp -r skills/diagnose ~/.agents/skills/
cp commands/inject-context.md ~/.config/opencode/commands/
cp plugins/env-protection.ts ~/.config/opencode/plugins/
cp -r lib ~/.config/opencode/
cp agents/reviewer.md ~/.config/opencode/agents/

# Or copy everything
cp -r skills ~/.agents/
cp -r agents commands plugins lib ~/.config/opencode/
```

> **Stow users:** If your OpenCode config is managed by [GNU Stow](https://www.gnu.org/software/stow/) or a similar symlink manager, the `cp` commands above will not work — they copy into the live path rather than your stow source directory. Either follow the [dotfiles setup](https://github.com/timmo001/dotfiles) this repo is published from, or ask an agent to adapt the files into your own stow structure.

Some skills and commands depend on plugins to function. Check the tables below for required plugins and install them alongside the skill or command.

### Importing Skills

Once you have the `import-external-skill` skill installed, you can use it to import skills from this or any public GitHub skills repo. Point it at a skill directory URL and it handles fetching, frontmatter conversion, and origin tracking:

```
# origin: https://github.com/timmo001/skills/tree/main/<skill-name>
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
- **Graduated agent permissions** — Agents range from workspace-read-only (`reviewer`, `ask`) through ask-gated (`build-ask`) to edit-capable (`refactorer`). Read-only primary agents use native task allowlists, while terminal read-only subagents cannot delegate further.
- **Scoped cleanup commands** — Commands like `/refactor-enforce-types`, `/refactor-cleanup-variables`, and `/refactor-remove-single-use` combine branch-context work-scope with a matching skill and route through the `refactorer` agent, keeping changes within the current git diff.
- **Skill-based routing** — Commands are thin wrappers that name an agent, declare required skills, and state whether branch context is needed. The workflow logic lives in skills and plugins, not in the command itself.
- **Secret protection** — The `env-protection` plugin blocks reads of `.env` files (except `.env.example`) across all agents.

## Skills

| Skill | Description | Requires | Works with |
|---|---|---|---|
| `agent-oxlint` | Run the optional advisory Oxlint pass during JavaScript or TypeScript cleanup and slop-reduction work in dot-managed repositories. Use after the repository's own lint workflow; act only on diagnostics intersecting changed diff lines, while the command checks private opt-in and local Oxlint precedence. |  | `install-timmo-oxlint-rules` skill |
| `branch-context-consumer` | Consume BranchContextPlugin injections in commands. Use when a command depends on an injected <branch-context> block for its scope. | `branch-context` plugin |  |
| `changeset-scope` | Keep all scoped code work contained to the user-defined changeset. Use for implementation, fixes, diagnosis, refactoring, cleanup, and review when explicit instructions, named files, diffs, branches, pull requests, or injected work scopes define the boundary. | `branch-context` plugin | `branch-context-consumer` skill |
| `check-skill-updates` | Check imported skills for upstream changes and review safe updates. Use when a tracked `# origin:` may have changed or when refreshing installed skills from their source repositories. |  | `import-external-skill` skill |
| `chill` | Stop overengineering and reinventing the wheel. Use ONLY when the user explicitly invokes /chill or asks to simplify an approach that has become unnecessarily complex. | `changeset-scope` skill,`evidence-first` skill |  |
| `cleanup-unnecessary-variables` | Safe removal of unnecessary variables during code review and refactoring. Use when simplifying code, inlining temporary or single-use variables, or removing redundant aliases, while preserving runtime behaviour, evaluation order, and variables kept for readability or debugging. |  |  |
| `code-review` | Review code changes along two axes - Standards (does it follow the repo's conventions, plus a Fowler code-smell baseline?) and Spec (does it implement what the originating issue or spec asked for?). Use when reviewing a pull request, a branch, work-in-progress changes, or a diff. | `changeset-scope` skill,`effect-principles` skill,`workflows-watch` skill |  |
| `effect-principles` | Apply the Effect way of reasoning in codebases that do not use Effect, in any programming language. Use when editing or reviewing non-Effect code so dependencies, failures, state, boundaries, resources, time, and workflows stay explicit without adding Effect-shaped architecture or broader scope. |  | `changeset-scope` skill |
| `evidence-first` | Check questions and uncertain statements before answering, while following clear user choices and limits. Use in any agent mode when the user asks why or how something works, says things like I think, I remember, or I don't think, asks whether something is correct, requests advice, or gives a firm preference such as I don't want this, reduce the scope, or this is going too far. | `research` skill |  |
| `git-commit` | Commit workflow using the dot git-commit gateway, splitting a reviewed changeset into coherent commits by default. Use only after the user explicitly requests a commit or push, including /commit, /commit-push, or /commit-push-watch. Never infer authorisation for later changes; never run raw git commit. | `workflows-watch` skill | `upstream` skill |
| `git-context` | Patterns for working with git branches, remotes, diffs against the default branch, and rebases. Use when resolving rebase conflicts, continuing interactive rebases, amending commits, or any git operation that would open an interactive editor. | `branch-context` plugin,`git-commit` skill,`workflows-watch` skill | `upstream` skill |
| `handoff` | Compact the current conversation into a handoff document for another agent to pick up. |  |  |
| `herdr-workflows` | Apply local safeguards for Herdr session recovery and transferring linked-worktree changes back to a host checkout. Use alongside the herdr skill when diagnosing Herdr socket routing, recovering the default session, or moving, consolidating, or continuing Herdr worktree changes from the main or host checkout. The herdr skill remains authoritative for all Herdr CLI, topology, targeting, lifecycle, and safety behaviour. |  |  |
| `home-assistant-frontend` | Home Assistant frontend skill routing and personal engineering overlays. Use when editing or reviewing the Home Assistant frontend so repository-local `ha-frontend-*` skills stay authoritative and applicable Lit, TypeScript, cleanup, and HA companion skills are also loaded. | `home-assistant-lit-rendering` skill,`lit-rendering` skill |  |
| `home-assistant-lazy-context` | Home Assistant frontend lazy-context, memoization, and `hass` removal guidance. Use when migrating Lit components from `hass!: HomeAssistant`, `.hass=${...}`, or broad `hass` access to context slices. |  |  |
| `home-assistant-list-components` | Home Assistant list component migration and usage guidance. Use when editing ha-list, ha-list-item, ha-md-list, or migrating to ha-list-nav, ha-list-selectable, ha-list-item-button, ha-list-item-option, or ha-list-item-base. |  |  |
| `home-assistant-lit-rendering` | Home Assistant Lit rendering extensions for HA components and context-aware picker callback shape. |  | `lit-rendering` skill |
| `human-step-guide` | Prepare a concise guide when progress is blocked by a genuinely human-only action. Use for approvals, physical actions, credential entry, or dashboard steps the agent cannot perform; do not use for work available tools can complete. |  |  |
| `import-external-skill` | Import skills from external repositories into this Agent Skills repository. Use when pulling in a public skill, reviewing an external skill set, or adapting upstream content into an existing skill. |  |  |
| `install-tool` | Install tools, applications, CLIs, runtimes, and packages. Use when an installation request should prefer mise for development tools, then fall back to pacman or yay for system-integrated software. | `pkexec-root` skill |  |
| `lit-rendering` | Lit rendering and picker callback-shape guidance for editing and reviewing Lit components. |  |  |
| `maintain-docs` | Keep documentation current and accurate with recent code changes, across in-code docs (docstrings, annotations, comments), in-repo docs sites, and external docs repositories. Use when asked to update docs, check docs accuracy, keep documentation current, document recent changes, refresh docstrings or annotations, or catch documentation up with the codebase. Matches the codebase's existing documentation density and stops before commit. |  |  |
| `opencode-effect` | Develop and migrate OpenCode V2 plugins, clients, SDK hosts, and HTTP API integrations. Use for the OpenCode plugin API, `@opencode-ai/client`, `@opencode-ai/sdk`, server API, Effect entrypoints, or V1-to-V2 API migration. |  |  |
| `pitchfork-dev-servers` | Manage long-running local dev servers by precedence - the project's own AGENTS.md workflow first, framework-native background mode next, then pitchfork as the fallback. Use when starting, stopping, restarting, checking, or tailing development servers, background servers, `pitchfork.toml`, pitchfork MCP tools, or local AGENTS/mise tasks that mention pitchfork. |  |  |
| `pkexec-root` | Use pkexec first for commands that need root directly or indirectly. |  |  |
| `plan` | Produce implementation-ready plans from the current conversation and repository context. Use when entering native plan mode, invoking /plan, or when a task needs concrete implementation sequencing before edits begin; do not use for round-based grilling. | `staged-implementation` skill,`writing-style` skill |  |
| `remove-single-use-functions` | Safe inlining and removal of single-use functions during code review and refactoring. Use when a local, non-exported helper has exactly one real call site and inlining preserves behaviour and readability. |  |  |
| `research` | Investigate a topic against primary sources and return cited findings, comparing credible maintainer and contributor perspectives when judgement is involved. Use when the user asks why, says show evidence, validate this, or use trusted sources; wants research, docs, API, or spec facts; needs external library or GitHub behaviour verified; compares competing views; or delegates reading legwork to a background agent. |  |  |
| `safe-process-signals` | Safe process killing and signal handling for agent/subprocess contexts. Use when running pkill, killall, kill, or any process termination command from a shell subprocess, automated script, or coding agent. |  |  |
| `session-coordination` | Coordinate delegated agent sessions with bounded assignments, asynchronous background scheduling, soft concurrency caps, context-window rotation, independent review cycles, and logged cleanup across native child sessions and Herdr-managed agents. Use when managing multiple agents, panes, tabs, branches, stages, or long-running tasks while keeping the coordinating session small. | `changeset-scope` skill,`code-review` skill,`gh-stack` skill,`git-commit` skill,`git-context` skill,`herdr-workflows` skill,`staged-implementation` skill,`workflows-watch` skill |  |
| `shared-workflows` | Use, configure, maintain, or create reusable GitHub Actions workflows for personal and organisation repositories. Use when a task mentions shared workflows, reusable workflows, `workflow_call`, cross-repository workflow `uses:`, or the personal workflows repository; do not use for repository-specific or proof-of-concept CI unless evaluating whether it should be shared. |  |  |
| `staged-implementation` | Execute broad changes one coherent, independently verifiable stage at a time. Use when work spans multiple independently reviewable changes, or when contracts, producer-consumer migrations, generated artefacts, or release packaging create an ordered multi-stage rollout; skip small single-purpose changes. |  | `handoff` skill |
| `types-enforce-ts` | TypeScript type-safety guidance for editing and reviewing `.ts`, `.tsx`, `.mts`, and `.cts` files. |  |  |
| `workflows-watch` | Watch GitHub Actions workflows in an experimental background task and return the result. Use when asked to watch checks, wait for workflows, or follow workflow runs without blocking the main agent; diagnose and fix only when the caller explicitly requests fix mode. | `changeset-scope` skill,`diagnose` skill | `git-commit` skill |
| `writing-dot-skills` | Craft for authoring Agent Skills that select reliably and stay lean. Use when creating or revising a skill's description, workflow, references, scripts, or structure. |  |  |
| `writing-style` | Write commit messages, PR and issue text, docs (README), code comments, and user-facing strings (notifications, UI labels, toasts, error messages) in the project owner's voice: concise, human, UK English, no em-dashes, no robotic or marketing tone. Use when authoring or editing any of these. Defer to a repo's established house style when it has one; otherwise this sets the default voice. |  |  |

### From External Sources

These skills were imported from other repos. Some are used as-is; others have been adapted for local workflows and conventions.

| Skill | Origin | Local Changes | Requires | Works with |
|---|---|---|---|---|
| `add-oxlint-rule` | [timmo001/oxlint-rules](https://github.com/timmo001/oxlint-rules/tree/main/skills/add-oxlint-rule) | No | `release-oxlint-rules` skill |  |
| `agentic-workflows` | [github/gh-aw](https://github.com/github/gh-aw/tree/main/.github/skills/agentic-workflows) | Yes |  |  |
| `ask-questions-if-underspecified` | [trailofbits/skills](https://github.com/trailofbits/skills/tree/main/plugins/ask-questions-if-underspecified/skills/ask-questions-if-underspecified) | Yes |  | `grilling` skill |
| `bro` | [dmmulroy/skills](https://github.com/dmmulroy/skills/tree/main/bro) | Yes |  |  |
| `browser-control` | [anomalyco/browser-control](https://github.com/anomalyco/browser-control/tree/main/skills/browser-control) | Yes |  | `handoff` skill |
| `codebase-design` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/codebase-design) | Yes |  |  |
| `css-motion-systems` | [stolinski/s-stack](https://github.com/stolinski/s-stack/tree/main/skills/css-motion-systems) | Yes |  |  |
| `diagnose` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/diagnosing-bugs) | Yes |  |  |
| `domain-modeling` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/domain-modeling) | Yes |  |  |
| `gh-stack` | [github/gh-stack](https://github.com/github/gh-stack/tree/main/skills/gh-stack) | Yes | `git-commit` skill,`git-context` skill |  |
| `grilling` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling) | Yes |  |  |
| `html` | [stolinski/s-stack](https://github.com/stolinski/s-stack/tree/main/skills/html) | Yes |  |  |
| `improve-codebase-architecture` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/improve-codebase-architecture) | Yes |  | `grilling` skill |
| `install-timmo-oxlint-rules` | [timmo001/oxlint-rules](https://github.com/timmo001/oxlint-rules/tree/main/skills/install-timmo-oxlint-rules) | No |  |  |
| `prototype` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/prototype) | Yes |  |  |
| `release-oxlint-rules` | [timmo001/oxlint-rules](https://github.com/timmo001/oxlint-rules/tree/main/skills/release-oxlint-rules) | No |  |  |
| `show-me` | [dmmulroy/.dotfiles](https://github.com/dmmulroy/.dotfiles/tree/main/home/.agents/skills/show-me) | Yes |  |  |
| `to-questionnaire` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/to-questionnaire) | Yes |  |  |

## Agents

| Agent | Description |
|---|---|
| `build-ask` | Build agent that executes clear tasks and relies on permissions for write actions |
| `coordinator` | Manages low-context delegated sessions through delivery |
| `general-readonly` | General-style parallel subagent that researches with read-only tools and a narrow shell inspection allowlist (for delegation from read-only primaries). |
| `grill` | Read-only planning stress-test agent for light or full round-based grilling |
| `refactorer` | Refactor code while preserving behavior and following local command and skill workflows |
| `researcher-readonly` | Primary-source research subagent that compares claim-specific evidence and cannot delegate further |
| `researcher` | Interactive primary-source research agent that compares claim-specific evidence and may delegate one layer of read-only legwork |
| `reviewer` | Reviews code for quality, bugs, security, and best practices |
| `workflow-watcher` | Watches host-resolved GitHub Actions targets and optionally fixes an explicitly scoped quick-check failure without rediscovering workflows |

## Commands

| Command | Description | Agent | Requires | Works with |
|---|---|---|---|---|
| `/all-lit-skills` | Apply all Lit rendering skills in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill | `lit-rendering` skill |
| `/all-ts-skills` | Apply all TypeScript-specific skills in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill | `cleanup-unnecessary-variables` skill,`remove-single-use-functions` skill,`types-enforce-ts` skill |
| `/bro` | Re-pitch the previous response plainly, concisely, and with enough context | default |  |  |
| `/check-skill-updates` | Check imported skills for upstream updates | default |  | `import-external-skill` skill |
| `/code-review` | Review current branch work with the code-review skill and BranchContextPlugin context | reviewer | `branch-context` plugin,`branch-context-consumer` skill,`changeset-scope` skill,`effect-principles` skill |  |
| `/commit-push-watch` | Split current changes into coherent commits, push, then watch workflows | default | `git-commit` skill,`workflows-watch` skill |  |
| `/commit-push` | Split current changes into coherent commits and push without workflow watchers | default | `git-commit` skill,`workflows-watch` skill |  |
| `/commit` | Split current changes into coherent commits via the dot git-commit gateway | default | `git-commit` skill |  |
| `/debug-frontend` | Debug browser-specific UI issues with Chrome DevTools tools first | default |  |  |
| `/explore-codebase` | Explore a codebase topic with the task explore subagent | default |  |  |
| `/fix-workflows` | Diagnose and fix recent GitHub Actions failures, optionally scoped to a workflow or run | default | `diagnose` skill,`shared-workflows` skill |  |
| `/grill` | Stress-test a plan, decision, or idea with light or full question rounds | grill | `grilling` skill |  |
| `/handoff` | Write a handoff document for the next agent session | default |  |  |
| `/handoffs-list` | List handoff notes for the current repository | default |  | `handoff` skill |
| `/home-assistant/all-frontend-skills` | Apply all Home Assistant frontend skills in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill | `home-assistant-frontend` skill,`home-assistant-lit-rendering` skill,`lit-rendering` skill |
| `/home-assistant/lazy-context` | Review and fix Home Assistant frontend lazy-context and memoization usage in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill,`home-assistant-frontend` skill,`home-assistant-lazy-context` skill | `home-assistant-lit-rendering` skill,`lit-rendering` skill |
| `/home-assistant/list-components` | Migrate Home Assistant list components from MWC to new primitives in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill,`home-assistant-frontend` skill,`home-assistant-list-components` skill,`lit-rendering` skill |  |
| `/home-assistant/lit-rendering` | Review and fix Home Assistant Lit rendering and picker callback-shape patterns in current git scope | default | `branch-context` plugin,`branch-context-consumer` skill,`home-assistant-frontend` skill,`home-assistant-lit-rendering` skill |  |
| `/home-assistant/replace-spacing` | Replace hardcoded spacing values with ha-space tokens from core.globals.ts | default |  |  |
| `/import-external-skill` | Import or review external skills for the local skill library | default |  |  |
| `/improve-codebase-architecture` | Review a codebase area for architectural friction and focused structural improvements | plan |  |  |
| `/inject-context` | Inject branch and codebase stack context and optionally execute an instruction | default |  |  |
| `/inject-stack` | Inject codebase stack context and optionally execute an instruction | default |  |  |
| `/investigate` | Investigate a topic, issue, or area without editing by default | default |  | `diagnose` skill |
| `/note-append` | Append new notes to an existing note file for the current repository | default |  |  |
| `/note-create` | Create a new note for the current repository in your Obsidian notes vault | default |  |  |
| `/note-reference` | Load one or more notes, relevant skills, and next steps for the current repository | default |  |  |
| `/notes-list` | List notes for the current repository, optionally filtered by tag | default |  |  |
| `/notes-search` | Search notes for the current repository by topic, keyword, or tag | default |  |  |
| `/plan` | Manual entrypoint to native plan mode from the current conversation context | plan |  |  |
| `/plannotator-annotate` | Open interactive annotation UI for a file, folder, or URL | default |  |  |
| `/plannotator-last` | Annotate the last assistant message | default |  |  |
| `/plannotator-review` | Open interactive code review for current changes or a PR URL; pass --git to force Git in JJ workspaces | default |  |  |
| `/refactor-cleanup-variables` | Refactor - inline and remove unnecessary variables from current git scope | refactorer | `branch-context` plugin,`branch-context-consumer` skill,`cleanup-unnecessary-variables` skill |  |
| `/refactor-current-work` | Refactor current branch work while preserving behaviour | refactorer | `branch-context` plugin,`branch-context-consumer` skill |  |
| `/refactor-enforce-types` | Refactor - enforce TypeScript type safety in current git scope | refactorer | `branch-context` plugin,`branch-context-consumer` skill,`types-enforce-ts` skill |  |
| `/refactor-remove-single-use` | Refactor - inline and remove safe single-use functions from current git scope | refactorer | `branch-context` plugin,`branch-context-consumer` skill,`remove-single-use-functions` skill |  |
| `/research` | Research a topic from primary sources and compare evidence where judgement is involved | researcher |  |  |
| `/reset-branch-reapply` | Reset branch to default and reapply current diff staged | build | `branch-context` plugin,`branch-context-consumer` skill |  |
| `/session-reference` | Load another OpenCode session into this conversation by its sidebar title | default |  |  |
| `/update-docs` | Keep documentation current with recent code changes, via Context MCP and delegated investigation | default | `maintain-docs` skill,`writing-style` skill |  |

## Plugins

| Plugin | Description |
|---|---|
| `branch-context` | Injects branch-context blocks into command prompts before execution |
| `commit-context` | Injects session-attributed commit scope into commit command prompts |
| `context-capture` | Opt-in capture of the assembled starter context for token profiling |
| `context-zone-warning` | Warns when long-context models enter less reliable context ranges |
| `env-protection` | Blocks direct access to .env files to prevent leaking secrets |
| `generated-artifact-guard` | Blocks direct mutation of generated dotfiles artefacts |
| `mcp-repo-gate` | Per-repo MCP server gating for OpenCode |
| `notes-guard` | Blocks direct file and shell access to the repository notes vault |
| `notification` | Sends contextual desktop notifications and terminal attention for agent events |
| `pitchfork-dev-server-guard` | Enforces a project's declared pitchfork dev-server workflow for agents |
| `readonly-subagent-shell-guard` | Rejects shell syntax that can turn read-only subagent commands into writes |
| `repo-notes` | Injects repo-note context into OpenCode note commands |
| `stack-context` | Injects codebase stack-context blocks into prompts |
| `subagent-chrome-devtools-guard` | Blocks Chrome DevTools tools from delegated subagent sessions |
| `workflow-manifest` | Resolves pushed GitHub Actions runs into a compact watcher manifest |

## Publishing

This repo is published automatically via GitHub Actions when the OpenCode config
[`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy-quattro/agents/.config/opencode) or the pinned
[`timmo001/skills`](https://github.com/timmo001/skills) revision changes.
