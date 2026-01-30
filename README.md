# Shaharia Lab - Claude Code Plugins Marketplace

Comprehensive AI agent systems and development workflows for Claude Code CLI.

## Installation

Add the marketplace:

```bash
/plugin marketplace add https://github.com/shaharia-lab/claude-power-user.git
```

Install plugins:

```bash
/plugin install cc-power-user@shaharia-lab
```

## Available Plugins

### CC Power User (v1.0.0)

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

## Usage

After installation, use the plugin commands (namespace is automatically `cc-power-user:`):

```bash
# Workflow commands
/cc-power-user:issue-refine <issue-number>
/cc-power-user:develop-issue <issue-number>

# Direct agent usage
/cc-power-user:backend-developer <task>
/cc-power-user:frontend-developer <task>
/cc-power-user:security-guardian <task>
```

## Future Plugins

More plugins coming soon:
- Security Auditor - Comprehensive security scanning
- Deployment Tools - Infrastructure automation
- Performance Analyzer - Performance profiling

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
**Version**: 1.0.0
