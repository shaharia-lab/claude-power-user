---
name: github-project-integration
description: Shared skill for GitHub Projects V2 API operations. Includes GraphQL mutations for adding issues, updating status, and managing iterations.
user-invocable: false
--- # GITHUB PROJECTS V2 INTEGRATION SKILL **Purpose:** Provide reusable patterns and helpers for interacting with GitHub Projects V2 API via GraphQL. Used by:
- `/github-issue-refining` command
- Other automation that needs project management --- ## PROJECT CONFIGURATION ### your project Project (GitHub) ```json
{ "organization": "your-org", "project_url": "https://github.com/orgs/your-org/projects/5", "project_number": 5, "project_id": "PROJECT_ID_PLACEHOLDER", "repository": "your-project"
}
``` ### Fields | Field Name | Field ID | Type | Options |
|-----------|----------|------|---------|
| Status | STATUS_FIELD_ID_PLACEHOLDER | Single Select | See below |
| Iteration | ITERATION_FIELD_ID_PLACEHOLDER | Iteration | See below | ### Status Options | Status Name | Option ID | Description |
|------------|-----------|-------------|
| Backlog | f75ad846 | Not yet ready |
| Ready | e18bf179 | Refined and ready to implement |
| In progress | 47fc9ee4 | Currently being implemented |
| In review | aba860b9 | PR under review |
| Done | 98236657 | Completed and merged | ### Iterations | Iteration Name | Iteration ID | Start Date | Duration | End Date |
|--------------|------------|-----------|----------|----------|
| Iteration 2 | 54cf5c95 | 2026-01-19 | 14 days | 2026-02-02 |
| Iteration 3 | d2c335bc | 2026-02-02 | 14 days | 2026-02-16 |
| Iteration 4 | b6a8f1bb | 2026-02-16 | 14 days | 2026-03-02 |
| Iteration 5 | 955c1297 | 2026-03-02 | 14 days | 2026-03-16 | --- ## CURRENT DATE REFERENCE **Today:** 2026-01-25 **Current Iteration:** Iteration 2 (54cf5c95)
- Started: 2026-01-19
- Ends: 2026-02-02
- Days remaining: 8 --- ## ITERATION DETECTION ALGORITHM ```python
def get_current_iteration(today_date): """ Determine which iteration is currently active based on date. Input: date object (e.g., 2026-01-25) Output: (iteration_name, iteration_id) """ iterations = [ {"name": "Iteration 2", "id": "ITERATION_ID", "start": "2026-01-19", "end": "2026-02-02"}, {"name": "Iteration 3", "id": "ITERATION_ID", "start": "2026-02-02", "end": "2026-02-16"}, {"name": "Iteration 4", "id": "ITERATION_ID", "start": "2026-02-16", "end": "2026-03-02"}, {"name": "Iteration 5", "id": "ITERATION_ID", "start": "2026-03-02", "end": "2026-03-16"}, ] for iteration in iterations: if start_date <= today_date < end_date: return iteration["name"], iteration["id"] # Default to current if beyond defined range return "Iteration 2", "54cf5c95"
``` --- ## COMMON OPERATIONS ### Operation 1: Get Issue Node ID **Purpose:** Retrieve the GraphQL node ID needed for project mutations **Command:**
```bash
gh issue view <issue-number> \ --repo your-org/your-project \ --json id,number,title \ --jq '.id,.number,.title'
``` **Output:**
```
MDU6SXNzdWUxNjI3NTQ3NTA=
456
"My issue title"
``` **Store:** `issue_node_id = MDU6SXNzdWUxNjI3NTQ3NTA=` --- ### Operation 2: Add Issue to Project **Purpose:** Add an issue to the your project GitHub Project **Prerequisites:**
- Issue node ID (from Operation 1)
- Project ID: `PROJECT_ID_PLACEHOLDER` **GraphQL Mutation:**
```graphql
mutation AddIssueToProject { addProjectV2ItemById(input: { projectId: "PROJECT_ID_PLACEHOLDER" contentId: "<issue-node-id>" }) { item { id } }
}
``` **Bash Command:**
```bash
ISSUE_ID="<issue-node-id>"
gh api graphql -f query=' mutation AddIssueToProject { addProjectV2ItemById(input: { projectId: "PROJECT_ID_PLACEHOLDER" contentId: "'$ISSUE_ID'" }) { item { id } } }
' -q '.data.addProjectV2ItemById.item.id'
``` **Response:**
```
PVTI_lADOB8xah84BL9hBzg7YmA
``` **Store:** `project_item_id = PVTI_lADOB8xah84BL9hBzg7YmA` **Error Handling:**
```bash
if [ $? -ne 0 ]; then echo "[ERROR] Failed to add issue to project" echo "Manual fallback: https://github.com/orgs/your-org/projects/5" exit 1
fi
``` --- ### Operation 3: Update Status Field **Purpose:** Change issue status on project board (e.g., Backlog → Ready) **Prerequisites:**
- Project item ID (from Operation 2)
- Status option ID (from table above)
- Project ID: `PROJECT_ID_PLACEHOLDER`
- Status field ID: `STATUS_FIELD_ID_PLACEHOLDER` **GraphQL Mutation:**
```graphql
mutation UpdateStatus { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "<project-item-id>" fieldId: "STATUS_FIELD_ID_PLACEHOLDER" value: { singleSelectOptionId: "<status-option-id>" } }) { projectV2Item { id } }
}
``` **Bash Command (to set status to "Ready"):**
```bash
ITEM_ID="<project-item-id>"
gh api graphql -f query=' mutation UpdateStatus { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "'$ITEM_ID'" fieldId: "STATUS_FIELD_ID_PLACEHOLDER" value: { singleSelectOptionId: "e18bf179" } }) { projectV2Item { id } } }
'
``` **Status Option IDs:**
```bash
# Backlog
STATUS_ID="f75ad846" # Ready
STATUS_ID="e18bf179" # In progress
STATUS_ID="47fc9ee4" # In review
STATUS_ID="aba860b9" # Done
STATUS_ID="98236657"
``` **Logging:**
```bash
echo "[PROJECT] Status updated to 'Ready'"
``` --- ### Operation 4: Update Iteration Field **Purpose:** Assign issue to an iteration on the project board **Prerequisites:**
- Project item ID (from Operation 2)
- Iteration ID (from table above, or use detection algorithm)
- Project ID: `PROJECT_ID_PLACEHOLDER`
- Iteration field ID: `ITERATION_FIELD_ID_PLACEHOLDER` **GraphQL Mutation:**
```graphql
mutation UpdateIteration { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "<project-item-id>" fieldId: "ITERATION_FIELD_ID_PLACEHOLDER" value: { iterationId: "<iteration-id>" } }) { projectV2Item { id } }
}
``` **Bash Command (to set to current iteration):**
```bash
ITEM_ID="<project-item-id>"
ITERATION_ID="54cf5c95" # Current: Iteration 2 gh api graphql -f query=' mutation UpdateIteration { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "'$ITEM_ID'" fieldId: "ITERATION_FIELD_ID_PLACEHOLDER" value: { iterationId: "'$ITERATION_ID'" } }) { projectV2Item { id } } }
'
``` **Iteration ID Reference:**
```bash
# Auto-detect current iteration (recommended)
ITERATION_ID=$(detect_current_iteration) # Or manually specify
ITERATION_ID="54cf5c95" # Iteration 2
ITERATION_ID="d2c335bc" # Iteration 3
ITERATION_ID="b6a8f1bb" # Iteration 4
ITERATION_ID="955c1297" # Iteration 5
``` **Logging:**
```bash
echo "[PROJECT] Iteration assigned to 'Iteration 2'"
``` --- ### Operation 5: Query Issue Status **Purpose:** Check if an issue is in "Ready" status for workflow processing **Prerequisites:**
- Issue number **GraphQL Query:**
```bash
gh api graphql -f query=' query { repository(owner: "your-org", name: "your-project") { issue(number: <issue-number>) { projectItems(first: 10) { nodes { fieldValueByName(name: "Status") { ... on ProjectV2ItemFieldSingleSelectValue { optionId name } } } } } } }
' -q '.data.repository.issue.projectItems.nodes[0].fieldValueByName | {optionId, name}'
``` **Bash Command:**
```bash
ISSUE_NUMBER=456
STATUS_RESULT=$(gh api graphql -f query=' query { repository(owner: "your-org", name: "your-project") { issue(number: '$ISSUE_NUMBER') { projectItems(first: 10) { nodes { fieldValueByName(name: "Status") { ... on ProjectV2ItemFieldSingleSelectValue { optionId name } } } } } } }
' -q '.data.repository.issue.projectItems.nodes[0].fieldValueByName') # Extract values
OPTION_ID=$(echo "$STATUS_RESULT" | jq -r '.optionId')
STATUS_NAME=$(echo "$STATUS_RESULT" | jq -r '.name') if [ "$OPTION_ID" = "e18bf179" ]; then echo "[PROJECT] Issue is Ready"
else echo "[PROJECT] Issue status is: $STATUS_NAME" exit 1
fi
``` **Response (if Ready):**
```json
{ "optionId": "e18bf179", "name": "Ready"
}
``` **Error Handling:**
```bash
if [ -z "$OPTION_ID" ] || [ "$OPTION_ID" = "null" ]; then echo "[ERROR] Could not determine issue status" echo "[INFO] Issue may not be in GitHub Projects" exit 1
fi
``` **Logging:**
```bash
echo "[PROJECT] Issue status verified: Ready ✅"
``` --- ## ORCHESTRATED WORKFLOW **Complete sequence to add issue to project:** ```bash
#!/bin/bash # 1. Get issue ID
ISSUE_NUMBER=$1
ISSUE_ID=$(gh issue view $ISSUE_NUMBER \ --repo your-org/your-project \ --json id \ --jq '.id') echo "[INFO] Issue node ID: $ISSUE_ID" # 2. Add to project
PROJECT_ITEM_ID=$(gh api graphql -f query=' mutation { addProjectV2ItemById(input: { projectId: "PROJECT_ID_PLACEHOLDER" contentId: "'$ISSUE_ID'" }) { item { id } } }
' -q '.data.addProjectV2ItemById.item.id') echo "[PROJECT] Added to your project (item ID: $PROJECT_ITEM_ID)" # 3. Update status to "Ready"
gh api graphql -f query=' mutation { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "'$PROJECT_ITEM_ID'" fieldId: "STATUS_FIELD_ID_PLACEHOLDER" value: { singleSelectOptionId: "e18bf179" } }) { projectV2Item { id } } }
' echo "[PROJECT] Status set to 'Ready'" # 4. Assign iteration (current)
ITERATION_ID="54cf5c95" # Iteration 2
gh api graphql -f query=' mutation { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "'$PROJECT_ITEM_ID'" fieldId: "ITERATION_FIELD_ID_PLACEHOLDER" value: { iterationId: "'$ITERATION_ID'" } }) { projectV2Item { id } } }
' echo "[PROJECT] Iteration assigned to 'Iteration 2'"
echo "[COMPLETE] Issue #$ISSUE_NUMBER fully integrated into your project project"
``` --- ## ERROR HANDLING ### Error: GraphQL Mutation Failed **Symptom:**
```
error: GraphQL error: Resource not accessible by integration
``` **Causes:**
- Incorrect field ID
- Incorrect project ID
- Incorrect option/iteration ID
- Permission issue **Resolution:**
```bash
# Verify field IDs
gh api graphql -f query=' query { organization(login: "your-org") { projectV2(number: 5) { id fields(first: 20) { nodes { ... on ProjectV2Field { id, name } ... on ProjectV2SingleSelectField { id, name, options { id, name } } ... on ProjectV2IterationField { id, name, configuration { iterations { id, title } } } } } } } }
'
``` Manual fallback:
```
https://github.com/orgs/your-org/projects/5
``` ### Error: Issue Not Found **Resolution:**
```bash
gh issue view <number> --repo your-org/your-project
# If not found, verify issue number and repository
``` ### Error: Issue Already in Project **Symptom:** GraphQL returns "ContentAlreadyAddedToProject" error **Resolution:**
```bash
# Query to get existing project item ID
ISSUE_ID="<issue-node-id>"
PROJECT_ITEM_ID=$(gh api graphql -f query=' query { node(id: "'$ISSUE_ID'") { ... on Issue { projectItems(first: 10) { nodes { id project { id number } } } } } }
' -q '.data.node.projectItems.nodes[] | select(.project.number == 5) | .id') # Then update status and iteration using this item ID
``` --- ## TESTING CHECKLIST Before deploying project integration code: - [ ] Test adding new issue to project
- [ ] Verify status updates correctly
- [ ] Verify iteration assigns correctly
- [ ] Test with already-integrated issue (should not duplicate)
- [ ] Test with custom iteration parameter
- [ ] Verify all error cases handled gracefully
- [ ] Test GraphQL queries independently via `gh api graphql`
- [ ] Confirm field IDs match current project configuration --- ## FIELD ID DISCOVERY If field IDs change or need verification, use this query: ```bash
gh api graphql -f query=' query { organization(login: "your-org") { projectV2(number: 5) { id title fields(first: 20) { nodes { ... on ProjectV2Field { id name } ... on ProjectV2SingleSelectField { id name options { id name } } ... on ProjectV2IterationField { id name configuration { iterations { id title startDate duration } } } } } } } }
'
``` Update this skill file if any IDs change. --- ## ISSUE LIFECYCLE & STATUS TRANSITIONS This section documents the complete issue lifecycle and when to trigger each status update. ### Lifecycle Stages ```
Issue Created ↓
[Backlog] - Issue exists but not refined ↓
Issue Refinement (/github-issue-refining) ↓
[Ready] - Refined with WHAT/WHY/HOW, ready for implementation ↓
Agent Starts Work (Phase 2 of /gh_issue_to_pr) ↓
[In Progress] - Implementation underway ↓
Code Complete + Draft PR Created ↓
All Checks Pass + PR Marked Ready ↓
[In Review] - Awaiting human review ↓
PR Merged ↓
[Done] - Completed and deployed
``` ### Status Transition Commands #### Transition 1: Backlog → Ready **Trigger:** Issue refinement complete via `/github-issue-refining` command
**Command:** Already handled by the refinement command
**Status Option ID:** `e18bf179` ```bash
# This is handled automatically by /github-issue-refining command
gh api graphql -f query='mutation { updateProjectV2ItemFieldValue(input: {projectId: "PROJECT_ID_PLACEHOLDER", itemId: "'$ITEM_ID'", fieldId: "STATUS_FIELD_ID_PLACEHOLDER", value: {singleSelectOptionId: "e18bf179"}}) { projectV2Item { id } } }'
``` #### Transition 2: Ready → In Progress **Trigger:** Agent begins implementation (Phase 2 start of `/gh_issue_to_pr`)
**Status Option ID:** `47fc9ee4`
**When:** Immediately before starting code generation ```bash
ISSUE_NUMBER=$1
ISSUE_NODE_ID=$(gh issue view $ISSUE_NUMBER --json id --jq .id) # Get or create project item
PROJECT_ITEM_ID=$(gh api graphql -f query=' mutation { addProjectV2ItemById(input: { projectId: "PROJECT_ID_PLACEHOLDER" contentId: "'$ISSUE_NODE_ID'" }) { item { id } } }
' --jq .data.addProjectV2ItemById.item.id 2>/dev/null || \ gh api graphql -f query=' query { node(id: "'$ISSUE_NODE_ID'") { ... on Issue { projectItems(first: 10) { nodes { id project { id } } } } } } ' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id') # Update to "In Progress"
gh api graphql -f query=' mutation { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "'$PROJECT_ITEM_ID'" fieldId: "STATUS_FIELD_ID_PLACEHOLDER" value: { singleSelectOptionId: "47fc9ee4" } }) { projectV2Item { id } } }
' echo "[PROJECT] Issue #$ISSUE_NUMBER status updated to 'In Progress'"
``` **Why this matters:**
- Signals work has begun
- Prevents duplicate work
- Makes progress visible on board #### Transition 3: In Progress → In Review **Trigger:** PR marked as ready for human review (Phase 4 end of `/gh_issue_to_pr`)
**Status Option ID:** `aba860b9`
**When:** After PR is marked ready using `gh pr ready` ```bash
ISSUE_NUMBER=$1
ISSUE_NODE_ID=$(gh issue view $ISSUE_NUMBER --json id --jq .id) # Get project item ID (should already exist from Phase 2)
PROJECT_ITEM_ID=$(gh api graphql -f query=' query { node(id: "'$ISSUE_NODE_ID'") { ... on Issue { projectItems(first: 10) { nodes { id project { id } } } } } }
' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id') # Update to "In Review"
gh api graphql -f query=' mutation { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "'$PROJECT_ITEM_ID'" fieldId: "STATUS_FIELD_ID_PLACEHOLDER" value: { singleSelectOptionId: "aba860b9" } }) { projectV2Item { id } } }
' echo "[PROJECT] Issue #$ISSUE_NUMBER status updated to 'In Review'"
``` **Why this matters:**
- Signals human review is needed
- Tracks PR review progress
- Separates implementation from review stages #### Transition 4: In Review → Done **Trigger:** PR merged to main
**Status Option ID:** `98236657`
**When:** Automatically after PR merge (can be automated via GitHub Actions) ```bash
# Usually handled by GitHub Actions on PR merge
# Manual command if needed:
gh api graphql -f query='mutation { updateProjectV2ItemFieldValue(input: {projectId: "PROJECT_ID_PLACEHOLDER", itemId: "'$ITEM_ID'", fieldId: "STATUS_FIELD_ID_PLACEHOLDER", value: {singleSelectOptionId: "98236657"}}) { projectV2Item { id } } }'
``` ### Quick Reference Table | From Status | To Status | Trigger | Command Location | Status Option ID |
|------------|-----------|---------|-----------------|-----------------|
| Backlog | Ready | Issue refinement complete | `/github-issue-refining` | `e18bf179` |
| Ready | In Progress | Agent starts implementation | `/gh_issue_to_pr` Phase 2 start | `47fc9ee4` |
| In Progress | In Review | PR marked ready for review | `/gh_issue_to_pr` Phase 4 end | `aba860b9` |
| In Review | Done | PR merged | GitHub Actions (or manual) | `98236657` | ### Error Handling for Transitions ```bash
#!/bin/bash
# Robust status update with error handling update_issue_status() { local issue_number=$1 local new_status_id=$2 local status_name=$3 echo "[PROJECT] Updating issue #$issue_number to '$status_name'..." local issue_node_id issue_node_id=$(gh issue view "$issue_number" --json id --jq .id 2>&1) if [ $? -ne 0 ]; then echo "[ERROR] Failed to get issue node ID: $issue_node_id" return 1 fi local project_item_id project_item_id=$(gh api graphql -f query=' query { node(id: "'$issue_node_id'") { ... on Issue { projectItems(first: 10) { nodes { id project { id } } } } } } ' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id' 2>&1) if [ -z "$project_item_id" ]; then echo "[ERROR] Issue not in project, adding it first..." project_item_id=$(gh api graphql -f query=' mutation { addProjectV2ItemById(input: { projectId: "PROJECT_ID_PLACEHOLDER" contentId: "'$issue_node_id'" }) { item { id } } } ' --jq .data.addProjectV2ItemById.item.id 2>&1) if [ $? -ne 0 ]; then echo "[ERROR] Failed to add issue to project: $project_item_id" return 1 fi fi local result result=$(gh api graphql -f query=' mutation { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "'$project_item_id'" fieldId: "STATUS_FIELD_ID_PLACEHOLDER" value: { singleSelectOptionId: "'$new_status_id'" } }) { projectV2Item { id } } } ' 2>&1) if echo "$result" | grep -q "errors"; then echo "[ERROR] GraphQL mutation failed: $result" return 1 fi echo "[SUCCESS] Issue #$issue_number updated to '$status_name'" return 0
} # Usage examples:
# update_issue_status 123 "47fc9ee4" "In Progress"
# update_issue_status 123 "aba860b9" "In Review"
``` --- ## REFERENCES - [GitHub Projects V2 API Docs](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-api-with-projects)
- [GraphQL Field Documentation](https://docs.github.com/en/graphql)
- [gh CLI GraphQL Reference](https://cli.github.com/manual/gh_api) --- ## CHANGELOG - 2026-01-25: Issue lifecycle and status transitions added - Added complete issue lifecycle documentation - Documented all status transitions (Ready → In Progress → In Review → Done) - Added status transition commands for each lifecycle stage - Added error handling for robust status updates - Added quick reference table for status transitions - 2026-01-25: Initial creation - Added core operations for project management - Documented all field and iteration IDs - Created orchestration example
