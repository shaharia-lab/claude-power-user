# GitHub Maintenance

Repo-agnostic maintenance skills for keeping repositories healthy over time. Designed to be run **periodically** — on a cron, from CI, or by hand on a slow Friday afternoon.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| **agent-context-sync** | `/github-maintenance:agent-context-sync` | Keep AI-context files (CLAUDE.md, AGENTS.md, `.cursorrules`, `.github/copilot-instructions.md`) accurate, lean, and useful |

## Installation

```bash
/plugin marketplace add https://github.com/shaharia-lab/claude-power-user.git
/plugin install github-maintenance@shaharia-lab
```

## agent-context-sync

A periodic doctor for the AI-context files in your repository. Verifies every concrete claim against the current repo, lints the file against an opinionated quality spec, and proposes the minimum edits needed to bring it back to health. **No-op when the file is already healthy** — safe to run on a weekly cron without producing churn.

### What it does

1. **Discovers** which AI-context files exist (CLAUDE.md, AGENTS.md, `.cursorrules`, `.github/copilot-instructions.md`).
2. **Verifies** every extractable claim — file paths, version numbers, build commands, agent/skill counts, "lives at X" references — against the current repository.
3. **Lints** against [`quality-spec.md`](skills/agent-context-sync/quality-spec.md): required sections, length budget, anti-patterns (directory catalogues, human-onboarding fluff, content redundant with manifest files).
4. **Augments** by examining `git log` since each file's last modification — surfaces new subsystems, commands, or conventions that the doc never mentions.
5. **Outputs** either a structured report (dry-run) or a pull request with the edits + report (default).

### Usage

```bash
# Report-only: print findings to stdout, no edits, no branch, no PR.
/github-maintenance:agent-context-sync --dry-run

# Default: edit files in place, create a branch, open a PR with the report as the body.
/github-maintenance:agent-context-sync
```

### Pruning policy

**Conservative.** The skill only deletes claims it can programmatically prove are wrong or stale (failed path, mismatched version, miscount, missing command). Suspected bloat, anti-patterns, and unverifiable claims are **flagged in the report** but left in place — humans decide whether to remove them.

### No-op when healthy

If every claim verifies, every lint check passes, and the recent git history surfaces no missing content, the skill prints:

```
[agent-context-sync] healthy — no changes proposed.
```

…and exits without writing, branching, or opening a PR. This is what makes periodic execution sane.

### Multi-file handling

When multiple AI-context files exist:
- Each is verified independently against the repo.
- Cross-file divergence (e.g. CLAUDE.md says "Go 1.22", `.cursorrules` says "Go 1.20") is flagged but **not auto-mirrored** — the formats and audiences differ enough that automatic mirroring is unsafe.
- The canonical file is CLAUDE.md if present; otherwise the most recently updated file.

### Recommended cadence

- **Weekly** for repositories under active development.
- **Monthly** for stable / mature repositories.
- **Before any major release** as a final sanity check.

Wire it up via your preferred scheduler (GitHub Actions cron, Renovate-style bot, local cron). The skill itself is invocation-agnostic — it does its job and exits with a clean report.

## Future skills

Planned siblings under this plugin:
- **stale-issue-pruner** — close abandoned issues with a polite template
- **dependabot-triager** — batch-review and merge low-risk Dependabot PRs
- **readme-doctor** — separate, human-audience version of agent-context-sync for README.md

## License

MIT — see [LICENSE](../../LICENSE).
