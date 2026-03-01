# Power User Basic Plugin

A set of 5 essential skills for everyday software development with Claude Code. Works with any project, any language, any framework.

## Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| **engineering** | `/engineering` | Implement features, fix bugs, refactor code |
| **context-updater** | `/context-updater` | Keep docs and AI context files up to date |
| **pr-reviewer** | `/pr-reviewer` | Review pull requests before merge |
| **architect-reviewer** | `/architect-reviewer` | Audit architecture, patterns, and technical debt |
| **security-reviewer** | `/security-reviewer` | Find vulnerabilities and security misconfigurations |

## Installation

```bash
/plugin install power-user-basic@shaharia-lab
```

## Usage

### Engineering
Implement any development task. The agent reads your project's docs and existing code before writing anything.

```
/engineering "add pagination to the list users endpoint"
/engineering "fix the authentication bug in the login flow"
/engineering "refactor the storage layer to use interfaces"
```

### Context Updater
Keep `CLAUDE.md`, `README.md`, and `docs/` accurate after code changes.

```
/context-updater "since last 7 days"
/context-updater "since last 3 days"
/context-updater "check all docs"
```

### PR Reviewer
Thorough pull request review across 11 dimensions (correctness, security, performance, architecture, UI/UX, observability, etc.).

```
/pr-reviewer 42
/pr-reviewer feature/auth-flow
/pr-reviewer "review current branch against main"
```

### Architect Reviewer
Honest architecture audit with actionable findings, impact assessment, and effort estimates.

```
/architect-reviewer "audit the codebase"
/architect-reviewer "review last 7 days changes"
/architect-reviewer "review the API layer"
```

### Security Reviewer
Security audit producing triaged findings with CWE references, attack scenarios, and remediation guidance.

```
/security-reviewer "audit the codebase"
/security-reviewer "review last 7 days changes"
/security-reviewer "audit the authentication flow"
```

## What Makes These Skills Different

- **No project assumptions** — skills work by reading your project's own docs and code, not by assuming your stack
- **Opinionated output** — structured reports with severity, location, and concrete fixes
- **Proportional depth** — small changes get light review, large changes get deep review
- **No nit-picking** — reviewers focus on real problems, not formatting or style

## Supporting Reference Files

Each reviewer skill includes reference checklists and patterns:

- `pr-reviewer/security-checks.md` — input handling, auth, data protection
- `pr-reviewer/architecture-checks.md` — layer violations, module boundaries, testability
- `pr-reviewer/ui-ux-checks.md` — accessibility, responsive design, states
- `architect-reviewer/code-smells.md` — function, class, and module level smells
- `architect-reviewer/architecture-patterns.md` — SOLID, patterns, anti-patterns
- `architect-reviewer/checklist.md` — 100-point comprehensive audit checklist
- `security-reviewer/owasp-checklist.md` — OWASP Top 10 (2021) mapped checks
- `security-reviewer/vulnerability-patterns.md` — Go and TypeScript/JS code examples
- `security-reviewer/secure-defaults.md` — secure configuration checklist
