# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Power User is a Claude Code CLI **plugin marketplace** (`shaharia-lab`) containing three independently installable plugins. There are no build/test/lint commands — the project is entirely Markdown-based plugin definitions (agents, skills, reference docs).

## Architecture

### Marketplace Structure

`.claude-plugin/marketplace.json` is the central registry. It lists all plugins with their `source` paths, versions, and metadata. Each plugin must also have its own `.claude-plugin/plugin.json` — without it, the plugin won't be discoverable via `/plugin install`.

### Three Plugins

| Plugin | Path | Purpose |
|--------|------|---------|
| **cc-power-user** (v1.0.4) | `plugins/cc-power-user/` | 20 agents, 2 workflow commands, 7 skills for autonomous development |
| **code-navigator** (v1.0.0) | `plugins/code-navigator/` | Token-efficient code navigation via `codenav` CLI |
| **power-user-basic** (v1.0.0) | `plugins/power-user-basic/` | 5 project-agnostic skills (engineering, review, security, architecture, docs) |

### Plugin Anatomy

Each plugin directory contains:
- `.claude-plugin/plugin.json` — **Required.** Name, version, description, keywords.
- `agents/` — Agent definitions as `.md` files with YAML frontmatter (`name`, `description`, `model: inherit`).
- `skills/` — Each skill is a directory with a `SKILL.md` (YAML frontmatter + instructions) and optional reference files.
- `README.md` — Plugin-specific documentation.

### Key Relationships

- **Versions must stay in sync** between each plugin's `plugin.json` and its entry in the marketplace `marketplace.json`.
- Skills in `cc-power-user` are composable: `phase1-refinement` → `phase2-codegen` → `phase3-rl-loop` → `phase4-pr` form a pipeline.
- Agents are self-contained prompts — each `.md` file is a complete agent definition.

## Conventions

- **Commit messages**: Conventional commits (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`).
- **Agent format**: YAML frontmatter (`name`, `description`, `model`) followed by Markdown with Purpose, Guidelines, and Examples sections.
- **Skill format**: YAML frontmatter (`name`, `description`) in `SKILL.md`, with supporting reference files in the same directory.
- Agents and skills must be **project-agnostic** — no hardcoded paths, repos, or project-specific assumptions.

## Adding a New Plugin

1. Create `plugins/<name>/` with a `skills/` or `agents/` directory.
2. Create `plugins/<name>/.claude-plugin/plugin.json` with name, version, description.
3. Add the plugin entry to `.claude-plugin/marketplace.json` with matching version.
4. Add a `README.md` to the plugin directory.
