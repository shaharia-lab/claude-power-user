---
name: quality-gates
description: Shared quality gates all phases must pass before proceeding. Single source of truth for pre-commit, tests, code quality, security, performance, documentation, CI, and acceptance criteria.
user-invocable: false
---

# Shared Quality Gates

Single source of truth for quality gates across all phases. Each phase links here instead of restating.

---

## Pre-commit

```bash
pre-commit run --all-files
```

Must pass: formatting (black/prettier/gofmt), linting (ruff/eslint/golangci-lint), type checking (mypy/tsc), import sorting, trailing whitespace, large-file detection.

---

## Tests

**Unit:** every new/modified function tested; happy path + edge cases + errors; deterministic; <5s for full unit suite.

**Integration:** main workflows covered; external dependencies mocked appropriately.

**Coverage:** meets or exceeds target (usually 80%+); no decrease from baseline.

---

## Code quality

- Imports grouped + alphabetized + no unused
- Type hints / annotations throughout
- Docstrings on public functions; comments only for non-obvious logic
- Specific exceptions, meaningful messages
- Constants, not magic numbers/strings
- Single responsibility; DRY
- No print statements / commented-out code / TODOs / debug logging in production paths

---

## Security

- No vulnerabilities detected (`gosec` / `npm audit` / `bandit`)
- Input validation on user inputs
- No sensitive data in logs/errors
- Auth/authz unchanged or properly updated
- Dependencies scanned

---

## Performance

- Benchmarks within targets
- No obvious N+1 queries
- Appropriate caching
- No memory leaks introduced

---

## Documentation

- Docstrings on public functions
- README updated if public API/install/config changed
- API docs updated if endpoints/schemas changed
- New env vars documented (purpose, default, range)

---

## CI/CD

All automated checks pass: build · unit tests · integration tests · linting · type checking · security · performance · coverage threshold.

---

## Acceptance criteria

- Each WHAT criterion satisfied
- Implementation matches HOW approach
- WHY business value delivered
- Deviations from plan documented

---

## Verification commands

```bash
# Python
pre-commit run --all-files && pytest && ruff check . && mypy src/

# Go
pre-commit run --all-files && go test ./... && golangci-lint run

# Node/TypeScript
pre-commit run --all-files && npm test && npm run lint && tsc --noEmit
```

---

## Exit criteria

Before proceeding to the next phase, ALL of the above must be green.
