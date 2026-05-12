---
name: phase1-refinement
description: Transform ambiguous GitHub issues into refined specifications with WHAT, WHY, and HOW sections. Use before implementing features or fixes when issue lacks clear requirements.
argument-hint: "[issue-number]"
---

# PHASE 1: REFINEMENT

Transform an ambiguous issue into a refined spec with WHAT / WHY / HOW sections, then update the issue body.

Primarily invoked by `issue-refine`. The `develop-issue` workflow does NOT auto-refine — issues must already be refined.

---

## STEP 1: ANALYZE

**Issue:** Read description + all comments. Understand intent. Note ambiguities.

**Codebase:** Find similar implementations, current patterns, relevant dependencies, project conventions (style/testing/docs).

**Scope:** List affected components, integration points, edge cases, technical risks.

---

## STEP 2: WHAT

```markdown
## WHAT

**Summary:** [one-line description]

**Scope:**
- [specific feature/fix/improvement]

**Affected Components:**
- `path/to/file` - [what changes]

**Acceptance Criteria:**
- [ ] [testable criterion — write a test for it]
- [ ] ...

**Out of Scope:**
- [explicitly excluded]
```

Aim for 3–6 acceptance criteria. Each must be testable and concrete (e.g. "API p95 < 200ms", not "make it faster").

---

## STEP 3: WHY

```markdown
## WHY

**Business Value:** [why this matters to users/business]
**Problem Statement:** [current state and its consequences]
**Expected Impact:** [UX / performance / debt / risk — quantify when possible]
**Context:** [related/depends-on/blocks issues, dates, dependencies]
```

Be realistic, not aspirational.

---

## STEP 4: HOW

```markdown
## HOW

**Technical Approach:** [2–3 sentence strategy; reference existing patterns]

**Implementation Details:**

### 1. [Component]
**File:** `path/to/file.ext`
**Changes:** [specific functions/classes to add/modify]

### 2. [Component]
...

### Testing Strategy
- Unit: [what's tested in isolation]
- Integration: [end-to-end flows]
- Test files: `tests/...`

### Documentation
- Update `README.md` section X, docstrings, API docs

### Configuration
- New deps: `package==version` — [why]
- New env vars: `VAR` — [purpose, default]

**Files to Create:** `src/...`
**Files to Modify:** `src/...`

**Risks:**
- Risk: [...] — Mitigation: [...]

**Complexity:** Low | Medium | High
- Low: single file, <100 LOC, clear pattern
- Medium: multiple files, <500 LOC, some new patterns
- High: many files, >500 LOC, new architecture
```

Someone else should be able to implement from the HOW section alone.

---

## STEP 5: UPDATE THE ISSUE

**Update issue BODY, not a comment.** Use `gh issue edit --body`.

```bash
ISSUE_NUMBER=<issue-number>
TEMP=/tmp/refined_issue_${ISSUE_NUMBER}.md

CURRENT_BODY=$(gh issue view $ISSUE_NUMBER --json body -q '.body')

cat > "$TEMP" << 'EOF'
${CURRENT_BODY}

---
## ISSUE REFINEMENT

[WHAT section]

[WHY section]

[HOW section]

---
**Refinement completed.** Ready for implementation.
EOF

gh issue edit $ISSUE_NUMBER --body-file "$TEMP"
gh issue view $ISSUE_NUMBER --json body -q '.body' | grep "ISSUE REFINEMENT"
rm "$TEMP"
```

**Labels:**
```bash
gh issue edit $ISSUE_NUMBER \
  --add-label refined \
  --add-label complexity:<low|medium|high>
```

---

## STEP 6: ASK FOR PERMISSION

Stop and ask:

```markdown
Issue #N refined.

Summary: [one-line]
Complexity: Low | Medium | High
Components affected: N
Risks: [None | Low | Medium | High]

Option 1: Start implementation now (`develop-issue N`)
Option 2: Wait — I'll stop here.

Type `1` or `2`.
```

Do NOT proceed to implementation without explicit permission.

---

## QUALITY CHECK BEFORE UPDATING

- WHAT: clear summary, specific scope, 3–6 testable criteria, out-of-scope listed
- WHY: business value, problem, realistic impact, context links
- HOW: approach explained, file paths + function/class names listed, tests defined, deps/config specified, risks with mitigations, complexity honest
- Overall: implementable from HOW alone, no ambiguity remaining
