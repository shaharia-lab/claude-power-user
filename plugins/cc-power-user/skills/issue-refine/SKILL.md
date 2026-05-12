---
name: issue-refine
description: Refine GitHub issues by creating WHAT/WHY/HOW sections and preparing them for implementation. Adds issues to GitHub Projects V2, sets status to "Ready", and assigns the current iteration.
argument-hint: "[issue-number|issue-url] [--iteration=<name>] [--analyze-service=<service>]"
---

# GITHUB ISSUE REFINEMENT

Refine a GitHub issue into a WHAT / WHY / HOW spec, update its body (never a comment), then add it to the project board with status "Ready" on the current iteration.

---

## ARGUMENTS

- **Issue:** `123` or `https://github.com/<org>/<repo>/issues/123`
- **`--iteration=<name>`** — override auto-detected iteration
- **`--analyze-service=<backend|frontend|cli|website|docs-site>`** — force a particular research delegation
- **`--no-project`** — skip GitHub Projects V2 integration

---

## STEPS

### 1. Validate

```bash
gh issue view <n> --json title,body,number,url,labels
```

Confirm the issue exists and capture title/body/labels.

### 2. Refinement status

Look for `## WHAT`, `## WHY`, `## HOW` in the body.
- All present → skip to step 5 (project integration).
- Otherwise → step 3.

### 3. Detect services (optional research)

Scan title/body/labels for service hints. Match against whichever developer agents are installed in this plugin; common categories:

| Hint keywords | Developer agent |
|---|---|
| "backend", "API", "database", "server" | `backend-developer` |
| "frontend", "UI", "component" | `frontend-developer` |
| "CLI", "command-line", "terminal" | `cli-developer` |
| "website", "marketing", "landing" | `website-developer` |
| "docs", "documentation" | `docs-site-developer` |

If `--analyze-service=<s>` is set, use only that. Otherwise, deeper research before refining is **optional**. If you want it, ask the user:

```
Detected services: <list>. Run service-specific research?
y = run all, n = skip, <name> = run one only.
```

If yes, delegate a read-only analysis (no code changes) to the matching developer agent:

```
Analyze the codebase for context on this issue: <title>.
Focus on affected modules, current implementations, patterns to follow,
dependencies, and integration points. Return architectural context, file
locations, and integration points. Do not write any code.
```

Merge results into the refinement context.

### 4. Refine

Read the `phase1-refinement` skill. Produce WHAT / WHY / HOW per its templates. Then update the issue **body** (not a comment):

```bash
ISSUE_NUMBER=<n>
TEMP=/tmp/refined_issue_${ISSUE_NUMBER}.md
CURRENT_BODY=$(gh issue view $ISSUE_NUMBER --json body -q '.body')

cat > "$TEMP" << EOF
${CURRENT_BODY}

---
## ISSUE REFINEMENT

[WHAT]
[WHY]
[HOW]

---
**Refined.** Ready for implementation.
EOF

gh issue edit $ISSUE_NUMBER --body-file "$TEMP"
gh issue view $ISSUE_NUMBER --json body -q '.body' | grep -q "ISSUE REFINEMENT" \
  && echo "[OK] body updated" || { echo "[ERROR] body update failed"; exit 1; }
rm "$TEMP"

gh issue edit $ISSUE_NUMBER --add-label refined --add-label complexity:<low|medium|high>
```

Never use `gh issue comment` for refinement content.

### 5. GitHub Projects V2 integration

Skip entirely if `--no-project` is set or the repo has no Projects V2 board.

Read the `github-project-integration` skill. Substitute the project's real IDs for the placeholders (`PROJECT_ID_PLACEHOLDER`, `STATUS_FIELD_ID_PLACEHOLDER`, etc.).

```bash
ISSUE_ID=$(gh issue view <n> --json id --jq .id)

# 5a. Add to project (recover existing item if already added)
PROJECT_ITEM_ID=$(gh api graphql -f query='
  mutation {
    addProjectV2ItemById(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      contentId: "'$ISSUE_ID'"
    }) { item { id } }
  }
' -q '.data.addProjectV2ItemById.item.id' 2>/dev/null \
  || gh api graphql -f query='
    query {
      node(id: "'$ISSUE_ID'") {
        ... on Issue { projectItems(first: 10) { nodes { id project { id } } } }
      }
    }
  ' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id')

# 5b. Status = Ready
gh api graphql -f query='
  mutation {
    updateProjectV2ItemFieldValue(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      itemId: "'$PROJECT_ITEM_ID'"
      fieldId: "STATUS_FIELD_ID_PLACEHOLDER"
      value: { singleSelectOptionId: "READY_OPTION_ID_PLACEHOLDER" }
    }) { projectV2Item { id } }
  }
'

# 5c. Iteration (auto-detect current, or use --iteration override)
gh api graphql -f query='
  mutation {
    updateProjectV2ItemFieldValue(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      itemId: "'$PROJECT_ITEM_ID'"
      fieldId: "ITERATION_FIELD_ID_PLACEHOLDER"
      value: { iterationId: "<current-iteration-id>" }
    }) { projectV2Item { id } }
  }
'
```

---

## ERROR HANDLING

- **Issue not found** → verify number and repo; STOP.
- **Already refined** → log it, skip step 4, run step 5.
- **GraphQL mutation fails** → verify IDs via the field-discovery query in `github-project-integration`; manual fallback is the project URL in a browser.
- **Body update verification fails** → STOP and report; do not proceed to project integration.

---

## LOGGING

```
[REFINE] Issue #<n>: validated · services=<...> · refined · labelled
[PROJECT] Added · status=Ready · iteration=<name>
[COMPLETE] #<n> ready for implementation
```

---

## NEXT STEP

Once refined and Ready, the user can implement with `develop-issue <n>`.

---

## INVARIANTS

- Update the issue **body** with `gh issue edit --body[-file]`, never `gh issue comment`.
- Read `phase1-refinement` before drafting WHAT/WHY/HOW.
- Skip Projects V2 mutations entirely if `--no-project` or the repo has no board.
- This command does not start implementation. The `develop-issue` workflow does.
