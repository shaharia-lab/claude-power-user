---
name: phase4-pr
description: Create comprehensive PR with automated review and feedback application. Creates a draft PR, runs automated review, fixes critical/high findings, re-verifies CI, and marks ready for human review.
argument-hint: "[pr-number]"
---

# PHASE 4: PR + AUTOMATED REVIEW

Create a draft PR, run an automated review, fix critical/high findings, re-verify CI, mark ready, update project board.

**Prerequisites:** All Phase 3 gates passed.

---

## PRE-PR QA DELEGATION

Trigger based on the nature of changes:

- **`security-guardian`** — changes to auth, authorization, payments, user data, API keys, secrets. Focus: auth flaws, input validation, sensitive data exposure, API security. Block on Critical/High.
- **`architecture-guardian`** — new patterns, service-structure changes, major refactor, new integrations. Focus: pattern consistency, separation of concerns, service boundaries, MVP-appropriate complexity.
- **`bug-finder`** — complex business logic, multi-step workflows, data transformations. Focus: logic errors, edge cases, race conditions, error handling gaps.

Critical/High issues must be fixed before proceeding. Medium: fix if simple, else document in the PR. Low: note for future.

---

## STEP 1: PR DESCRIPTION

Use this template. Fill every section; do not leave placeholders.

```markdown
## Summary
[2–3 sentences: what was done, briefly why]

Closes #<issue-number>

---

## What Changed

### Files Created
- `path/file.ext` - [purpose]

### Files Modified
- `path/file.ext` - [what changed]

### Key Changes
1. [Component]: [change]
2. ...

---

## Why
**Business Value:** [from WHY section]
**Problem Solved:** [from WHY section]
**Expected Impact:**
- UX: [...]
- Performance: [...]
- Maintainability: [...]

---

## How
**Technical Approach:** [from HOW section]
**Architecture:** [component interactions; optional ASCII diagram]
**Key Implementation Details:**
1. [...]
2. [...]

---

## Testing
- Unit tests: X new
- Integration tests: Y scenarios
- Total: Z; coverage: N% (target: M%)
- Scenarios: happy path / edge cases / errors / performance

**Manual test:**
1. [step]
2. [expected outcome]

---

## Acceptance Criteria
[Copy from WHAT, check each one]
- [x] [criterion 1]
- [x] [criterion 2]

---

## Configuration & Deployment
**New env vars:** [name — purpose, default]
**New deps:** [package==version — why]
**Config files changed:** [...]
**Deployment notes:**
- [ ] Update prod env vars
- [ ] Run migrations (if any)
**Breaking changes:** [None | description + migration]

---

## Security
- Security scan: clean
- Auth: [unchanged | description]
- Input validation: [...]
- Data exposure: [...]

---

## Performance
- API latency: Xms (target: <Yms)
- Memory: XMB (target: <YMB)
- DB queries per request: X

---

## AI Workflow Stats
- RL loop iterations: N
- Issues auto-resolved: Y
- Total time: ~Z min

---

## Reviewer Notes
Focus on: business logic correctness, edge cases, security implications, architectural fit.
[Optional: specific questions for reviewer]

---

## Related
- Related to: #N
- Depends on: #N
- Blocks: #N
```

Pull WHAT/WHY/HOW directly from the refined issue. Be specific in "Files Modified" and "Key Implementation Details". Confirm every acceptance criterion is genuinely met before checking it.

---

## STEP 2: METADATA

**Link issue:** `Closes #N` in the PR body (auto-closes on merge).

**Labels:** ensure `ready-for-human-review` and `automated-review-done` exist; create if missing:

```bash
gh label list --search "ready-for-human-review" > /dev/null 2>&1 || \
  gh label create "ready-for-human-review" --description "PR is ready for human review" --color "0E8A16"
gh label list --search "automated-review-done" > /dev/null 2>&1 || \
  gh label create "automated-review-done" --description "Automated PR review completed" --color "1D76DB"
```

Also add: complexity label from the issue, type label (feature/bug/etc.), and any of `documentation`/`breaking-change`/`needs-migration` that apply.

**Reviewers:** assign team member or team. **Project/milestone:** add if applicable.

---

## STEP 3: CREATE DRAFT PR

### 3A — Session ID (optional, for traceability)

```bash
SESSION_ID="${CLAUDE_SESSION_ID_CUSTOM:-unknown}"
[ "$SESSION_ID" = "unknown" ] && echo "[WARN] CLAUDE_SESSION_ID_CUSTOM not set"
```

### 3B — Draft PR + labels

```bash
cat > /tmp/pr_description.md << 'EOF'
<paste full PR description from Step 1>
EOF

gh pr create --draft \
  --title "feat: <brief-title>" \
  --body-file /tmp/pr_description.md \
  --base main

PR_NUMBER=$(gh pr view --json number --jq .number)
rm /tmp/pr_description.md

gh pr edit $PR_NUMBER \
  --add-label "ready-for-human-review" \
  --add-label "automated-review-done" \
  --add-label "complexity:<level>"

gh pr view $PR_NUMBER --json isDraft,title,state
```

### 3C — Wait for CI green

```bash
while true; do
  FAILED=$(gh pr view $PR_NUMBER --json statusCheckRollup \
    --jq '.statusCheckRollup[] | select(.conclusion != "SUCCESS") | .conclusion')
  PENDING=$(gh pr view $PR_NUMBER --json statusCheckRollup \
    --jq '.statusCheckRollup[] | select(.status == "IN_PROGRESS" or .status == "PENDING") | .name' | wc -l)
  if [ -z "$FAILED" ] && [ "$PENDING" -eq 0 ]; then echo "CI green"; break; fi
  echo "$FAILED" | grep -q "FAILURE" && { echo "CI FAILED"; exit 1; }
  sleep 30
done
```

If CI fails: return to Phase 3.

### 3D — Automated PR review

Delegate to `pr-reviewer`:

```
Review PR #$PR_NUMBER for security vulnerabilities, code quality, potential
bugs, breaking changes, and performance concerns. Focus on OWASP Top 10,
critical and high severity bugs, and code quality violations. Provide
actionable feedback with severity ratings.
```

Categorize findings: Critical / High / Medium / Low.

- No Critical/High → skip 3E, proceed to 3F.
- Critical/High found → go to 3E.

### 3E — Apply fixes

Group by category (security first, then bugs, then quality). For each: read affected file, understand context, apply fix.

Validate locally:
```bash
make lint && make test       # backend
npm run lint && npm test     # frontend/cli/website
```

Commit & push:
```bash
git add .
git commit -m "$(cat <<'EOF'
fix: apply automated PR review feedback

Critical fixes:
- ...

High severity fixes:
- ...
EOF
)"
git push origin HEAD
```

Document Medium/Low items as a PR comment for follow-up.

### 3F — Re-verify CI after fixes

Reuse the wait loop from 3C. If CI fails after fixes: STOP and request human help — automated fixes introduced regressions.

### 3G — Mark PR ready

```bash
gh pr ready $PR_NUMBER
gh pr view $PR_NUMBER --json isDraft --jq .isDraft   # expect: false
```

### 3H — Update issue status → "In Review"

Use the `github-project-integration` skill. Same mutation pattern as Phase 1, with the "In Review" status option ID for your project.

```bash
ISSUE_NODE_ID=$(gh issue view $ISSUE_NUMBER --json id --jq .id)
PROJECT_ITEM_ID=$(gh api graphql -f query='
  query {
    node(id: "'$ISSUE_NODE_ID'") {
      ... on Issue { projectItems(first: 10) { nodes { id project { id } } } }
    }
  }
' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id')

gh api graphql -f query='
  mutation {
    updateProjectV2ItemFieldValue(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      itemId: "'$PROJECT_ITEM_ID'"
      fieldId: "STATUS_FIELD_ID_PLACEHOLDER"
      value: { singleSelectOptionId: "IN_REVIEW_OPTION_ID_PLACEHOLDER" }
    }) { projectV2Item { id } }
  }
'
```

Skip this step if GitHub Projects integration is not configured for the repo.

---

## STEP 4: VERIFY PR

CI badges green, file changes look reasonable (no accidental deletions/additions), no merge conflicts, branch up to date with main.

---

## STEP 5: COMMENT ON THE ISSUE

```markdown
Implementation complete — ready for human review.

PR: #<pr-number>

- All acceptance criteria met
- All tests passing
- Automated review feedback addressed
- Security/perf clean
- RL loop iterations: N

Please review for business logic, edge cases, and architectural fit.

cc @<reviewer>
```

---

## STEP 6: STOP

Workflow complete. Do not look for more issues, do not make extra commits unless a reviewer asks. Wait for human feedback.
