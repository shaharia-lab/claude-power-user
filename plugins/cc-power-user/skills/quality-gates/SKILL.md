# Shared Quality Gates All development phases must pass these quality gates before proceeding to the next phase. --- ## Pre-commit Validation Run all pre-commit hooks:
```bash
pre-commit run --all-files
``` **Must pass:**
- Code formatting (black, prettier, gofmt)
- Linting (ruff, eslint, golangci-lint)
- Type checking (mypy, tsc)
- Import sorting
- Trailing whitespace removal
- Large file detection --- ## Test Requirements ### Unit Tests
- [ ] All new functions have unit tests
- [ ] All modified functions have updated tests
- [ ] Happy path tested
- [ ] Edge cases tested
- [ ] Error handling tested
- [ ] Tests are deterministic (no flaky tests)
- [ ] Tests run fast (<5 seconds for unit suite) ### Integration Tests
- [ ] Main workflows covered
- [ ] Component interactions tested
- [ ] External dependencies mocked appropriately ### Coverage
- [ ] Code coverage meets or exceeds target (usually 85%+)
- [ ] No decrease in coverage from baseline --- ## Code Quality ### Required Standards
- [ ] Proper imports (grouped, alphabetized, no unused)
- [ ] Type hints/annotations throughout
- [ ] Docstrings on all public functions
- [ ] Meaningful error messages
- [ ] No magic numbers/strings (use constants)
- [ ] Clear, descriptive variable names
- [ ] Single responsibility principle followed
- [ ] DRY principles applied ### Code Health
- [ ] No print statements in production code
- [ ] No commented-out code blocks
- [ ] No TODO comments (move to issues)
- [ ] No debug logging left enabled --- ## Security - [ ] No security vulnerabilities detected
- [ ] Input validation on all user inputs
- [ ] No sensitive data in logs or error messages
- [ ] Authentication/authorization unchanged or properly updated
- [ ] Dependencies scanned for vulnerabilities --- ## Performance - [ ] Performance benchmarks within targets
- [ ] No obvious N+1 queries
- [ ] Appropriate caching implemented
- [ ] No memory leaks introduced --- ## Documentation - [ ] Code documentation (docstrings, inline comments for complex logic)
- [ ] README updated (if public API changed)
- [ ] API documentation updated (if endpoints changed)
- [ ] Configuration changes documented
- [ ] New environment variables documented --- ## CI/CD Pipeline All automated checks must pass:
- [ ] Build succeeds
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Linting passes
- [ ] Type checking passes
- [ ] Security scans clean
- [ ] Performance benchmarks acceptable
- [ ] Code coverage meets threshold --- ## Acceptance Criteria - [ ] Each acceptance criterion from WHAT section is satisfied
- [ ] Implementation matches HOW section approach
- [ ] Business value from WHY section is delivered
- [ ] No deviations from plan (or documented why) --- ## Quick Verification Commands ```bash
# Python projects
pre-commit run --all-files
pytest
ruff check .
mypy src/ # Go projects
pre-commit run --all-files
go test ./...
golangci-lint run # Node/TypeScript projects
pre-commit run --all-files
npm test
npm run lint
tsc --noEmit
``` --- ## Exit Criteria Summary Before proceeding to the next phase, verify: 1. **All pre-commit hooks pass**
2. **All tests pass (unit + integration)**
3. **Linting passes with no errors**
4. **Type checking passes**
5. **Code coverage meets target**
6. **No security vulnerabilities**
7. **Performance benchmarks acceptable**
8. **All acceptance criteria satisfied** Only proceed when ALL gates are green.
