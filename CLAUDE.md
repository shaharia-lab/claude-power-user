# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Power User is a Claude Code CLI **plugin marketplace** (`shaharia-lab`) containing three independently installable plugins. There are no build/test/lint commands — the project is entirely Markdown-based plugin definitions (agents, skills, reference docs) plus JSON manifests. Changes are validated by installing the plugin via `/plugin install` and exercising its skills/agents in Claude Code.

## Architecture

### Marketplace Structure

`.claude-plugin/marketplace.json` is the central registry. It declares the marketplace `name`, owner, and a `plugins[]` array — each entry pointing at a local plugin via `source: "./plugins/<name>"` with its `version`, `description`, `keywords`, `category`, and `tags`.

Every plugin must **also** carry its own `.claude-plugin/plugin.json` (the per-plugin manifest). Without it, `/plugin install <name>@shaharia-lab` cannot discover the plugin even if the marketplace entry exists.

### Three Plugins

| Plugin | Path | Version | Purpose |
|--------|------|---------|---------|
| **cc-power-user** | `plugins/cc-power-user/` | 2.0.0 | Autonomous GitHub issue-to-PR pipeline: 20 agents + 9 composable skills |
| **code-navigator** | `plugins/code-navigator/` | 1.0.0 | Token-efficient code navigation via the external `codenav` CLI (87–92% token savings) |
| **power-user-basic** | `plugins/power-user-basic/` | 1.0.0 | 5 project-agnostic skills (engineering, context-updater, pr-reviewer, architect-reviewer, security-reviewer) |

Marketplace metadata version is tracked separately in `.claude-plugin/marketplace.json` under `metadata.version` (currently 1.1.0) — bump this when adding or removing plugins.

### Plugin Anatomy

Each plugin directory contains:
- `.claude-plugin/plugin.json` — **Required.** `name`, `version`, `description`, `author`, `homepage`, `repository`, `license`, `keywords`.
- `skills/<skill-name>/SKILL.md` — Skill definition. YAML frontmatter (`name`, `description`, optional `argument-hint`, optional `disable-model-invocation: true` for command-only skills) followed by instructions. Supporting reference files live in the same directory.
- `agents/...` — Agent definitions as `.md` files with YAML frontmatter (`name`, `description`, `model: inherit`) and full prompt body. In `cc-power-user` these are grouped under `agents/engineering/`.
- `README.md` — Plugin-specific docs.
- `examples/` — Optional sample outputs (cc-power-user only).

### cc-power-user Pipeline

The headline feature is the autonomous issue-to-PR workflow. Two top-level skills are exposed as namespaced commands (`disable-model-invocation: true` keeps them user-triggered):

- `/cc-power-user:github-issue-refiner <issue>` — turns a vague issue into a WHAT/WHY/HOW spec, edits the issue body, optionally syncs to GitHub Projects V2 with status `Ready`.
- `/cc-power-user:github-issue-to-pr <issue>` — runs the full pipeline.

The pipeline composes four phase skills in strict order — never skip, never reorder:

1. **phase1-refinement** — verify the issue is refined; refuse to proceed otherwise.
2. **phase2-codegen** — delegate to stack-specific developer sub-agents (backend, frontend, cli, website, docs-site) for implementation + tests. Sub-agents return a structured handoff; they never commit.
3. **phase3-rl-loop** — `quality-gates` + `pr-review-ci` run lint/tests/security/architecture/bug-finder; auto-remediate via specialist agents. Max 10 iterations, no human in the loop.
4. **phase4-pr** — open a draft PR, wait for CI, run automated `pr-reviewer`, apply critical/high feedback, re-verify, then mark ready.

Supporting skills: `github-project-integration` (Projects V2 GraphQL), `quality-gates`, `pr-review-ci`.

The 20 agents under `agents/engineering/` split into:
- **Developers** (8): `backend-developer`, `frontend-developer`, `cli-developer`, `website-developer`, `docs-site-developer`, plus research agents `backend`, `frontend`, `docs-writer`.
- **QA** (5): `security-guardian`, `architecture-guardian`, `bug-finder`, `pr-reviewer`, `production-error-analyzer`.
- **Context collectors & automation** (7): `*-context-collector` for each stack, plus `golang-lint-fixer`, `openapi-schema-manager`.

All agents are namespaced as `/cc-power-user:<agent-name>` when invoked directly.

### power-user-basic vs cc-power-user

`power-user-basic` skills are intentionally project-agnostic and are exposed **without** a `cc-power-user:` namespace — they invoke as `/engineering`, `/pr-reviewer`, `/architect-reviewer`, `/security-reviewer`, `/context-updater`. They read the user's own docs/code rather than assuming a stack. Don't add stack-specific assumptions to these skills.

`cc-power-user` agents bake in an opinionated stack (Go 1.22, Chi, Wire, PostgreSQL 15, React 19 + Vite 6 + Tailwind v4, Bun + Commander.js, Docusaurus 3). Users are expected to customize agent files to match their own stack.

### code-navigator

A single skill (`codenav-navigation`) that teaches Claude to drive the external `codenav` CLI for indexed call-graph queries. Requires `codenav` on PATH and a pre-built index — the plugin itself ships only instructions.

## Conventions

- **Commit messages**: Conventional commits (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`). Use `feat!:` / `BREAKING CHANGE:` for breaking plugin changes (e.g. the v2.0.0 issue-to-PR rename).
- **Version sync**: A plugin's `version` in its `plugin.json` MUST match its entry in `.claude-plugin/marketplace.json`. CI does not enforce this — check by hand.
- **Skill frontmatter**: `name` (kebab-case, matches directory), `description` (used for auto-invocation matching — write it as a trigger sentence), optional `argument-hint`, optional `disable-model-invocation: true` for user-only commands.
- **Agent frontmatter**: `name`, `description` (include concrete invocation examples so Claude recognizes when to use it), `model: inherit`.
- **Reference files**: Long checklists / patterns live as sibling files in the skill directory (e.g. `pr-reviewer/security-checks.md`), not inline in `SKILL.md`.
- **Project-agnostic language** in `power-user-basic` and any skills meant for general use: no hardcoded paths, repos, org names, or stack assumptions. Use placeholders like `<your-org>`, `<your-repo>`, `[ISSUE_URL_PLACEHOLDER]`.

## Adding or Modifying Plugins

**New plugin:**
1. Create `plugins/<name>/` with `skills/` and/or `agents/` subdirectories.
2. Create `plugins/<name>/.claude-plugin/plugin.json` with name, version (start at `1.0.0`), description, author, homepage, repository, license, keywords.
3. Append a matching entry to `.claude-plugin/marketplace.json` (`name`, `source: "./plugins/<name>"`, version, description, author, homepage, repository, license, keywords, `category`, `tags`).
4. Bump `metadata.version` in `marketplace.json`.
5. Add `plugins/<name>/README.md` and link it from the top-level `README.md`.

**Modifying an existing plugin:**
1. Edit the skill/agent `.md` files.
2. Bump `version` in **both** the plugin's `plugin.json` and its `marketplace.json` entry (patch for fixes, minor for new skills/agents, major for breaking renames or removals).
3. Update the plugin's `README.md` if the user-facing surface changed.
4. Test by reinstalling: `/plugin marketplace update shaharia-lab` then `/plugin install <name>@shaharia-lab`.

## What Not to Do

- Don't add build tooling, package.json, or CI for this repo — it is pure Markdown/JSON content and adding tooling raises the bar for contributors without buying anything.
- Don't put project-specific paths (e.g. `$HOME/Projects/your-project/...`) into `power-user-basic` skills — that plugin must stay stack-agnostic.
- Don't remove `disable-model-invocation: true` from `github-issue-to-pr` or `github-issue-refiner`. They are destructive (open PRs, mutate issues) and must stay user-triggered.
- Don't bump a plugin's `version` in only one of the two manifests.
