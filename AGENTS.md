# AGENTS

Instructions for coding agents working in this repository.

## Source

This repo is generated from the OpenCode configuration in [`timmo001/dotfiles`](https://github.com/timmo001/dotfiles).

OpenCode config source: [`agents/.config/opencode/`](https://github.com/timmo001/dotfiles/tree/distro/arch-omarchy/agents/.config/opencode)

Shared skills source: [`timmo001/skills`](https://github.com/timmo001/skills)

Do not edit generated files here directly. Make OpenCode config changes in [`timmo001/dotfiles`](https://github.com/timmo001/dotfiles) and skill changes in [`timmo001/skills`](https://github.com/timmo001/skills).

## Structure

```
skills/      OpenCode skills (SKILL.md per directory, optional references/)
agents/      Agent definitions (YAML frontmatter + Markdown body)
commands/    Slash commands (YAML frontmatter + Markdown workflow)
plugins/     Lifecycle plugins (ESM TypeScript)
lib/         Shared modules imported by plugins
```

## Skills

Each skill is a directory containing a `SKILL.md` with YAML frontmatter (`name`, `description`) and a Markdown body with checklists and guidance. Some skills include a `references/` subdirectory with supporting documents.

Imported skills include `# origin:` and `# upstream-sha:` comments in their frontmatter for tracking upstream changes.

## Importing

To import a skill into your own OpenCode setup, use the `import-external-skill` workflow with a GitHub tree URL pointing at the skill directory:

```
https://github.com/timmo001/skills/tree/main/<skill-name>
```

Agents, commands, and plugins can be copied directly into your OpenCode config directory.
