# OpenCode Config

Shared [OpenCode 2](https://opencode.ai) skills, agents, plugins, and commands. The plugins use the OpenCode 2 Effect plugin API (`@opencode/plugin`).

Generated and published from [`timmo001/dotfiles`](https://github.com/timmo001/dotfiles), with shared skills sourced from [`timmo001/skills`](https://github.com/timmo001/skills).

See the [OpenCode overview](https://dotfiles.timmo.dev/agents/opencode/overview/) for the overview, MCP notes, and generated reference pages.

## Installation

Clone the repo and copy what you need into your OpenCode config directory:

```bash
git clone https://github.com/timmo001/opencode-config.git
cd opencode-config

# Copy individual items
cp -r skills/diagnose ~/.agents/skills/
cp commands/code-review.md ~/.config/opencode/commands/
cp -r plugins lib ~/.config/opencode/
bun install --cwd ~/.config/opencode/plugins
cp agents/reviewer.md ~/.config/opencode/agents/

# Or copy everything
cp -r skills ~/.agents/
cp -r agents commands plugins lib ~/.config/opencode/
bun install --cwd ~/.config/opencode/plugins
```

> **Stow users:** If your OpenCode config is managed by [GNU Stow](https://www.gnu.org/software/stow/) or a similar symlink manager, the `cp` commands above will not work — they copy into the live path rather than your stow source directory. Either follow the [dotfiles setup](https://github.com/timmo001/dotfiles) this repo is published from, or ask an agent to adapt the files into your own stow structure.

Plugins share modules from `plugins/lib/` and `lib/`, and their dependencies are declared in `plugins/package.json`, so copy the whole `plugins` and `lib` directories rather than single plugin files. Some skills and commands depend on plugins to function. Check the tables below for required plugins and install them alongside the skill or command.

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

Place it at `~/.config/opencode/opencode.json` (or `opencode.jsonc` for comments). See the [OpenCode docs](https://opencode.ai/v2/docs/config) for the full configuration reference.

## How It Fits Together

The config is built around a few patterns:

- **Graduated agent permissions** — Agents range from workspace-read-only (`reviewer`, `ask`) through ask-gated (`build-ask`) to edit-capable (`refactorer`). Read-only primary agents use native task allowlists, while terminal read-only subagents cannot delegate further.
- **Secret protection** — The `env-protection` plugin blocks reads of `.env` files (except `.env.example`) across all agents.

## Skills

| Skill | Description | Requires | Works with |
|---|---|---|---|
| `asking-questions` | Decide when to ask the user a question and how to present it. Use before any clarifying question, choice, or confirmation, whether through a structured question tool or in chat, and to decide whether to hold questions until findings are presented or divert to grilling; do not use to run light or full grilling rounds. | `grilling` skill | `bro` skill |
| `browser-access` | Decide whether browser access is needed and keep authorised checks narrow. Use for frontend or UI diagnosis, before proposing or using Chrome DevTools or other browser automation, and when the user explicitly requests browser interaction. |  |  |
| `changeset-scope` | Keep all scoped code work contained to the user-defined changeset. Use for implementation, fixes, diagnosis, refactoring, cleanup, and review when explicit instructions, named files, diffs, branches, pull requests, or injected work scopes define the boundary. |  |  |
| `check-skill-updates` | Check imported skills for upstream changes and review safe updates. Use when a tracked `# origin:` may have changed or when refreshing installed skills from their source repositories. |  | `import-external-skill` skill |
| `chill` | Stop overengineering and reinventing the wheel. Use when the user says a change has become overly complicated, out of scope, or is rebuilding something an existing tool or library already does. | `changeset-scope` skill,`evidence-first` skill | `gh-stack` skill |
| `cleanup-unnecessary-variables` | Safe removal of unnecessary variables during code review and refactoring. Use when simplifying code, inlining temporary or single-use variables, or removing redundant aliases, while preserving runtime behaviour, evaluation order, and variables kept for readability or debugging. |  |  |
| `code-review` | Review a pull request, branch, work-in-progress changes, or diff for concrete defects, unmet requirements, and repository convention violations. Keep findings scoped, evidenced, and proportionate. | `changeset-scope` skill | `effect-principles` skill,`session-coordination` skill,`testing` skill |
| `effect-principles` | Apply the Effect way of reasoning in codebases that do not use Effect, in any programming language. Use when editing or reviewing non-Effect code so dependencies, failures, state, boundaries, resources, time, and workflows stay explicit without adding Effect-shaped architecture or broader scope. |  | `changeset-scope` skill,`testing` skill |
| `evidence-first` | Check uncertain claims against the source that owns them before answering, while following clear user choices and limits. Use when the user's statement is tentative or possibly wrong (I think, I remember, I don't think, is this correct), when giving advice or a recommendation, or when the user sets a firm preference such as I don't want this, reduce the scope, or this is going too far. Load once per conversation; it stays in effect. | `research` skill | `asking-questions` skill |
| `git-context` | Patterns for working with git branches, remotes, diffs against the default branch, and rebases. Use when resolving rebase conflicts, continuing interactive rebases, amending commits, or any git operation that would open an interactive editor. |  |  |
| `github-development-rulesets` | Create GitHub Development rulesets from the bundled JSON baseline, compare and migrate existing rulesets, or update required CI checks. Use when setting up a Development ruleset, choosing among existing rulesets, or reconciling their policy and emitted check names. |  |  |
| `github-repository-setup` | Create GitHub repositories with the preferred settings, ask about licensing using GitHub templates, enable watching during creation or induction, offer CI and automerge workflows, and finish first-push setup with a Development ruleset. Use when creating or inducting a GitHub repository, using gh repo create or dot repo induct, applying repository defaults, or completing initial GitHub setup. |  | `github-development-rulesets` skill,`shared-workflows` skill |
| `handoff` | Save concise continuation context when work moves to another session or the user requests a handoff. |  |  |
| `herdr-workflows` | Apply local safeguards for Herdr session recovery and transferring linked-worktree changes back to a host checkout. Use alongside the herdr skill when diagnosing Herdr socket routing, recovering the default session, or moving, consolidating, or continuing Herdr worktree changes from the main or host checkout. The herdr skill remains authoritative for all Herdr CLI, topology, targeting, lifecycle, and safety behaviour. |  | `session-coordination` skill |
| `home-assistant-frontend` | Home Assistant frontend skill routing and personal engineering overlays. Use only when editing or reviewing code inside a Home Assistant frontend (home-assistant/frontend) checkout or a Home Assistant custom card or dashboard repository, so the frontend's `ha-frontend-*` skills stay authoritative and applicable Lit, TypeScript, cleanup, and HA companion skills are also loaded. | `home-assistant-lit-rendering` skill,`lit-rendering` skill | `testing` skill |
| `home-assistant-lit-rendering` | Home Assistant Lit rendering extensions for HA components and context-aware picker callback shape. Use with lit-rendering only when editing or reviewing Lit render logic inside a Home Assistant frontend (home-assistant/frontend) checkout or a Home Assistant custom card or dashboard repository. |  | `lit-rendering` skill |
| `human-step-guide` | Prepare a concise guide when progress is blocked by a genuinely human-only action. Use for approvals, physical actions, credential entry, or dashboard steps the agent cannot perform; do not use for work available tools can complete. |  |  |
| `import-external-skill` | Import skills from external repositories into this Agent Skills repository. Use when pulling in a public skill, reviewing an external skill set, or adapting upstream content into an existing skill. |  |  |
| `lit-rendering` | Lit rendering and picker callback-shape guidance. Use when editing or reviewing Lit component render logic, derived render data, memoization, or picker callbacks. |  |  |
| `maintain-docs` | Keep documentation current and accurate with recent code changes, across in-code docs (docstrings, annotations, comments), in-repo docs sites, and external docs repositories. Use when asked to update docs, check docs accuracy, keep documentation current, document recent changes, refresh docstrings or annotations, or catch documentation up with the codebase. Matches the codebase's existing documentation density and stops before commit. |  | `research` skill |
| `npm-jsr-first-publish` | Prepare a brand-new npm and JSR package for its first release from GitHub Actions. The bundled script publishes an npm 0.0.0 placeholder, adds the npm trusted publisher, and creates and links the JSR package. Use before a new package's first release, or when its release fails because the npm package doesn't exist, has no trusted publisher, or the JSR package isn't linked. |  |  |
| `opencode-effect` | Develop and migrate OpenCode V2 plugins, clients, SDK hosts, and HTTP API integrations. Use for the OpenCode plugin API, `@opencode/client`, `@opencode/sdk`, server API, Effect entrypoints, or V1-to-V2 API migration. |  |  |
| `pitchfork-dev-servers` | Manage long-running local dev servers by precedence - the project's own AGENTS.md workflow first, pitchfork next, then a framework-native background mode where a repo has no pitchfork config - and set them up to a standard with stable https://<daemon>.<project>.localhost addresses. Use when starting, stopping, restarting, checking, or tailing development servers, background servers, pitchfork MCP tools, or local AGENTS/mise tasks that mention pitchfork; when adding or changing a `pitchfork.toml`; or when a dev server's port clashes or its proxy URL doesn't resolve. |  |  |
| `pkexec-root` | Use pkexec first for commands that need root directly or indirectly. |  |  |
| `plan` | Produce implementation-ready plans from the current conversation and repository context. Use when entering native plan mode, invoking /plan, or when a task needs concrete implementation sequencing before edits begin; do not use for round-based grilling. | `staged-implementation` skill,`writing-style` skill | `session-coordination` skill,`testing` skill |
| `remove-single-use-functions` | Safe inlining and removal of single-use functions during code review and refactoring. Use when a local, non-exported helper has exactly one real call site and inlining preserves behaviour and readability. |  |  |
| `research` | Investigate a topic against primary sources and return cited findings, comparing credible maintainer and contributor perspectives when judgement is involved. Use when the user asks why, says show evidence, validate this, or use trusted sources; wants research, docs, API, or spec facts; needs external library or GitHub behaviour verified; compares competing views; or delegates reading legwork to a background agent. |  | `asking-questions` skill,`diagnose` skill |
| `safe-process-signals` | Safe process killing and signal handling for agent/subprocess contexts. Use when running pkill, killall, kill, or any process termination command from a shell subprocess, automated script, or coding agent. |  |  |
| `session-coordination` | Split independent work into visible Herdr sessions and choose models and effort variants for each assignment. Use when tasks benefit from parallel workers or different model capabilities, when the user requests coordination or model selection, or when a coordinator agent is selected. Ask before launching workers. | `changeset-scope` skill | `code-review` skill,`evidence-first` skill,`handoff` skill,`herdr-workflows` skill,`staged-implementation` skill |
| `shared-workflows` | Use, configure, maintain, or create reusable GitHub Actions workflows for personal and organisation repositories. Use when a task mentions shared workflows, reusable workflows, `workflow_call`, cross-repository workflow `uses:`, or the personal workflows repository; do not use for repository-specific or proof-of-concept CI unless evaluating whether it should be shared. |  |  |
| `skill-prefixes` | Name skills with an owner prefix so they do not collide with shared, imported or other repositories' skills. Use when creating, renaming, importing or reviewing a skill name, especially for repository-local or tool-owned skills. |  | `code-review` skill,`testing` skill,`writing-style` skill |
| `staged-implementation` | Execute broad changes one coherent, independently verifiable stage at a time. Use when work spans multiple independently reviewable changes, or when contracts, producer-consumer migrations, generated artefacts, or release packaging create an ordered multi-stage rollout; skip small single-purpose changes. | `session-coordination` skill | `handoff` skill,`testing` skill |
| `task-focus` | Keep the original task on track when the user raises a side thought, side question, tentative branch idea, or explicit change of task. Use before diverting work, switching branches, or choosing between a BTW session, a fresh session, and the current conversation, especially with a large context window. |  |  |
| `task-runners` | Run and write project tasks so checks, builds, tests and dev servers finish fast - run only what a change needs, in parallel, skipping work that is already up to date. Use when running linters, type checks, test suites or builds; when writing or reviewing task runner config (mise.toml, package.json scripts, justfile, deno.json, Makefile, pitchfork.toml) or CI steps; or when a check, build or task run is slow. |  | `testing` skill |
| `testing` | Choose tests for their concrete regression value, avoid low-value coverage, and verify changes by exercising the real built behaviour. Use during implementation, fixes, planning, diagnosis, and code review when choosing verification, adding or changing tests, considering a missing-test finding, or when the user asks to test, check, or verify a change. |  | `task-runners` skill |
| `types-enforce-ts` | TypeScript type-safety guidance. Use when editing or reviewing `.ts`, `.tsx`, `.mts`, or `.cts` files. |  |  |
| `writing-dot-skills` | Craft for authoring Agent Skills that select reliably and stay lean. Use when creating or revising a skill's description, workflow, references, scripts, or structure. |  |  |
| `writing-style` | Write commit messages, PR and issue text and comments, docs (README), code comments, and user-facing strings (notifications, UI labels, toasts, error messages) in the project owner's voice: concise, human, UK English, no em-dashes, no robotic or marketing tone. Use when writing, editing, or reviewing these, including requests to make writing sound natural or remove jargon. Keep the meaning and follow the repo's established writing style. |  |  |

### From External Sources

These skills were imported from other repos. Some are used as-is; others have been adapted for local workflows and conventions.

| Skill | Origin | Local Changes | Requires | Works with |
|---|---|---|---|---|
| `agentic-workflows` | [github/gh-aw](https://github.com/github/gh-aw/tree/main/.github/skills/agentic-workflows) | Yes |  |  |
| `bro` | [dmmulroy/skills](https://github.com/dmmulroy/skills/tree/main/bro) | Yes |  |  |
| `decision-models` | [ollaya-dev/ollaya](https://github.com/ollaya-dev/ollaya/tree/main/skills/ollaya-decisions) | Yes |  |  |
| `diagnose` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/diagnosing-bugs) | Yes |  | `browser-access` skill,`testing` skill |
| `gh-stack` | [github/gh-stack](https://github.com/github/gh-stack/tree/main/skills/gh-stack) | Yes | `git-context` skill |  |
| `grilling` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling) | Yes |  | `asking-questions` skill |
| `release-oxlint-rules` | [timmo001/oxlint-rules](https://github.com/timmo001/oxlint-rules/tree/main/skills/release-oxlint-rules) | Yes |  |  |
| `retro` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/retro) | Yes | `writing-dot-skills` skill | `code-review` skill |
| `show-me` | [humanlayer/skills](https://github.com/humanlayer/skills/tree/main/plugins/show-me/skills/show-me) | Yes |  |  |
| `to-questionnaire` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/to-questionnaire) | Yes |  |  |
| `wrangler` | [cloudflare/skills](https://github.com/cloudflare/skills/tree/main/skills/wrangler) | Yes |  |  |

## Agents

| Agent | Description |
|---|---|
| `build-ask` | Build agent that executes clear tasks and relies on permissions for write actions |
| `coordinator` | Manages low-context delegated sessions through delivery |
| `general-readonly` | General-style parallel subagent that researches with read-only tools and a narrow shell inspection allowlist (for delegation from read-only primaries). |
| `grill` | Read-only planning stress-test agent for light or full round-based grilling |
| `refactorer` | Refactor code while preserving behavior and following local command and skill workflows |
| `researcher-readonly` | Primary-source research subagent that compares claim-specific evidence and cannot delegate further |
| `researcher` | Interactive primary-source research agent that compares claim-specific evidence |
| `reviewer` | Reviews code for quality, bugs, security, and best practices |

## Commands

| Command | Description | Agent | Requires | Works with |
|---|---|---|---|---|
| `/code-review` | Review current work or a pull request with the code-review skill in the read-only reviewer agent | reviewer | `changeset-scope` skill,`effect-principles` skill |  |
| `/commit-push` | Split current changes into coherent commits and push | default |  |  |
| `/commit` | Split current changes into coherent commits via the dot git-commit gateway | default |  |  |
| `/grill` | Stress-test a plan, decision, or idea with light or full question rounds | grill | `grilling` skill |  |
| `/plan` | Manual entrypoint to native plan mode from the current conversation context | plan |  |  |
| `/research` | Research a topic from primary sources and compare evidence where judgement is involved | researcher |  |  |

## Plugins

| Plugin | Description |
|---|---|
| `commit-context` | Injects session-attributed commit scope into commit command prompts |
| `context-capture` | Opt-in capture of the assembled starter context for token profiling |
| `env-protection` | Blocks direct access to .env files, and any extra secret files listed in env-protection.yml, to prevent leaking secrets |
| `generated-artifact-guard` | Blocks direct mutation of generated dotfiles artefacts |
| `mcp-repo-gate` | Per-repo MCP server gating for OpenCode |
| `notes-guard` | Blocks direct file access to the repository notes vault |
| `notification` | Sends contextual desktop notifications and terminal attention for agent events |
| `pitchfork-dev-server-guard` | Enforces a project's declared pitchfork dev-server workflow for agents |
| `pr-reviews` | Polls the review threads on the pull request for the session's checkout. Threads only reach the model when the user sends them |
| `question-presentation-guard` | Rejects question tool calls that are not preceded by findings in chat |
| `readonly-subagent-shell-guard` | Rejects shell syntax that can turn read-only subagent commands into writes |
| `subagent-chrome-devtools-guard` | Blocks Chrome DevTools tools from delegated subagent sessions |

## Publishing

This repo is published automatically via GitHub Actions when the OpenCode config
[`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy-quattro/agents/.config/opencode) changes, and daily to
follow the latest [`timmo001/skills`](https://github.com/timmo001/skills) `main`.
