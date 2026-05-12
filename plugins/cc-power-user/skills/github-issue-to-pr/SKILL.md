---
name: github-issue-to-pr
description: Transform a GitHub issue into a production-ready pull request with automated PR review. Executes full workflow including refinement check, code generation, RL loop, testing, PR creation, automated review, feedback application, and re-verification before marking ready for human review.
argument-hint: "[issue-number|issue-url]"
disable-model-invocation: true
---

# AUTONOMOUS DEVELOPMENT WORKFLOW

Transform ONE GitHub issue into a production-ready PR. Automated PR review and feedback application happen BEFORE marking the PR ready for humans.

**Assigned issue:** `[ISSUE_URL_PLACEHOLDER]` — work on this one only.

---

## INVARIANTS

These rules apply to every phase. Do not restate, just follow.

1. ONE issue only. Never look at, reference, or suggest others.
2. Phases run in order. Never skip.
3. Stay in RL loop (Phase 3) until ALL quality gates pass; max 10 iterations, no human help mid-loop.
4. Read the relevant skill at the top of each phase.
5. Worktree path: `/tmp/<project-name>/.worktree/<issue-number>` — all work happens there.
6. Sub-agents NEVER commit or push. The orchestrator commits.
7. Sub-agents MUST return a structured handoff (template in Phase 2B); validate before committing.
8. Parallel sub-agent calls when independent; sequential when dependent.
9. PR is created as draft and only marked ready after all checks pass.
10. After PR is marked ready: update issue status to "In Review" (if Projects V2 configured) and STOP.

**Logging format (mandatory):**
```
[PHASE-X] [ACTION|RESULT|DECISION|COMPLETE] message
[RL-LOOP-N] [ACTION|RESULT|DECISION] message
```

---

## PHASE -1: PRE-FLIGHT CHECKS

All three checks must pass.

### 1. Refinement check

Read the issue. It must have **WHAT**, **WHY**, **HOW** sections. If missing:

> Issue #N is not refined. Run the `github-issue-refiner` skill from this plugin first (e.g. `/cc-power-user:github-issue-refiner N`), then re-run this workflow. STOP.

Do not refine automatically.

### 2. Project status check (skip if GitHub Projects V2 is not configured)

Query the issue's Projects V2 status. Must be "Ready" (option ID `READY_OPTION_ID_PLACEHOLDER`).

```bash
gh api graphql -f query='
  query {
    repository(owner: "<your-org>", name: "<your-repo>") {
      issue(number: <issue-number>) {
        projectItems(first: 10) {
          nodes {
            fieldValueByName(name: "Status") {
              ... on ProjectV2ItemFieldSingleSelectValue { optionId name }
            }
          }
        }
      }
    }
  }
' -q '.data.repository.issue.projectItems.nodes[0].fieldValueByName | {optionId, name}'
```

If not "Ready", report current status and STOP. The user must move it to Ready (via refinement or manually).

### 3. Worktree

```bash
mkdir -p /tmp/<project-name>/.worktree
git worktree add /tmp/<project-name>/.worktree/<issue-number> -b feature/issue-<issue-number>
cd /tmp/<project-name>/.worktree/<issue-number>
```

On failure: `git worktree list && git worktree repair`, then retry.

---

## PHASE 0: ISSUE ASSESSMENT

Confirm WHAT/WHY/HOW present (verified in Phase -1) and current directory is the worktree. Proceed to Phase 1.

---

## PHASE 1: STATUS → IN PROGRESS

Update before any sub-agent delegation. This signals work started, prevents duplicate work, makes progress visible.

Skip if GitHub Projects V2 is not configured for the repo.

```bash
ISSUE_NUMBER=<issue-number>
ISSUE_NODE_ID=$(gh issue view $ISSUE_NUMBER --json id --jq .id)

PROJECT_ITEM_ID=$(gh api graphql -f query='
  query {
    node(id: "'$ISSUE_NODE_ID'") {
      ... on Issue {
        projectItems(first: 10) { nodes { id project { id } } }
      }
    }
  }
' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id')

gh api graphql -f query='
  mutation {
    updateProjectV2ItemFieldValue(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      itemId: "'$PROJECT_ITEM_ID'"
      fieldId: "STATUS_FIELD_ID_PLACEHOLDER"
      value: { singleSelectOptionId: "IN_PROGRESS_OPTION_ID_PLACEHOLDER" }
    }) { projectV2Item { id } }
  }
'
```

If the mutation fails: STOP and report. Do not proceed.

See the `github-project-integration` skill for the full transition reference.

---

## PHASE 2: CODE GENERATION

Read the `phase2-codegen` skill first.

Three steps: 2A delegate → 2B validate handoff → 2C commit.

### Service detection

Map the issue's HOW section to a single developer agent (or several, run in dependency order). Customise this table for your stack:

| Area of change | Developer agent |
|---|---|
| Backend / API | `backend-developer` |
| Frontend | `frontend-developer` |
| CLI | `cli-developer` |
| Marketing site | `website-developer` |
| Docs site | `docs-site-developer` |

For cross-cutting changes (small config/docs), the orchestrator can self-implement instead of delegating.

### 2A — Delegate to developer(s)

Run in parallel when independent, sequentially when one service depends on another. Prompt template:

```
Implement issue #[N] in workspace /tmp/<project-name>/.worktree/[N] on branch feature/issue-[N].

## WHAT
{WHAT_SECTION}

## WHY
{WHY_SECTION}

## HOW
{HOW_SECTION}

## STANDARDS
Test coverage ≥80%. Linting and type checking must pass. Follow this project's
existing conventions (the developer agent already has stack-specific guidance).

## HANDOFF (MANDATORY — return this exact structure)

## IMPLEMENTATION COMPLETE

### Files Modified
- `path/file.ext` - description

### Tests Added
- Total new tests: X
- Coverage: Y%
- All passing: true/false

### Quality Checks
- Linting: pass/fail
- Type checking: pass/fail
- Formatting: pass/fail

### Key Architectural Decisions
1. [Decision] - [Justification]

### Blockers or Concerns
None | [details]

### Ready Status
TRUE | FALSE - [reason]

## RULES
- Write code to the worktree.
- Run tests/linting locally.
- DO NOT commit or push.
- Return the handoff above.
```

### 2B — Validate handoff

Required: Files Modified, Tests Added (counts/coverage), Quality Checks, Decisions, Ready Status. Gates: Ready=TRUE, all tests passing, linting passed, coverage ≥80%, no blockers. If any fail, re-delegate with specific fixes; do not commit.

### 2C — Commit (orchestrator only)

```bash
cd /tmp/<project-name>/.worktree/[N]
git add .
git commit -m "$(cat <<'EOF'
feat: [brief description]

[detailed description from WHAT/WHY]

Implementation:
- [key change 1]
- [key change 2]

Tests: X new, Y% coverage
Quality: Linting pass, Type checking pass

Implements: #[N]
EOF
)"
git push origin HEAD
```

Proceed to Phase 3.

---

## PHASE 3: RL LOOP

Read the `phase3-rl-loop` skill first.

```
iteration = 0
while not all_gates_passed and iteration < 10:
    iteration += 1
    run pre-commit; fix all failures; do NOT commit yet
    commit & push from worktree
    wait for GitHub Actions; collect all job outputs
    read PR review comments; categorize blocker/suggestion/nit
    if all CI green and no blockers: break
    apply fixes (delegate to specialist if appropriate); next iteration

if iteration == 10: document remaining issues, request human help, STOP
```

**Specialist delegations during the loop (only when the relevant agent exists in your install):**
- Stack-specific linting that needs bulk repair → tech-specific lint-fixer agent (e.g. `golang-lint-fixer` for Go).
- New/changed API endpoints → schema-manager agent (e.g. `openapi-schema-manager`).
- Security scan findings → `security-guardian`.

**Exit criteria (ALL true):**
pre-commit pass · CI green · zero blocker comments · coverage maintained or improved · no security findings · performance acceptable.

---

## PHASE 4: PR + AUTOMATED REVIEW

Read the `phase4-pr` skill first.

Trigger pre-PR QA delegations as applicable, fix any critical/high findings, then:

- security-sensitive changes (auth, payments, user data, secrets) → `security-guardian`
- architectural changes (new patterns, service boundaries, major refactor) → `architecture-guardian`
- complex business logic → `bug-finder`

**Step sequence:**
1. Create PR as **draft** with comprehensive description (template in skill file), link `Closes #N`.
2. Add labels: `ai-generated`, `ready-for-review`, complexity label. Create labels if missing.
3. Wait for CI green.
4. Delegate to `pr-reviewer`; parse findings.
5. If critical/high found: fix, commit, push, wait for CI green again. If CI fails after fixes: STOP, request human help.
6. `gh pr ready $PR_NUMBER` → PR no longer draft.
7. Update issue status to "In Review" (option ID `IN_REVIEW_OPTION_ID_PLACEHOLDER`) — skip if Projects V2 not configured.
8. Comment on the original issue with the PR link.
9. STOP. Do not look for next work.

---

## AVAILABLE SUB-AGENTS

The plugin ships a default set; override per project as needed.

**Developers:** `backend-developer`, `frontend-developer`, `cli-developer`, `website-developer`, `docs-site-developer`

**QA:** `security-guardian`, `architecture-guardian`, `bug-finder`, `pr-reviewer`

**Stack-specific specialists (optional, use only if installed):** e.g. `golang-lint-fixer`, `openapi-schema-manager`

---

## SKILL FILES

Read these at the top of the corresponding phase:

- `phase1-refinement` — refinement templates (only relevant if invoked via the refining command; this workflow assumes refinement is already done)
- `phase2-codegen` — code quality standards
- `phase3-rl-loop` — error handling and recovery
- `phase4-pr` — PR template and checklist
- `quality-gates` — shared gates (single source of truth)
- `github-project-integration` — Projects V2 IDs and status transitions
