---
name: issue-refine
description: Refine GitHub issues by creating WHAT/WHY/HOW sections and preparing them for implementation. Adds issues to GitHub Projects V2, sets status to "Ready", and manages iterations.
argument-hint: "[issue-number|issue-url] [--iteration=<name>] [--analyze-service=<service>]"
--- # GITHUB ISSUE REFINEMENT COMMAND **Purpose:** Transform GitHub issues into refined specifications with WHAT/WHY/HOW sections, then automatically:
- Add issue to your project GitHub Project
- Set status to "Ready"
- Assign to current iteration (or specified iteration)
- Optionally delegate service-specific analysis --- ## WORKFLOW 1. **Parse Arguments**: Extract issue number/URL and optional parameters
2. **Validate Issue**: Confirm issue exists and retrieve full details
3. **Check Refinement Status**: Is issue already refined?
4. **Detect Services**: Identify which service(s) need analysis
5. **Optionally Delegate**: If user wants, delegate to service-specific agents
6. **Refine Issue**: Create/update WHAT/WHY/HOW sections
7. **GitHub Project Integration**: Update project status, iteration, and add to board
8. **Complete**: Confirm all actions and present summary --- ## STEP 1: PARSE ARGUMENTS Extract from user input:
- **Issue number or URL**: `123` or `https://github.com/org/repo/issues/123`
- **Optional flags:** - `--iteration=<name>` - Specify iteration (defaults to current: Iteration 2) - `--analyze-service=<service>` - Service to analyze: `backend`, `frontend`, `cli`, `website`, `docs-site` - `--no-project` - Skip GitHub Projects integration (test mode) Example:
```
/github-issue-refining 456 --analyze-service=backend
/github-issue-refining https://github.com/your-org/your-project/issues/123
/github-issue-refining 789 --iteration="Iteration 3"
``` --- ## STEP 2: VALIDATE ISSUE Use GitHub CLI to fetch issue details: ```bash
gh issue view <issue-number> --json title,body,number,url,labels
``` **Check:**
- Issue exists
- Has title and description
- Extract current labels
- Note any existing WHAT/WHY/HOW markers --- ## STEP 3: CHECK REFINEMENT STATUS Read issue body and comments. Look for indicators:
- ✅ Has `## WHAT` section
- ✅ Has `## WHY` section
- ✅ Has `## HOW` section **Decision:**
- If REFINED (has all 3 sections): - Log: `[STATUS] Issue already refined, skipping refinement` - Proceed to GitHub Project integration (Step 7) - If NOT REFINED: - Log: `[REFINE] Issue needs refinement, analyzing...` - Proceed to service detection (Step 4) --- ## STEP 4: DETECT SERVICES Analyze issue title, description, and labels to identify affected services: ```
IF issue mentions "backend" OR "API" OR "database" OR "Go" OR "server": → Add "backend service" to services list IF issue mentions "frontend" OR "React" OR "UI" OR "component": → Add "frontend" to services list IF issue mentions "CLI" OR "command-line" OR "terminal": → Add "cli" to services list IF issue mentions "website" OR "marketing" OR "landing": → Add "website" to services list IF issue mentions "docs" OR "documentation" OR "Docusaurus": → Add "docs-site" to services list IF --analyze-service flag provided: → Override with specified service
``` Log detected services:
```
[DETECT] Services identified: backend service, frontend
``` --- ## STEP 5: OPTIONAL SERVICE DELEGATION **Only if issue is NOT yet refined:** Ask user if they want service-specific analysis: ```markdown
I detected this issue affects: **backend service, frontend** Would you like me to analyze each service for additional context?
This will help create more detailed implementation specs. Options:
- Type `y` or `yes` - Analyze all detected services
- Type `n` or `no` - Skip service analysis (faster)
- Type `backend` / `frontend` / etc - Analyze specific services only
- Type `all` - Analyze all services in the project Your choice?
``` **If user agrees**, delegate to appropriate agents: ```
For backend analysis:
Task tool with:
- subagent_type: backend
- prompt: "Analyze the codebase for context on this issue: {issue_title}. Focus on affected modules, current implementations, patterns to follow, and dependencies that should be considered in the implementation plan. Return architectural context, file locations, and integration points." For frontend analysis:
Task tool with:
- subagent_type: frontend
- prompt: "Analyze the codebase for context on this issue: {issue_title}. Focus on affected components, current implementations, UI patterns, state management considerations, and integration points. Return architectural context, component locations, and dependencies." For CLI analysis:
Task tool with:
- subagent_type: cli-developer (research mode)
- prompt: "Research the codebase context for this issue: {issue_title}. Focus on command structure, existing patterns, argument handling, and integration points with backend." For website/docs-site:
Follow similar patterns, focusing on their specific tech stacks.
``` Collect results and merge into refinement context. Log: `[DELEGATE] Service analysis completed, proceeding to refinement` --- ## STEP 6: REFINE ISSUE **Mandatory First Step:** Read the refinement skill file:
```
.claude/skills/phase1_refinement.md
``` **Core refinement steps:** 1. **Analyze issue deeply:** - Read full issue description and all comments - Understand problem domain - Identify current patterns in codebase - List affected components 2. **Create WHAT section** (use template from skill file) - Clear one-line summary - Specific scope items - 3-6 testable acceptance criteria - Affected components list - Out-of-scope clarifications 3. **Create WHY section** (use template from skill file) - Business value statement - Problem description - Expected impact across dimensions - Related issues and dependencies 4. **Create HOW section** (use template from skill file) - Technical approach overview - Implementation details by component - Specific file paths and function names - Testing strategy (unit, integration) - Config/dependency changes - Risk identification and mitigation - Complexity assessment 5. **Format for GitHub:** ```markdown --- ## 🔍 ISSUE REFINEMENT [WHAT section] [WHY section] [HOW section] --- ``` 6. **Update GitHub issue body (NOT comment):** **CRITICAL:** Always update the issue BODY directly using `gh issue edit --body` or `--body-file`. NEVER use `gh issue comment` for refinement content. **Recommended Approach - Using temporary file:** ```bash ISSUE_NUMBER=<issue-number> TEMP_FILE="/tmp/refined_issue_${ISSUE_NUMBER}.md" # Fetch current body to preserve original description CURRENT_BODY=$(gh issue view $ISSUE_NUMBER --json body -q '.body') # Write updated body to temp file (append refinement to original) cat > "$TEMP_FILE" << 'EOF'
${CURRENT_BODY} ---
## 🔍 ISSUE REFINEMENT {formatted refinement with complete WHAT/WHY/HOW sections} ---
**Refined by:** AI Issue Refiner
**Date:** {today}
**Status:** Ready for GitHub Projects
EOF # Update the issue body (replaces entire body with appended content) gh issue edit $ISSUE_NUMBER --body-file "$TEMP_FILE" # Verify the update succeeded if gh issue view $ISSUE_NUMBER --json body -q '.body' | grep -q "ISSUE REFINEMENT"; then echo "[SUCCESS] Issue body updated with refinement" else echo "[ERROR] Failed to update issue body" exit 1 fi # Clean up temp file rm "$TEMP_FILE" ``` **Alternative - Direct inline update (for simpler cases):** ```bash # Fetch current body CURRENT_BODY=$(gh issue view <issue-number> --json body -q '.body') # Append refinement to existing content UPDATED_BODY="$CURRENT_BODY --- ## 🔍 ISSUE REFINEMENT {formatted refinement} --- **Refined by:** AI Issue Refiner **Date:** $(date +%Y-%m-%d) **Status:** Ready for GitHub Projects " # Update issue body gh issue edit <issue-number> --body "$UPDATED_BODY" ``` 7. **Add labels:** ```bash gh issue edit <issue-number> \ --add-label refined \ --add-label complexity:<assessed-level> ``` Log: `[REFINE] Issue body updated with WHAT/WHY/HOW sections (not comment)` --- ## STEP 7: GITHUB PROJECT INTEGRATION **Read helper skill file:**
```
.claude/skills/common/github-project-integration.md
``` ### 7.1 Add Issue to your project Project Use GraphQL mutation to add issue to project: ```bash
gh api graphql -f query='mutation { addProjectV2ItemById(input: { projectId: "PROJECT_ID_PLACEHOLDER" contentId: "{issue-node-id}" }) { item { id } }
}'
``` **Get issue node ID:**
```bash
gh issue view <issue-number> --json id,number
``` Log: `[PROJECT] Issue added to your project project` ### 7.2 Update Status to "Ready" Use GraphQL mutation to set status field: ```bash
gh api graphql -f query='mutation { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "{item-id}" fieldId: "STATUS_FIELD_ID_PLACEHOLDER" value: { singleSelectOptionId: "e18bf179" } }) { projectV2Item { id } }
}'
``` Log: `[PROJECT] Status updated to "Ready"` ### 7.3 Set Iteration Detect current iteration based on date (today: 2026-01-25): ```
Current iteration: Iteration 2 (started 2026-01-19, ends 2026-02-02) IF --iteration flag provided: → Use specified iteration ID
ELSE: → Auto-detect current iteration based on date
``` Available iterations:
- Iteration 2: 2026-01-19 to 2026-02-02 (ID: 54cf5c95)
- Iteration 3: 2026-02-02 to 2026-02-16 (ID: d2c335bc)
- Iteration 4: 2026-02-16 to 2026-03-02 (ID: b6a8f1bb)
- Iteration 5: 2026-03-02 to 2026-03-16 (ID: 955c1297) Use GraphQL mutation to set iteration: ```bash
gh api graphql -f query='mutation { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "{item-id}" fieldId: "ITERATION_FIELD_ID_PLACEHOLDER" value: { iterationId: "{iteration-id}" } }) { projectV2Item { id } }
}'
``` Log: `[PROJECT] Iteration set to "Iteration 2"` --- ## ERROR HANDLING **Issue not found:**
```
[ERROR] Issue #<number> not found. Verify issue number and try again.
``` **Issue already refined:**
```
[INFO] Issue #<number> is already refined.
Skipping refinement, proceeding to GitHub Projects integration only.
``` **GitHub API errors:**
```
[ERROR] GitHub API error: {error_message}
Check authentication: gh auth status
Try again with: /github-issue-refining <issue-number>
``` **GraphQL mutation failures:**
```
[ERROR] Failed to update GitHub Projects: {error_details}
Verify project IDs and field IDs are correct.
Manual update required: https://github.com/orgs/your-org/projects/5
``` --- ## LOGGING FORMAT ```
[START] Issue refinement workflow: #<number>
[PARSE] Issue URL parsed: {url}
[VALIDATE] Issue found: {title}
[DETECT] Services identified: {list}
[REFINE] Analyzing issue...
[REFINE] WHAT section created
[REFINE] WHY section created
[REFINE] HOW section created
[UPDATE] Issue updated with refinement
[LABEL] Added labels: refined, complexity:{level}
[PROJECT] Added to your project project
[PROJECT] Status updated to "Ready"
[PROJECT] Iteration set to "{iteration-name}"
[COMPLETE] Issue #<number> refinement complete! Summary:
- Complexity: {level}
- Components affected: {count}
- Iteration: {name}
- Project: your project (Ready)
``` --- ## QUALITY CHECKLIST Before completing, verify: **GitHub Issue Update:**
- [ ] WHAT section present (scope, criteria, components)
- [ ] WHY section present (value, problem, impact)
- [ ] HOW section present (approach, details, risks)
- [ ] Labels applied (refined, complexity)
- [ ] **Issue BODY updated with refinement (NOT posted as comment)**
- [ ] Verification: `gh issue view <number> --json body` shows refinement in body **GitHub Project Integration:**
- [ ] Issue added to your project project
- [ ] Status field set to "Ready"
- [ ] Iteration assigned correctly
- [ ] Project link accessible --- ## EXAMPLES **Example 1: Refine and integrate a backend issue**
```
/github-issue-refining 456 --analyze-service=backend
``` Result:
```
[START] Issue refinement workflow: #456
[VALIDATE] Issue found: "Add user role-based access control"
[DETECT] Services identified: backend service
[DELEGATE] Analyzing backend context...
[REFINE] WHAT/WHY/HOW sections created
[UPDATE] Issue #456 updated with refinement
[PROJECT] Added to your project, status=Ready, iteration=Iteration 2
[COMPLETE] Ready for implementation!
``` **Example 2: Refine existing refined issue**
```
/github-issue-refining 789
``` Result:
```
[VALIDATE] Issue found: "Implement search optimization"
[INFO] Issue already refined, skipping refinement
[PROJECT] Added to your project, status=Ready, iteration=Iteration 2
[COMPLETE] Project integration complete!
``` **Example 3: Refine with custom iteration**
```
/github-issue-refining 234 --iteration="Iteration 3"
``` Result:
```
[VALIDATE] Issue found: "Fix payment processing bug"
[REFINE] WHAT/WHY/HOW sections created
[PROJECT] Added to your project, status=Ready, iteration=Iteration 3
[COMPLETE] Ready for implementation!
``` --- ## KEY DIFFERENCES FROM WORKFLOW | Aspect | Workflow (`gh_issue_to_pr`) | Refining Command (`github-issue-refining`) |
|--------|---------------------------|------------------------------------------|
| Purpose | Full issue → PR pipeline | Issue refinement only |
| Phase 1 | Included | Primary focus |
| Phase 2-4 | Included | Not included |
| User Permission | Asked after Phase 1 | N/A (refinement only) |
| GitHub Project | Not handled | Full integration |
| Iteration | Not managed | Automatically assigned |
| Service Delegation | For implementation | Optional for analysis |
| Complexity | 50+ min typical | 5-10 min typical | --- ## NEXT STEPS AFTER REFINEMENT Once issue is refined and added to GitHub Projects:
- User can review refinement on GitHub
- Issue appears on project board with "Ready" status
- User can: - Start implementation immediately using `/workflows:gh_issue_to_pr` - Request additional analysis - Update iteration or status manually --- ## CRITICAL NOTES 1. **Issue Body vs Comment**: **ALWAYS** update issue body using `gh issue edit --body` or `--body-file`. NEVER use `gh issue comment` for refinement content. This is a critical requirement.
2. **Skill File Required**: Always read `.claude/skills/phase1_refinement.md` before creating WHAT/WHY/HOW
3. **GitHub Projects V2 API**: Uses GraphQL, requires correct field IDs
4. **Service-Specific Analysis**: Optional but recommended for better refinement
5. **Iteration Auto-Detection**: Uses today's date (2026-01-25) to select current iteration
6. **Error Recovery**: If GraphQL mutation fails, provide manual fallback link
7. **No Permission Gate**: This command completes automatically (unlike the workflow which asks for permission to implement) --- ## ANTI-PATTERNS ❌ **CRITICAL: Using `gh issue comment` for refinement** - ALWAYS use `gh issue edit --body` instead
❌ Skipping skill file reading
❌ Using wrong field IDs for GraphQL
❌ Not handling already-refined issues
❌ Ignoring service detection
❌ Proceeding without issue validation
❌ Not logging all actions clearly
❌ Not verifying issue body was updated successfully --- ## FILES INVOLVED **Command dispatcher:**
- `.claude/commands/workflows/github-issue-refining.md` (this file) **Shared skills:**
- `.claude/skills/phase1_refinement.md` - WHAT/WHY/HOW templates
- `.claude/skills/common/github-project-integration.md` - GraphQL helpers **Related files:**
- `.claude/commands/workflows/gh_issue_to_pr.md` - Full workflow reference
