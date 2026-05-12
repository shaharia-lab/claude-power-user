---
name: phase2-codegen
description: Generate production-quality code following acceptance criteria from refined issue, with comprehensive tests and documentation. Use after issue refinement (Phase 1) is complete.
argument-hint: "[issue-number-or-description]"
---

# PHASE 2: CODE GENERATION

Generate complete, production-quality implementation from a refined issue.

**Prerequisites:** Issue has WHAT/WHY/HOW; user approved implementation.

---

## AGENT DELEGATION

Delegate to the developer agent matching the affected service:

| File pattern | Service | Developer |
|---|---|---|
| backend service paths, `*.go`, `internal/**`, `cmd/**` | Backend API | `backend-developer` |
| `frontend/**`, `src/components/**`, `src/pages/**` | Frontend | `frontend-developer` |
| `cli/**`, `src/commands/**` | CLI | `cli-developer` |
| `website/**` | Marketing site | `website-developer` |
| `docs-site/**`, `docs/**/*.md` | Docs site | `docs-site-developer` |

Delegate for service-specific changes. Self-implement for simple cross-cutting changes (small config/docs). For multi-service changes, run sequentially when dependent (backend → frontend) or in parallel when independent.

Delegation prompt template lives in the orchestrator workflow (`develop-issue` Step 2B).

---

## STEP 1: READ THE REFINED ISSUE

Extract: files to create/modify, functions/classes to implement, edge cases, tests to write. Map each acceptance criterion to the code that satisfies it.

---

## STEP 2: IMPLEMENT

For each component in HOW: create/open the file, implement with the exact names from HOW, handle the listed edge cases, add error handling. Follow the project's existing style, naming, file organization, and architectural patterns.

**Required in all code:**
- Imports grouped (stdlib, third-party, local), alphabetized, no unused
- Type hints / static types
- Docstrings on public functions, comments only for non-obvious logic
- Specific exceptions, not generic catches
- Constants instead of magic numbers/strings
- Clear names; one function = one responsibility
- DRY: extract repeated logic

**Language pointers:**
- Python: PEP 8, type hints, dataclasses, context managers, pathlib
- Go: idiomatic errors (`fmt.Errorf` with `%w`), defer cleanup, interfaces for abstractions
- TS/JS: `const`/`let`, async/await, strict types, project ESLint rules

---

## STEP 3: TESTS

For every new function: happy path, edge cases, error cases. Use AAA pattern (Arrange/Act/Assert).

- One test file per source file (`test_<source>.py` or similar)
- Unit tests mock external deps and run in milliseconds
- Integration tests cover end-to-end flows; mock external APIs but use real DB/FS where safe
- Each acceptance criterion has at least one test verifying it
- No flaky tests (no `time.sleep`, no unfixed randomness, no shared state)

---

## STEP 4: DOCUMENTATION

- Docstrings on every public function (args, returns, raises)
- Inline comments only for non-obvious logic
- Update README if public API changed, new feature added, install/config changed
- Update API docs if endpoints/schemas changed
- Document new env vars (purpose, default, valid range)

---

## STEP 5: DEPENDENCIES & CONFIG

- Add new packages to requirements/package.json with pinned versions; note purpose
- Update lock files
- New env vars: declare with defaults in code, document in `.env.example` and README
- Sensible defaults; document valid ranges

---

## STEP 6: VALIDATE AGAINST ACCEPTANCE CRITERIA

For each WHAT criterion, confirm: code implements it, a test would fail without it, you can demo it. Any unmet criterion blocks Phase 3 — implement and test it.

---

## SELF-CHECK BEFORE PHASE 3

```bash
# Python
pytest && ruff check . && mypy src/

# Go
go test ./... && golangci-lint run

# Node/TS
npm test && npm run lint && tsc --noEmit
```

No print statements, no commented-out code, no leftover TODOs, no debug logging.

For the canonical quality gate list see the `quality-gates` skill.

---

## EXIT CHECKLIST

- All HOW-section functions/classes implemented; edge cases handled; errors handled
- All new functions have tests; all acceptance criteria have tests; tests pass; coverage maintained or improved
- Docstrings on public functions; README/API/config docs updated as needed
- Deps + env vars + example configs in place
- Each WHAT criterion satisfied; implementation matches HOW (deviations documented)

Proceed to Phase 3 (`phase3-rl-loop` skill).
