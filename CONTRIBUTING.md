# Contributing to Claude Power User

Thank you for your interest in contributing! This document provides guidelines for contributing to the project.

## How to Contribute

### Reporting Issues

If you find a bug or have a feature request:

1. **Search existing issues** to avoid duplicates
2. **Create a new issue** with:
   - Clear title and description
   - Steps to reproduce (for bugs)
   - Expected vs actual behavior
   - Your environment (Claude Code version, OS, etc.)
   - Relevant logs or error messages

### Contributing Code

1. **Fork the repository**
   ```bash
   git clone https://github.com/shaharia-lab/claude-power-user.git
   cd claude-power-user
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**
   - Follow existing code style and conventions
   - Test your changes thoroughly
   - Update documentation as needed

4. **Commit your changes**
   ```bash
   git add .
   git commit -m "feat: add your feature description"
   ```

   Use conventional commit messages:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `docs:` for documentation changes
   - `refactor:` for code refactoring
   - `test:` for adding tests
   - `chore:` for maintenance tasks

5. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

6. **Create a Pull Request**
   - Provide clear title and description
   - Reference related issues
   - Include examples of usage
   - Ensure all tests pass

## Development Guidelines

### Agent Development

When creating or modifying agents:

1. **Follow the agent template structure**:
   ```markdown
   ---
   name: agent-name
   description: Clear, concise description
   model: inherit
   ---

   # Agent Name

   ## Purpose and Scope
   [Clear description of what the agent does]

   ## Guidelines
   [Specific instructions for the agent]

   ## Examples
   [Usage examples]
   ```

2. **Use generic, project-agnostic language**
   - Avoid hardcoded paths (use `${PROJECT_ROOT}`)
   - Avoid specific company/project names
   - Use placeholder values for IDs

3. **Test the agent** in real scenarios before submitting

### Command Development

When creating workflow commands:

1. **Keep commands focused** - One clear purpose per command
2. **Provide examples** - Show common usage patterns
3. **Document parameters** - Explain all flags and options
4. **Handle errors gracefully** - Provide helpful error messages

### Skill Development

When creating reusable skills:

1. **Make skills composable** - They should work well together
2. **Keep skills focused** - Single responsibility principle
3. **Document prerequisites** - What needs to be in place
4. **Provide examples** - Show how to use the skill

## Documentation

- Update relevant docs when changing functionality
- Add examples for new features
- Keep README.md up to date
- Update CONFIGURATION.md for new settings

## Code Style

### Markdown Files

- Use clear, descriptive headers
- Keep line length reasonable (80-100 chars)
- Use code blocks with language specification
- Include examples for complex concepts

### Python Scripts (if applicable)

- Follow PEP 8 style guide
- Use type hints where appropriate
- Add docstrings to functions
- Keep functions focused and testable

## Testing

Before submitting:

1. **Test your changes** with Claude Code CLI
2. **Verify sanitization** - No personal/project-specific data
3. **Test on clean environment** if possible
4. **Update tests** if applicable

## Questions?

- Open an issue for questions
- Tag with `question` label
- Provide context for your question

## Code of Conduct

- Be respectful and inclusive
- Focus on constructive feedback
- Help others learn and grow
- Assume positive intent

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing to Claude Power User! 🎉
