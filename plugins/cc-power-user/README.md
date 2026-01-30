# Claude Power User

AI agent system for autonomous development with Claude Code CLI. Includes 20 specialized agents, 2 workflow commands, and 7 reusable skills for backend, frontend, security, and quality assurance.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Installation

```bash
# Clone into Claude plugins directory
git clone https://github.com/shaharia-lab/claude-power-user.git ~/.claude/plugins/claude-power-user

# Or clone anywhere and symlink
git clone https://github.com/shaharia-lab/claude-power-user.git ~/Projects/claude-power-user
ln -s ~/Projects/claude-power-user ~/.claude/plugins/claude-power-user
```

Restart Claude Code CLI or restart your terminal.

## Configuration

Set environment variables for your project (optional, only if using GitHub Projects integration):

```bash
# In your shell profile (~/.bashrc, ~/.zshrc, etc.)
export GITHUB_ORG=your-org
export GITHUB_REPO=your-repo
export PROJECT_ROOT=$HOME/Projects/your-repo
```

For GitHub Projects V2 integration (optional), get your Project ID:

```bash
gh api graphql -f query='
  query {
    organization(login: "your-org") {
      projectV2(number: 1) {
        id
        fields(first: 20) { nodes { id name } }
      }
    }
  }
'
```

Add to your shell:
```bash
export GITHUB_PROJECT_ID=PVT_xxx
export STATUS_FIELD_ID=PVTSSF_xxx
```

## Usage

### Workflow Commands

All commands are automatically namespaced with `cc-power-user:` to avoid conflicts:

**Issue Refinement**: Transform vague issues into structured WHAT/WHY/HOW specifications
```bash
/cc-power-user:issue-refine 123
```

**Full Development**: Autonomous implementation from refined issue to PR
```bash
/cc-power-user:develop-issue 456
```

### Direct Agent Usage

Agents are also namespaced. Use them directly in Claude Code CLI:
```bash
/cc-power-user:backend-developer Add user authentication API
/cc-power-user:frontend-developer Build dashboard component
/cc-power-user:security-guardian Audit payment module
```

## What's Included

### Development Agents (8 agents)

Autonomous developers that implement features and fix bugs following established patterns:

- **backend-developer**: Go backend development with PostgreSQL, Chi router, Wire DI. Implements APIs, writes table-driven tests, creates migrations, follows clean architecture.
- **frontend-developer**: React/TypeScript development with Tailwind CSS. Builds components, manages state, writes tests (Jest + Playwright), ensures accessibility.
- **cli-developer**: TypeScript/Bun CLI development with Commander.js. Builds commands, argument parsing, interactive prompts, help generation.
- **website-developer**: Marketing website development with React. Landing pages, SEO optimization, responsive design, performance optimization.
- **docs-site-developer**: Docusaurus documentation management. Creates docs, organizes structure, API references, tutorials, version management.
- **backend**: Backend architecture research and analysis. Maps APIs, understands database schemas, identifies patterns, documents decisions.
- **frontend**: Frontend architecture analysis. Component structure, state management, routing patterns, UI patterns, relationships.
- **docs-writer**: Technical documentation writing. API docs, tutorials, architecture docs, ADRs, README files.

### Quality Assurance Agents (5 agents)

Security, architecture, and quality review specialists:

- **security-guardian**: Security vulnerability analysis. OWASP Top 10 scanning, auth/authz review, injection detection, crypto review, compliance validation.
- **architecture-guardian**: Design pattern and consistency review. Clean architecture compliance, SOLID principles, anti-patterns detection, dependency validation, maintainability.
- **bug-finder**: Systematic bug detection. Edge cases, error handling gaps, race conditions, null pointer risks, boundary conditions.
- **pr-reviewer**: Comprehensive code review. Quality checks, test coverage analysis, documentation review, performance, security, breaking changes.
- **production-error-analyzer**: Root cause analysis from logs. Parse stack traces, identify causes, suggest fixes, recommend monitoring, create reproduction steps.

### Context & Automation (7 agents)

Generate guidelines and automate specialized tasks:

- **backend-context-collector**: Generate comprehensive developer guidelines for backend service. Outputs architecture overview, directory structure, design patterns, database conventions, development standards to docs/developer-guidelines/backend-api/
- **frontend-context-collector**: Generate developer guidelines for frontend. Component patterns, state management, styling conventions, testing patterns to docs/developer-guidelines/frontend/
- **cli-context-collector**: Generate CLI development guidelines. Command structure, argument patterns, configuration management, testing strategies.
- **website-context-collector**: Generate website development guidelines. Page structure, SEO patterns, component library, deployment process.
- **docs-site-context-collector**: Generate documentation site guidelines. Doc structure, writing style, version management, publishing process.
- **golang-lint-fixer**: Bulk fix golangci-lint issues. Automatic fixes, preserves semantics, handles bulk operations.
- **openapi-schema-manager**: Manage OpenAPI specifications. Update specs, validate changes, generate TypeScript types, document endpoints.

### Skills (7 skills)

Reusable workflow components:

- **phase1_refinement**: Transform issues into structured WHAT/WHY/HOW specifications. Analyzes context, creates scope, explains business value, provides technical approach.
- **phase2_codegen**: Autonomous code generation following established patterns. Reads guidelines, analyzes existing code, implements with tests, updates documentation.
- **phase3_rl_loop**: Quality gates with auto-remediation (max 10 iterations). Runs linting, tests, security scans, architecture review, bug analysis. Auto-fixes issues via specialized agents.
- **phase4_pr**: Professional PR creation and handoff. Generates comprehensive description, creates PR, updates issue status, notifies reviewers.
- **github-project-integration**: Manage GitHub Projects V2 integration. Add issues to boards, update statuses, assign iterations, set fields, link PRs.
- **quality-gates**: Standard quality checks. Linting, unit tests, integration tests, E2E tests, security scans, architecture review, bug analysis.
- **pr-review-ci**: Parse CI/CD feedback and auto-remediate. Handles GitHub Actions, CircleCI, GitLab CI. Auto-fixes linting, analyzes test failures, triggers re-runs.

## Tech Stack Support

Out of box:
- **Backend**: Go 1.22, PostgreSQL 15, Chi router, Wire DI, golang-migrate
- **Frontend**: React 19, TypeScript, Vite 6, Tailwind CSS v4
- **CLI**: TypeScript, Bun 1.0, Commander.js
- **Docs**: Docusaurus 3.x
- **Testing**: Table-driven tests (Go), Jest, Playwright
- **Tools**: golangci-lint, ESLint, Prettier

Easily customizable for Node.js, Python, Vue, Java, or other stacks - just update agent instructions.

## How It Works

**Issue Refinement Workflow** (`/cc-power-user:issue-refine`):
1. Fetches GitHub issue
2. Analyzes context and delegates to appropriate analyzer agents
3. Creates WHAT (scope + acceptance criteria), WHY (business value), HOW (technical approach)
4. Updates issue body with refinement
5. Adds to GitHub Project board, sets status to "Ready"

**Development Workflow** (`/cc-power-user:develop-issue`):
1. Validates issue is refined and status is "Ready"
2. Creates git worktree, updates status to "In Progress"
3. Delegates to developer agents (backend/frontend/cli) for implementation with tests
4. Runs quality gates loop (linting, tests, security, architecture review, bug analysis)
5. Auto-remediates issues via specialized agents (max 10 iterations)
6. Creates comprehensive PR with description, links, and metadata
7. Updates status to "In Review", assigns reviewer

## Requirements

- Claude Code CLI v1.0.33+
- GitHub CLI (`gh`) authenticated
- Git configured
- Language runtimes for your stack (Go 1.22+ / Node 18+ / Python 3.8+)

## Customization

Update agent files in `agents/engineering/` to match your tech stack, patterns, and conventions. Agents automatically read `CLAUDE.md` from your project root if present.

## Examples

See `examples/generated-output/` for sample issue refinements and PR descriptions.

## License

MIT License - see [LICENSE](LICENSE)

## Repository

https://github.com/shaharia-lab/claude-power-user

## Support

[GitHub Issues](https://github.com/shaharia-lab/claude-power-user/issues)
