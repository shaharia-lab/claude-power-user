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
/cc-power-user:issue-refine 123
/cc-power-user:develop-issue 456
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

## Usage

After installation, use the plugin commands:

```bash
# CC Power User - workflow commands
/cc-power-user:issue-refine <issue-number>
/cc-power-user:develop-issue <issue-number>

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
**Version**: 1.1.0
