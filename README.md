# OpenCode Config

Shared [OpenCode](https://opencode.ai) skills, agents, plugins, and commands.

Published from [`timmo001/dotfiles`](https://github.com/timmo001/dotfiles) — source at [`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.config/opencode).

## Installation

Clone the repo and copy what you need into your OpenCode config directory:

```bash
git clone https://github.com/timmo001/opencode-config.git
cd opencode-config

# Copy individual items
cp -r skills/diagnose ~/.config/opencode/skills/
cp commands/git-workflow.md ~/.config/opencode/commands/
cp plugins/env-protection.js ~/.config/opencode/plugins/
cp agents/reviewer.md ~/.config/opencode/agents/

# Or copy everything
cp -r skills agents commands plugins ~/.config/opencode/
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
| `check-skill-updates` | Check imported skills for upstream changes and apply updates. Use when reviewing whether externally imported skills have new upstream content, or when `dot skill-updates` reports available changes. |  | `import-external-skill` skill |
| `cleanup-unnecessary-variables` | Safe unnecessary-variable cleanup guidance for code review and refactoring. |  |  |
| `dotfiles-stow` | REQUIRED when changing configs managed by ~/.config/dotfiles or ~/.config/dotfiles-private. Enforces editing stow source paths (not ad-hoc live paths) and using the dot command for stow/update/validation workflows. |  |  |
| `git-workflow` | Patterns for working with git branches, remotes, and diffs against the default branch | `branch-context` plugin |  |
| `import-external-skill` | Import skills from external repos into the local dotfiles skill library. Use when pulling in a skill from a public repo, reviewing an external skill set for useful additions, or adapting external skill content into existing local skills. |  |  |
| `pkexec-root` | Use pkexec first for commands that need root directly or indirectly. |  |  |
| `pr-review` | Guidelines for reviewing pull requests - what to analyze, review etiquette, and output formatting |  |  |
| `remove-single-use-functions` | Safe single-use function removal guidance for code review and refactoring. |  |  |
| `types-enforce-ts` | TypeScript type-safety guidance for editing and reviewing `.ts`, `.tsx`, `.mts`, and `.cts` files. |  | `fallow` skill |

### From External Sources

These skills were imported from other repos. Some are used as-is; others have been adapted for local workflows and conventions.

| Skill | Origin | Local Changes | Requires | Works with |
|---|---|---|---|---|
| `ask-questions-if-underspecified` | [trailofbits/skills](https://github.com/trailofbits/skills/tree/main/plugins/ask-questions-if-underspecified/skills/ask-questions-if-underspecified) | Yes |  |  |
| `css-motion-systems` | [stolinski/s-stack](https://github.com/stolinski/s-stack/tree/main/skills/css-motion-systems) | Yes |  |  |
| `diagnose` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/diagnose) | Yes |  |  |
| `effect` | [anomalyco/opencode](https://github.com/anomalyco/opencode/tree/dev/.opencode/skills/effect) | No |  |  |
| `fallow` | [fallow-rs/fallow-skills](https://github.com/fallow-rs/fallow-skills/tree/main/fallow/skills/fallow) | No |  |  |
| `handoff` | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/productivity/handoff) | No |  |  |
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
| `general-readonly` | General-style parallel subagent that researches and runs commands but cannot modify workspace files via file tools (for delegation from read-only primaries). |
| `refactorer` | Refactor code while preserving behavior and following local command and skill workflows |
| `reviewer` | Reviews code for quality, bugs, security, and best practices |

## Commands

| Command | Description | Agent | Requires | Works with |
|---|---|---|---|---|
| `/check-skill-updates` | Check imported skills for upstream updates | default |  | `import-external-skill` skill |
| `/cleanup-unnecessary-variables` | Inline and remove unnecessary variables from current git scope | refactorer | `branch-context` plugin |  |
| `/debug-frontend` | Debug browser-specific UI issues with Chrome DevTools tools first | ask |  |  |
| `/explore-codebase` | Explore a codebase topic with the task explore subagent | ask |  |  |
| `/fallow-audit` | Audit changed JavaScript or TypeScript code with Fallow | ask | `fallow` skill |  |
| `/fallow-project-analyse` | Analyse a JavaScript or TypeScript project with Fallow | ask | `fallow` skill |  |
| `/git-workflow` | Read branch, diff, and PR context from BranchContextPlugin without extra git calls | ask | `branch-context` plugin |  |
| `/handoff` | Write a handoff document for the next agent session | default |  |  |
| `/import-external-skill` | Import or review external skills for the local skill library | default |  |  |
| `/improve-codebase-architecture` | Review a codebase area for architectural friction and focused structural improvements | ask |  |  |
| `/investigate` | Investigate a topic, issue, or area without editing by default | ask |  | `diagnose` skill |
| `/plan` | Manual entrypoint to native plan mode from the current conversation context | plan |  |  |
| `/refactor-current-work` | Refactor current branch work while preserving behaviour | refactorer | `branch-context` plugin |  |
| `/remove-single-use-functions` | Inline and remove safe single-use functions from current git scope | refactorer | `branch-context` plugin |  |
| `/reset-branch-reapply` | Reset branch to default and reapply current diff staged | build | `branch-context` plugin |  |
| `/review-current-work` | Review current branch work with BranchContextPlugin context | reviewer | `branch-context` plugin,`pr-review` skill |  |
| `/types-enforce-ts` | Enforce TypeScript type safety in current git scope | refactorer | `branch-context` plugin |  |

## Plugins

| Plugin | Description |
|---|---|
| `branch-context` | Injects branch-context blocks into command prompts before execution |
| `env-protection` | Blocks reads of .env files to prevent leaking secrets |
| `notification` | Plays a desktop notification sound when agent tasks complete |
| `readonly-subagent-task-guard` | Forces read-only primary agents to delegate only to non-modifying subagents |

## Publishing

This repo is published automatically via GitHub Actions when the source
[`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.config/opencode) changes.
