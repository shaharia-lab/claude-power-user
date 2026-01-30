---
name: quality-gates
description: Shared quality gates that all development phases must pass before proceeding to the next phase
---

# Shared Quality Gates

All development phases must pass these quality gates before proceeding to the next phase.

---

## Pre-commit Validation

Run all pre-commit hooks:
```bash
pre-commit run --all-files
```

**Must pass:**
- Code formatting (black, prettier, gofmt)
- Linting (ruff, eslint, golangci-lint)
- Type checking (mypy, tsc)
- Import sorting
- Trailing whitespace removal
- Large file detection

---

## Test Requirements

### Unit Tests
- All unit tests must pass
- Coverage must be ≥80% for new code
- No skipped tests without justification

### Integration Tests
- All integration tests must pass
- Database migrations tested

### E2E Tests
- Critical user flows tested
- No UI regressions

---

## Security Scan

```bash
# Go projects
gosec ./...

# Node projects
npm audit
# or
yarn audit

# Python projects
bandit -r .
```

**Must pass:**
- No high or critical vulnerabilities
- All medium vulnerabilities documented
- Dependencies up to date

---

## Architecture Review

Delegate to `architecture-guardian` agent for review:
- Design patterns followed
- SOLID principles maintained
- No anti-patterns introduced
- Dependency directions correct

---

## Bug Analysis

Delegate to `bug-finder` agent:
- Edge cases covered
- Error handling comprehensive
- Race conditions eliminated
- Boundary conditions tested

---

## Performance Check

- No performance regressions
- Database queries optimized
- API response times acceptable
- Memory leaks prevented

---

## Documentation

- API changes documented
- README updated if needed
- Code comments for complex logic
- Architecture decisions recorded (ADR)

---

## Success Criteria

All gates must be ✅ before proceeding:

- [ ] Pre-commit hooks pass
- [ ] All tests pass (unit, integration, E2E)
- [ ] Security scan clean
- [ ] Architecture review approved
- [ ] Bug analysis clean
- [ ] Performance acceptable
- [ ] Documentation updated
