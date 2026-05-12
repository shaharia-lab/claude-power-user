---
name: github-project-integration
description: Shared skill for GitHub Projects V2 API operations. Includes GraphQL mutations for adding issues, updating status, and managing iterations.
user-invocable: false
---

# GITHUB PROJECTS V2 INTEGRATION

Reusable GraphQL patterns for managing issues on a GitHub Projects V2 board.

---

## CONFIGURATION

Set these per-project (e.g. via env vars or by replacing the placeholders below). Discover them with the field-discovery query at the bottom of this file.

```
Organization:    <your-org>
Project URL:     https://github.com/orgs/<your-org>/projects/<n>
Project number:  <n>
Project ID:      PROJECT_ID_PLACEHOLDER          (PVT_...)
Repository:      <your-repo>
```

### Fields

| Field | Field ID | Type |
|---|---|---|
| Status | `STATUS_FIELD_ID_PLACEHOLDER` (PVTSSF_...) | Single select |
| Iteration | `ITERATION_FIELD_ID_PLACEHOLDER` (PVTIF_...) | Iteration |

### Status options

Replace each option-ID placeholder with the value from your project (see field-discovery query):

| Status | Option ID | Meaning |
|---|---|---|
| Backlog | `BACKLOG_OPTION_ID_PLACEHOLDER` | Not yet ready |
| Ready | `READY_OPTION_ID_PLACEHOLDER` | Refined; ready to implement |
| In progress | `IN_PROGRESS_OPTION_ID_PLACEHOLDER` | Currently being implemented |
| In review | `IN_REVIEW_OPTION_ID_PLACEHOLDER` | PR under review |
| Done | `DONE_OPTION_ID_PLACEHOLDER` | Completed and merged |

### Iterations

Replace with the iteration IDs from your project. Auto-detect the current iteration by date.

```python
def get_current_iteration(today_date, iterations):
    for it in iterations:
        if it["start"] <= today_date < it["end"]:
            return it["name"], it["id"]
    return iterations[0]["name"], iterations[0]["id"]
```

---

## OPERATIONS

### Get issue node ID

```bash
gh issue view <issue-number> --repo <your-org>/<your-repo> --json id --jq '.id'
```

### Add issue to project

```bash
ISSUE_ID="<issue-node-id>"
PROJECT_ITEM_ID=$(gh api graphql -f query='
  mutation {
    addProjectV2ItemById(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      contentId: "'$ISSUE_ID'"
    }) {
      item { id }
    }
  }
' -q '.data.addProjectV2ItemById.item.id')
```

If already in the project, the mutation errors with `ContentAlreadyAddedToProject`. Recover with "Get existing project item ID".

### Update status

```bash
ITEM_ID="<project-item-id>"
STATUS_ID="<see status options table>"
gh api graphql -f query='
  mutation {
    updateProjectV2ItemFieldValue(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      itemId: "'$ITEM_ID'"
      fieldId: "STATUS_FIELD_ID_PLACEHOLDER"
      value: { singleSelectOptionId: "'$STATUS_ID'" }
    }) {
      projectV2Item { id }
    }
  }
'
```

### Update iteration

```bash
ITEM_ID="<project-item-id>"
ITERATION_ID="<iteration-id>"
gh api graphql -f query='
  mutation {
    updateProjectV2ItemFieldValue(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      itemId: "'$ITEM_ID'"
      fieldId: "ITERATION_FIELD_ID_PLACEHOLDER"
      value: { iterationId: "'$ITERATION_ID'" }
    }) {
      projectV2Item { id }
    }
  }
'
```

### Query issue status

```bash
ISSUE_NUMBER=<n>
gh api graphql -f query='
  query {
    repository(owner: "<your-org>", name: "<your-repo>") {
      issue(number: '$ISSUE_NUMBER') {
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

### Get existing project item ID

```bash
ISSUE_NODE_ID="<issue-node-id>"
PROJECT_ITEM_ID=$(gh api graphql -f query='
  query {
    node(id: "'$ISSUE_NODE_ID'") {
      ... on Issue {
        projectItems(first: 10) { nodes { id project { id } } }
      }
    }
  }
' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id')
```

---

## ORCHESTRATED FLOW (add → status Ready → assign iteration)

```bash
#!/bin/bash
ISSUE_NUMBER=$1

ISSUE_ID=$(gh issue view $ISSUE_NUMBER --repo <your-org>/<your-repo> --json id --jq '.id')

PROJECT_ITEM_ID=$(gh api graphql -f query='
  mutation {
    addProjectV2ItemById(input: {
      projectId: "PROJECT_ID_PLACEHOLDER"
      contentId: "'$ISSUE_ID'"
    }) { item { id } }
  }
' -q '.data.addProjectV2ItemById.item.id')

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

## ISSUE LIFECYCLE & TRANSITIONS

```
Backlog → Ready → In Progress → In Review → Done
```

| From → To | Trigger | Status Option ID | Where |
|---|---|---|---|
| Backlog → Ready | Refinement complete | `READY_OPTION_ID_PLACEHOLDER` | `issue-refine` |
| Ready → In Progress | Implementation starts | `IN_PROGRESS_OPTION_ID_PLACEHOLDER` | `develop-issue` Phase 1 |
| In Progress → In Review | PR marked ready | `IN_REVIEW_OPTION_ID_PLACEHOLDER` | `develop-issue` Phase 4 (Step 3H) |
| In Review → Done | PR merged | `DONE_OPTION_ID_PLACEHOLDER` | GitHub Actions (or manual) |

Each transition uses the **Update status** mutation above. For Ready → In Progress and In Progress → In Review, fetch `PROJECT_ITEM_ID` with "Get existing project item ID" first.

---

## ERROR HANDLING

### GraphQL mutation fails

`error: GraphQL error: Resource not accessible by integration` — usually wrong field/project/option ID, or permissions.

Verify IDs against the live project (field-discovery query below). Update placeholders. Manual fallback: open the project board in a browser and update by hand.

### Issue already in project

Use "Get existing project item ID" to recover the existing `PROJECT_ITEM_ID`, then update status/iteration.

### Robust update helper

```bash
update_issue_status() {
  local issue_number=$1
  local new_status_id=$2
  local status_name=$3

  local issue_node_id
  issue_node_id=$(gh issue view "$issue_number" --json id --jq .id) || return 1

  local project_item_id
  project_item_id=$(gh api graphql -f query='
    query {
      node(id: "'$issue_node_id'") {
        ... on Issue { projectItems(first: 10) { nodes { id project { id } } } }
      }
    }
  ' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id')

  if [ -z "$project_item_id" ]; then
    project_item_id=$(gh api graphql -f query='
      mutation {
        addProjectV2ItemById(input: {
          projectId: "PROJECT_ID_PLACEHOLDER"
          contentId: "'$issue_node_id'"
        }) { item { id } }
      }
    ' --jq .data.addProjectV2ItemById.item.id) || return 1
  fi

  local result
  result=$(gh api graphql -f query='
    mutation {
      updateProjectV2ItemFieldValue(input: {
        projectId: "PROJECT_ID_PLACEHOLDER"
        itemId: "'$project_item_id'"
        fieldId: "STATUS_FIELD_ID_PLACEHOLDER"
        value: { singleSelectOptionId: "'$new_status_id'" }
      }) { projectV2Item { id } }
    }
  ' 2>&1)

  echo "$result" | grep -q "errors" && { echo "[ERROR] $result"; return 1; }
  echo "[OK] Issue #$issue_number → $status_name"
}
```

---

## FIELD-DISCOVERY QUERY

Run once per project to obtain the real IDs, then update this file's placeholders:

```bash
gh api graphql -f query='
  query {
    organization(login: "<your-org>") {
      projectV2(number: <n>) {
        id
        fields(first: 20) {
          nodes {
            ... on ProjectV2Field { id name }
            ... on ProjectV2SingleSelectField { id name options { id name } }
            ... on ProjectV2IterationField { id name configuration { iterations { id title startDate duration } } }
          }
        }
      }
    }
  }
'
```

---

## REFERENCES

- GitHub Projects V2 API: https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-api-with-projects
- GraphQL fields: https://docs.github.com/en/graphql
- gh CLI GraphQL: https://cli.github.com/manual/gh_api
