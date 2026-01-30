---
name: phase4-pr
description: Create comprehensive PR description and handoff for human review after all RL loop quality gates pass. Final phase of autonomous development workflow.
argument-hint: "[pr-number]"
--- # PHASE 4: HUMAN REVIEW HANDOFF SKILL **Purpose:** Prepare a perfect PR for human review focused on business logic validation. --- ## PREREQUISITES Before starting Phase 4:
- ✅ All RL loop quality gates passed
- ✅ All tests passing
- ✅ No blocker review comments
- ✅ All acceptance criteria met --- ## PRE-PR QUALITY ASSURANCE DELEGATION Before creating the PR, delegate quality reviews based on the nature of changes: ### Security Review (Required for sensitive changes) **Trigger:** Changes to authentication, authorization, payments, user data, API keys, or secrets handling. ```
Task tool with:
- subagent_type: security-guardian
- prompt: | Review these changes for security vulnerabilities: Changed files: {list changed files} Focus on: - Authentication/authorization flaws - Input validation gaps - Sensitive data exposure - API security issues Report any Critical/High severity issues.
``` ### Architecture Review (Required for structural changes) **Trigger:** New patterns introduced, service structure changes, major refactoring, new integrations. ```
Task tool with:
- subagent_type: architecture-guardian
- prompt: | Review architectural consistency of these changes: Changed files: {list changed files} Focus on: - Pattern consistency with existing code - Separation of concerns - Service boundaries - MVP-appropriate complexity Report any architectural concerns.
``` ### Bug Finding (Recommended for complex features) **Trigger:** Complex business logic, multi-step workflows, data transformations. ```
Task tool with:
- subagent_type: bug-finder
- prompt: | Scan for bugs and edge cases in: Changed files: {list changed files} Focus on: - Logic errors - Edge case handling - Race conditions - Error handling gaps Report any bugs found with severity.
``` ### Handling QA Results - **Critical/High issues:** Must be fixed before proceeding
- **Medium issues:** Document in PR, fix if simple
- **Low issues:** Note for future improvement --- ## STEP 1: CREATE PR DESCRIPTION ### Use This Template ```markdown
## Summary
[2-3 sentence clear description of what was implemented] This PR implements [feature/fix] as described in issue #[number].
[Brief explanation of the approach taken and why]. ## Addresses Issue
Closes #[issue-number] --- ## What Changed ### Files Created
- `src/path/to/new_file.py` - [Purpose: e.g., "Elasticsearch client with connection pooling"]
- `tests/test_new_file.py` - [Purpose: e.g., "Unit tests for search functionality"]
- `config/elasticsearch.yaml` - [Purpose: e.g., "ES configuration with defaults"] ### Files Modified
- `src/path/to/existing.py` - [What changed: e.g., "Integrated ElasticsearchClient into search endpoint"]
- `README.md` - [What changed: e.g., "Added search API documentation"]
- `requirements.txt` - [What changed: e.g., "Added elasticsearch==8.10.0 dependency"] ### Key Changes
1. **[Component/Feature 1]**: [Brief description of change]
2. **[Component/Feature 2]**: [Brief description of change]
3. **[Component/Feature 3]**: [Brief description of change] --- ## Why These Changes ### Business Value
[From WHY section - explain the business value being delivered] ### Problem Solved
[From WHY section - what problem this solves] ### Expected Impact
- **User Experience:** [How users benefit]
- **Performance:** [Performance improvements, if any]
- **Maintainability:** [How this improves code quality]
- **[Other impacts]:** [Additional benefits] --- ## How It Works ### Technical Approach
[From HOW section - high-level explanation of the implementation] ### Architecture
[Brief explanation of how components interact] ```
[Optional: ASCII diagram or description of architecture] User Request → API Endpoint → Search Service → Elasticsearch ↓ Redis Cache
``` ### Key Implementation Details
1. **[Detail 1]**: [Explanation]
2. **[Detail 2]**: [Explanation]
3. **[Detail 3]**: [Explanation] --- ## Testing ### Test Coverage
- **Unit tests:** [X] new tests added
- **Integration tests:** [Y] scenarios covered
- **Total test count:** [Z] tests
- **Code coverage:** [N]% (target: [M]%) ### Test Scenarios Covered
✅ Happy path: [Brief description]
✅ Edge case: [Brief description]
✅ Error handling: [Brief description]
✅ Performance: [Brief description] ### How to Test Manually
1. [Step 1 - e.g., "Start the application: `python main.py`"]
2. [Step 2 - e.g., "Send request: `curl http://localhost:8000/search?q=test`"]
3. [Step 3 - e.g., "Verify response contains results array"]
4. [Expected outcome] --- ## Acceptance Criteria Met [Copy from WHAT section and check off each one] - [x] [Criterion 1 from WHAT section]
- [x] [Criterion 2 from WHAT section]
- [x] [Criterion 3 from WHAT section]
- [x] [Criterion 4 from WHAT section] **All acceptance criteria from the original issue have been met.** ✅ --- ## Configuration & Deployment ### New Environment Variables
```bash
# Required
ELASTICSEARCH_URL=http://localhost:9200 # Elasticsearch cluster endpoint # Optional
SEARCH_CACHE_TTL=300 # Cache duration in seconds (default: 300)
SEARCH_MAX_RESULTS=100 # Max results per query (default: 100)
``` ### New Dependencies
- `elasticsearch==8.10.0` - Official Elasticsearch Python client
- `redis==5.0.0` - For response caching ### Configuration Files Changed
- `config/elasticsearch.yaml` - ES cluster settings
- `.env.example` - Added new environment variable examples ### Deployment Notes
- [ ] Update production environment variables
- [ ] Ensure Elasticsearch cluster is accessible
- [ ] Redis instance must be running for caching
- [ ] Run database migrations (if applicable): `python manage.py migrate` ### Breaking Changes
[None] OR [List any breaking changes and migration steps] --- ## Security Considerations ✅ **Security scan:** No vulnerabilities detected
✅ **Authentication:** [Uses existing auth / No changes to auth]
✅ **Input validation:** [All user inputs validated and sanitized]
✅ **Data exposure:** [No sensitive data exposed in logs/errors] [Any specific security notes or considerations] --- ## Performance ### Benchmarks
- **API latency:** [X]ms (target: <[Y]ms) ✅
- **Memory usage:** [X]MB (target: <[Y]MB) ✅
- **Database queries:** [X] queries per request ### Performance Improvements
[None] OR [List performance improvements achieved] ### Known Limitations
[None] OR [List any known performance limitations] --- ## Screenshots / Demos [If applicable - especially for UI changes, API responses, etc.] ```json
Example API response:
{ "results": [ {"id": 1, "name": "Result 1"}, {"id": 2, "name": "Result 2"} ], "total": 2, "query": "test"
}
``` --- ## AI Autonomous Development Statistics **Workflow Execution:**
- **Issue refinement status:** [Already refined / Refined in Phase 1]
- **Code generation time:** ~[X] minutes
- **RL loop iterations:** [N] iterations
- **Issues auto-resolved:** [Y] - [Type of issue 1]: [count] - [Type of issue 2]: [count]
- **Total time to PR:** ~[Z] minutes
- **Quality gates:** All passed ✅ **RL Loop Breakdown:**
- Iteration 1: [What was fixed]
- Iteration 2: [What was fixed]
- [Continue if more iterations] --- ## Checklist **Code Quality:**
- [x] All tests passing
- [x] Code coverage meets/exceeds target
- [x] Linting and formatting passing
- [x] Type checking passing (if applicable)
- [x] No security vulnerabilities **Documentation:**
- [x] Code documented (docstrings, comments)
- [x] README updated (if needed)
- [x] API documentation updated (if needed)
- [x] Configuration documented **Testing:**
- [x] Unit tests for new functionality
- [x] Integration tests for workflows
- [x] Edge cases covered
- [x] Error handling tested **Process:**
- [x] All acceptance criteria met
- [x] Issue linked with "Closes #X"
- [x] Follows project conventions
- [x] No breaking changes (or documented)
- [x] Deployment notes provided --- ## Reviewer Notes **Focus Areas for Review:**
1. **Business logic validation:** Does this actually solve the problem correctly?
2. **Edge cases:** Are there scenarios not covered in tests?
3. **Security:** Any security implications not considered?
4. **Architecture:** Does this fit well with existing architecture? **Questions for Reviewer:**
[Optional: Any specific questions or areas where you want reviewer input] --- ## Related PRs / Issues - Related to: #[issue-number]
- Depends on: #[pr-number] (if any)
- Blocks: #[issue-number] (if any) --- **This PR is ready for human review.** All automated quality gates have passed.
Please review for business logic correctness and architectural fit.
``` --- ## STEP 2: CUSTOMIZE THE TEMPLATE **Fill in ALL sections:**
- Don't leave placeholders
- Don't skip sections
- Be specific and concrete
- Reference the WHAT/WHY/HOW from the issue **Tips for each section:** **Summary:**
- Keep it brief (2-3 sentences)
- Focus on WHAT was done
- Mention WHY briefly **What Changed:**
- List ALL files created/modified
- Explain purpose of each
- Be exhaustive **Why These Changes:**
- Pull directly from WHY section of issue
- Make the business case clear
- Help reviewer understand importance **How It Works:**
- Pull from HOW section of issue
- Explain the technical approach
- Can include diagrams if helpful **Testing:**
- Be specific about what's tested
- Show coverage numbers
- Provide manual testing steps **Acceptance Criteria:**
- Copy EXACTLY from WHAT section
- Check off each one
- Ensure ALL are truly met --- ## STEP 3: FINALIZE PR METADATA ### Link the Issue **In PR description:**
```markdown
Closes #123
``` **Why this matters:**
- Automatically links PR to issue
- Auto-closes issue when PR merges
- Creates traceability ### Add Labels **Required labels:**
- `ai-generated` - Identifies this as AI-created
- `ready-for-review` - Signals it's ready for human review **From issue:**
- Copy complexity label: `complexity:low/medium/high`
- Copy type label: `feature`, `bug`, `enhancement`, etc. **Additional labels as appropriate:**
- `documentation` - If docs were updated
- `breaking-change` - If there are breaking changes
- `needs-migration` - If deployment requires migration ### Set Reviewers **Assign to:**
- Specific team member (if you know who)
- Team (if using GitHub teams)
- Default reviewer for the repository ### Set Milestone / Project **If applicable:**
- Add to current sprint/milestone
- Add to project board
- Set due date (if urgent) --- ## STEP 3A: CREATE DRAFT PR **CRITICAL:** Always create PR as DRAFT initially. Only mark as ready after all checks pass. ### Create Draft PR Command ```bash
# Create draft PR with comprehensive description
gh pr create --draft \ --title "feat: <brief-title-from-issue>" \ --body "$(cat <<'EOF'
<paste-the-pr-description-from-step-1>
EOF
)" \ --base main # Capture PR number for later use
PR_NUMBER=$(gh pr view --json number --jq .number)
echo "Draft PR #$PR_NUMBER created"
``` **Alternative - Using body file:**
```bash
# Write PR description to temp file
cat > /tmp/pr_description.md << 'EOF'
<paste-the-pr-description-from-step-1>
EOF # Create draft PR
gh pr create --draft \ --title "feat: <brief-title>" \ --body-file /tmp/pr_description.md \ --base main PR_NUMBER=$(gh pr view --json number --jq .number)
rm /tmp/pr_description.md
``` ### Add Labels to Draft PR ```bash
# Add required labels
gh pr edit $PR_NUMBER \ --add-label "ai-generated" \ --add-label "ready-for-review" \ --add-label "complexity:medium" # Match issue complexity
``` ### Verify Draft Status ```bash
# Confirm PR is in draft state
gh pr view $PR_NUMBER --json isDraft,title,state # Expected output:
# {
# "isDraft": true,
# "state": "OPEN",
# "title": "feat: Add user authentication"
# }
``` **Logging:**
```
[PHASE-4] [ACTION] Creating draft PR
[PHASE-4] [RESULT] Draft PR #<number> created
[PHASE-4] [ACTION] Adding labels to PR
``` **Why create as draft:**
- Signals work is not yet ready for human review
- Prevents premature reviews while CI runs
- Allows final verification before requesting review
- Industry best practice for AI-generated PRs --- ## STEP 3B: WAIT FOR CI CHECKS **Before marking PR as ready, verify all CI checks pass:** ```bash
# Check PR status including checks
gh pr view $PR_NUMBER --json statusCheckRollup # Wait for checks to complete (if still running)
while true; do STATUS=$(gh pr view $PR_NUMBER --json statusCheckRollup --jq '.statusCheckRollup[] | select(.conclusion != "SUCCESS") | .conclusion') if [ -z "$STATUS" ]; then echo "All checks passed ✅" break else echo "Waiting for checks to complete..." sleep 30 fi
done
``` **If any check fails:**
- Do NOT mark as ready
- Fix the issue (return to RL loop if needed)
- Wait for checks to pass --- ## STEP 3C: MARK PR AS READY **Only after all checks pass, mark the draft PR as ready for human review:** ```bash
# Mark PR as ready for review
gh pr ready $PR_NUMBER # Verify it's no longer a draft
gh pr view $PR_NUMBER --json isDraft --jq .isDraft
# Expected output: false
``` **Logging:**
```
[PHASE-4] [ACTION] All checks passed, marking PR as ready
[PHASE-4] [RESULT] PR #<number> marked as ready for review
``` --- ## STEP 3D: UPDATE ISSUE STATUS TO "IN REVIEW" **CRITICAL:** After PR is marked ready, update the GitHub Projects issue status to "In Review". ```bash
ISSUE_NUMBER=<issue-number>
ISSUE_NODE_ID=$(gh issue view $ISSUE_NUMBER --json id --jq .id) # Get project item ID (should exist from Phase 2)
PROJECT_ITEM_ID=$(gh api graphql -f query=' query { node(id: "'$ISSUE_NODE_ID'") { ... on Issue { projectItems(first: 10) { nodes { id project { id } } } } } }
' --jq '.data.node.projectItems.nodes[] | select(.project.id == "PROJECT_ID_PLACEHOLDER") | .id') # Update status to "In Review" (option ID: aba860b9)
gh api graphql -f query=' mutation { updateProjectV2ItemFieldValue(input: { projectId: "PROJECT_ID_PLACEHOLDER" itemId: "'$PROJECT_ITEM_ID'" fieldId: "STATUS_FIELD_ID_PLACEHOLDER" value: { singleSelectOptionId: "aba860b9" } }) { projectV2Item { id } } }
'
``` **Verification:**
```bash
# Verify status was updated
gh api graphql -f query=' query { repository(owner: "your-org", name: "your-project") { issue(number: '$ISSUE_NUMBER') { projectItems(first: 10) { nodes { fieldValueByName(name: "Status") { ... on ProjectV2ItemFieldSingleSelectValue { name } } } } } } }
' --jq '.data.repository.issue.projectItems.nodes[0].fieldValueByName.name'
# Expected output: "In Review"
``` **Logging:**
```
[PHASE-4] [ACTION] Updating issue #<number> status to "In Review"
[PHASE-4] [RESULT] Issue status updated - PR awaiting human review
``` **Why this matters:**
- Makes it clear on project board that issue is under review
- Prevents duplicate work
- Tracks progress through development lifecycle
- Enables reporting and metrics **See:** `.claude/skills/common/github-project-integration.md` for status IDs reference --- ## STEP 4: VERIFY PR STATUS **Before considering complete:** ### Check CI Status
```
All checks have passed ✅ ✓ Tests (12/12 passed) ✓ Linting ✓ Type checking ✓ Security scan ✓ Code coverage (87%) ✓ Performance benchmarks
``` **If any check is failing:**
- Do NOT mark as ready
- Fix the issue
- Return to RL loop if needed ### Check PR Display **Verify these are visible in PR:**
- All CI badges are green
- File changes are reasonable (not accidental deletions/additions)
- No merge conflicts
- Branch is up to date with main/master --- ## STEP 5: UPDATE THE ISSUE **Add comment to original issue:** ```markdown
✅ **Implementation Complete - Ready for Human Review** **PR Created:** #[pr-number] **Summary:**
This issue has been fully implemented following the refined WHAT/WHY/HOW specification. **Completion Details:**
- All acceptance criteria met: ✅
- All tests passing: ✅
- Code review feedback addressed: ✅
- Security scan clean: ✅
- Performance benchmarks met: ✅ **Statistics:**
- Implementation time: ~[X] minutes
- RL loop iterations: [N]
- Quality gates: All passed ✅
- Files changed: [+Y new, ~Z modified] **Next Steps:**
Please review PR #[pr-number] for:
1. Business logic correctness
2. Edge case coverage
3. Architectural fit
4. Any domain-specific considerations If approved, this issue will auto-close when the PR is merged. --- cc @[reviewer-username]
``` --- ## STEP 6: FINAL CHECKS **Complete this checklist:** - [ ] PR description is comprehensive and clear
- [ ] All sections of template filled in
- [ ] Issue linked with "Closes #X"
- [ ] All required labels applied
- [ ] Reviewer assigned
- [ ] PR marked as "Ready for review" (not draft)
- [ ] All CI checks are green
- [ ] No merge conflicts
- [ ] Comment added to original issue
- [ ] PR URL copied for reference --- ## STEP 7: STOP AND WAIT **Your work is COMPLETE.** ```
[PHASE-4] [COMPLETE] PR #[number] created and ready for review
[COMPLETE] Total workflow time: [X] minutes
[COMPLETE] Issue #[issue] → PR #[pr] ready for human validation
[STOP] Awaiting next assignment
``` **Do NOT:**
- ❌ Look for other issues to work on
- ❌ Start working on something else
- ❌ Report on backlog or suggest next work
- ❌ Make additional commits (unless reviewer requests) **DO:**
- ✅ Wait for human feedback
- ✅ Be ready to address review comments if asked
- ✅ Celebrate a job well done! 🎉 --- ## COMMON MISTAKES TO AVOID ❌ **Incomplete PR description** - Leaving sections blank or with placeholders
❌ **Not linking issue** - Forgetting "Closes #X"
❌ **Wrong labels** - Missing `ai-generated` or `ready-for-review`
❌ **Skipping issue comment** - Not updating the original issue
❌ **CI not green** - Marking ready when checks are still failing
❌ **Continuing work** - Starting on next issue instead of stopping
❌ **Generic description** - Copy-pasting template without customization --- ## EXAMPLE PR DESCRIPTIONS ### Example 1: Feature Implementation ```markdown
## Summary
Implemented Elasticsearch-based search functionality with Redis caching to
improve search performance from 3-5 seconds to under 200ms for 95th percentile. ## Addresses Issue
Closes #123 --- ## What Changed ### Files Created
- `src/services/search_service.py` - Elasticsearch client with connection pooling
- `src/jobs/indexer.py` - Background job for indexing existing records
- `tests/unit/test_search_service.py` - Unit tests for search functionality
- `tests/integration/test_search_integration.py` - Integration tests with actual ES
- `config/elasticsearch.yaml` - ES configuration with production defaults ### Files Modified
- `src/api/search.py` - Replaced SQLite FTS with ElasticsearchClient
- `requirements.txt` - Added elasticsearch==8.10.0 and redis==5.0.0
- `README.md` - Updated with search API documentation
- `.env.example` - Added ELASTICSEARCH_URL and SEARCH_CACHE_TTL [Continue with rest of template...]
``` ### Example 2: Bug Fix ```markdown
## Summary
Fixed pagination bug where search results were not respecting the limit parameter,
causing performance issues and excessive data transfer for large result sets. ## Addresses Issue
Closes #456 --- ## What Changed ### Files Modified
- `src/api/search.py` - Added limit parameter validation and enforcement
- `tests/test_search.py` - Added pagination test cases ### Key Changes
1. **Limit enforcement:** Search results now properly respect `limit` parameter
2. **Input validation:** Added validation for limit (1-100 range)
3. **Default limit:** Applied sensible default of 10 when not specified [Continue with rest of template...]
``` --- ## PR QUALITY CHECKLIST **Your PR description should:**
- [ ] Tell a clear story (Summary → What → Why → How)
- [ ] Be detailed enough that reviewer understands changes without reading code
- [ ] Reference the original issue's WHAT/WHY/HOW
- [ ] Include test coverage details
- [ ] Provide deployment/configuration instructions
- [ ] List all acceptance criteria and confirm they're met
- [ ] Show AI workflow statistics for transparency **Your PR should:**
- [ ] Have all CI checks passing
- [ ] Have all files properly formatted
- [ ] Have comprehensive tests
- [ ] Have clear commit history
- [ ] Have no merge conflicts
- [ ] Link to the original issue **Your process should:**
- [ ] Update the original issue with PR link
- [ ] Apply all required labels
- [ ] Assign appropriate reviewer
- [ ] Mark as ready for review
- [ ] Then STOP completely --- ## SUCCESS CRITERIA You've successfully completed Phase 4 when: ✅ PR description is comprehensive and accurate
✅ All metadata (labels, reviewers, links) configured
✅ All CI checks passing
✅ Original issue updated with completion status
✅ PR marked as ready for review
✅ **You have STOPPED and are waiting for next assignment** --- **Congratulations! You've completed the Autonomous Development Workflow.** 🎉 From a GitHub issue to a production-ready PR, you've:
- ✅ Refined requirements (or worked with refined issue)
- ✅ Generated high-quality code
- ✅ Iterated through RL loop until perfect
- ✅ Created comprehensive PR for human review Now rest. Wait for feedback. Be ready for the next challenge.
