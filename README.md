# Shaharia Lab - Claude Code Plugins Marketplace

Comprehensive AI agent systems and development workflows for Claude Code CLI.

## Installation

Add the marketplace:

```bash
/plugin marketplace add https://github.com/shaharia-lab/claude-power-user.git
```

Install plugins:

```bash
# Install CC Power User
/plugin install cc-power-user@shaharia-lab

# Install Power User Basic
/plugin install power-user-basic@shaharia-lab

# Install Code Navigator
/plugin install code-navigator@shaharia-lab

# Install GitHub Maintenance
/plugin install github-maintenance@shaharia-lab
```

## Available Plugins

### Code Navigator (v1.0.0)

Efficient code navigation using codenav CLI. Save up to 92% on token usage by navigating codebases intelligently.

**Features**:
- Codenav-based navigation skill
- Token-efficient code exploration
- Call graph analysis
- Dependency tracing
- Smart function finding

**Benefits**:
- 87-92% token savings vs blind file reading
- Faster code understanding
- Precise navigation to relevant code
- Architectural insights without reading files

**Usage**:
```bash
# Automatic - just ask questions
"How does authentication work?"
"Find where data comes from"

# Manual invocation
/codenav-navigation functionName
```

**Requires**: `codenav` CLI tool and indexed codebase

[View plugin documentation →](./plugins/code-navigator/README.md)

---

### CC Power User (v1.0.4)

20 specialized agents, 2 workflow commands, and 7 reusable skills for autonomous development.

**Features**:
- 8 development agents (backend, frontend, CLI, website, docs)
- 5 QA agents (security, architecture, bug finder, PR review, error analysis)
- 7 context & automation agents
- 7 reusable workflow skills
- 2 workflow commands (issue refinement, full development cycle)

**Tech stack**: Go, React/TypeScript, Node.js CLI, Docusaurus

**Commands**:
```bash
/cc-power-user:github-issue-refiner 123
/cc-power-user:github-issue-to-pr 456
/cc-power-user:backend-developer Add API endpoint
/cc-power-user:security-guardian Audit payment module
```

[View plugin documentation →](./plugins/cc-power-user/README.md)

---

### Power User Basic (v1.0.0)

5 essential, project-agnostic skills for everyday development — works with any project, language, or framework.

**Skills**:
- `/engineering` — Implement features, fix bugs, refactor code
- `/context-updater` — Keep CLAUDE.md, README.md, and docs accurate after changes
- `/pr-reviewer` — 11-dimension code review (correctness, security, performance, architecture, UI/UX, observability)
- `/architect-reviewer` — Architecture audit with 100-point checklist and code smell detection
- `/security-reviewer` — Security audit with CWE references, OWASP checks, and remediation guidance

**Usage**:
```bash
/engineering "fix the authentication bug"
/pr-reviewer 42
/security-reviewer
/architect-reviewer
/context-updater
```

[View plugin documentation →](./plugins/power-user-basic/README.md)

---

### GitHub Maintenance (v1.0.0)

Repo-agnostic maintenance skills designed to be run **periodically** — on a cron, from CI, or by hand. Ships `agent-context-sync`: a periodic doctor that keeps AI-context files (`CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `.github/copilot-instructions.md`) accurate, lean, and useful.

**Skills**:
- `/github-maintenance:agent-context-sync` — Verify every concrete claim in your AI-context files against the repo, lint against an opinionated quality spec, surface newly-added subsystems the docs miss, and open a PR (or print a report) with the minimum edits to restore health.

**Highlights**:
- **No-op when healthy** — safe to run weekly on a cron without producing churn.
- **Conservative pruning** — only deletes claims it can programmatically prove are wrong; flags bloat without removing.
- **Multi-file** — handles CLAUDE.md, AGENTS.md, `.cursorrules`, and `.github/copilot-instructions.md` together; reports cross-file divergence.
- **Two modes** — `--dry-run` for report-only, default for branch + PR.

**Usage**:
```bash
# Print a structured health report, no edits.
/github-maintenance:agent-context-sync --dry-run

# Apply edits, create a branch, open a PR.
/github-maintenance:agent-context-sync
```

[View plugin documentation →](./plugins/github-maintenance/README.md)

## Usage

After installation, use the plugin commands:

```bash
# CC Power User - workflow commands
/cc-power-user:github-issue-refiner <issue-number>
/cc-power-user:github-issue-to-pr <issue-number>

# CC Power User - direct agent usage
/cc-power-user:backend-developer <task>
/cc-power-user:frontend-developer <task>
/cc-power-user:security-guardian <task>

# Power User Basic - project-agnostic skills
/engineering <task>
/pr-reviewer <pr-number>
/security-reviewer
/architect-reviewer
/context-updater

# GitHub Maintenance - periodic repo upkeep
/github-maintenance:agent-context-sync --dry-run
/github-maintenance:agent-context-sync
```

## Future Plugins

More plugins coming soon:
- Security Auditor - Comprehensive security scanning and vulnerability detection
- Deployment Tools - Infrastructure automation and deployment workflows
- Performance Analyzer - Performance profiling and optimization recommendations
- Test Automation - Automated test generation and execution

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on contributing plugins.

## License

MIT - See [LICENSE](LICENSE) file for details.

## Support

- Issues: [GitHub Issues](https://github.com/shaharia-lab/claude-power-user/issues)
- Documentation: See individual plugin READMEs in `plugins/` directory

---

**Repository**: https://github.com/shaharia-lab/claude-power-user
**Marketplace**: `shaharia-lab`
**Version**: 1.2.0
