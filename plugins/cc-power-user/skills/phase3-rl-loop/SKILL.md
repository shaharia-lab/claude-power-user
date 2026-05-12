---
name: phase3-rl-loop
description: Iteratively improve code through CI/CD feedback until all quality gates pass. Use after code generation (Phase 2) to perfect implementation through automated testing and review.
argument-hint: "[branch-name]"
---

# PHASE 3: RL LOOP

Iterate pre-commit → commit/push → CI → review → fix, until all gates pass. Typically 2–4 iterations, hard max 10.

---

## SPECIALIST DELEGATION

During the loop, delegate when criteria are met. Use only agents that are installed alongside this plugin; the examples below are common but optional.

- **Bulk lint repair** — if the linter reports ≥5 issues, or issues spanning multiple files, delegate to a stack-specific lint-fixer (e.g. `golang-lint-fixer` for Go). Prompt: `Fix all lint issues. Errors: {output}. Run the project's lint command and fix systematically.`
- **API schema drift** — new/modified API endpoints, or CI fails on schema validation. Delegate to a schema-manager agent (e.g. `openapi-schema-manager`). Prompt: `Update the API schema for: {endpoint changes}. Analyze handlers and update accordingly.`
- **Security findings** — security scan flags vulnerabilities → `security-guardian`. Prompt: `Review and fix these vulnerabilities: {scan output}.`

---

## ITERATION

### Step 1: Pre-commit hooks

```bash
pre-commit run --all-files
```

Pre-commit covers formatting (black/prettier/gofmt), linting (ruff/eslint/golangci-lint), type checking, unit tests, import sorting, trailing whitespace, large-file detection.

If any hook fails: read the full error, fix ALL instances (not just the first one), re-run, do NOT commit until clean. Never silence a check (`# noqa`, `--no-verify`) — fix the cause.

### Step 2: Commit & push

```bash
git commit -m "<type>: <description>

[body with bullets]

Closes #<n>"
git push origin feature/issue-<n>
```

Type: `feat | fix | refactor | test | docs | chore`.

### Step 3: Monitor CI

Wait for GitHub Actions to start, then collect outputs from every job: integration tests, security scans, performance benchmarks, code coverage. Note which passed, which failed, exact errors, warnings.

### Step 4: Read PR review comments

Categorize: **BLOCKER** (must fix), **SUGGESTION** (fix if reasonable), **NITPICK** (low priority).

### Step 5: Evaluate

Exit the loop when ALL are true: pre-commit pass · all CI jobs green · zero blockers · coverage meets target · no security findings · performance within targets · all WHAT criteria demonstrably met. Otherwise continue.

### Step 6: Fix

Priority order: failing tests / security / blockers / breaking changes → performance / quality suggestions / coverage gaps / warnings → style nitpicks.

Validate fixes locally before the next iteration:
```bash
pytest tests/test_foo.py::test_bar  # specific failing test
pytest                              # full suite
pre-commit run --all-files
```

---

## DEBUGGING POINTERS

- **Same lint error repeating** — you're treating the symptom, not the cause. Read the rule, understand why it exists, look at similar code in the repo, fix properly.
- **Tests pass locally but fail in CI** — check Python/Node version pinning, timezone (`datetime.now(timezone.utc)`), nondeterminism (sort results, seed RNG), path handling (`Path` not strings).
- **Vague review comments** — look at the line, find similar patterns in the codebase, choose the safest/most-maintainable option, document your decision in code.
- **Stuck after 5+ iterations** — re-read the refined issue (did you misunderstand?), review what changed across iterations (did you regress?), check assumptions about types/None/empty, simplify, look for systemic causes (env, config, missing dep).
- **Common error patterns** — missing module → add to requirements + clear CI cache; type errors → handle None explicitly; flaky tests → replace `time.sleep` with explicit wait, seed random data; perf regression → profile, add caching, eliminate N+1, batch.

---

## FAILURE EXIT

If iteration_count reaches 10, STOP. Document:

```markdown
After 10 iterations, unresolved:
1. [issue + error]
2. [issue + error]

Attempted:
- [...]
- [...]

State: branch [link], test summary, unresolved review items.

Likely causes: [your analysis].

Human intervention needed.
```

Do not continue.

---

## EXIT CRITERIA

See the `quality-gates` skill for the single source of truth. When all gates pass, proceed to Phase 4 (`phase4-pr` skill).
