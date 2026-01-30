---
name: phase1-refinement
description: Transform ambiguous GitHub issues into refined specifications with WHAT, WHY, and HOW sections. Use before implementing features or fixes when issue lacks clear requirements.
argument-hint: "[issue-number]"
--- **NOTE:** As of 2026-01-25, this skill is primarily used by `/workflows:github-issue-refining` command. The `/workflows:gh_issue_to_pr` workflow no longer uses automatic refinement. Issues MUST be refined before running that workflow. --- # PHASE 1: PLANNING & REFINEMENT SKILL **Purpose:** Transform an ambiguous GitHub issue into a refined specification with WHAT, WHY, and HOW sections. --- ## WHEN TO USE THIS SKILL Execute Phase 1 ONLY when the assigned issue lacks proper WHAT/WHY/HOW structure. --- ## STEP 1: DEEP ANALYSIS ### Analyze the Issue
1. Read issue description and all comments thoroughly
2. Understand the problem domain and user intent
3. Identify ambiguities and missing information ### Analyze the Codebase
1. Examine related files and modules
2. Find similar implementations (patterns to follow)
3. Identify current architectural patterns
4. Review relevant dependencies
5. Check project conventions (style, testing, documentation) ### Identify Scope
1. List ALL affected components/modules
2. Determine integration points
3. Identify potential edge cases
4. Assess technical risks --- ## STEP 2: CREATE WHAT SECTION ### Template: ```markdown
## WHAT **Summary:** [One-line clear description] **Scope:**
- [Specific feature/functionality to implement]
- [Specific bug to fix]
- [Specific improvement to make] **Affected Components:**
- `component/module/path1` - [What changes here]
- `component/module/path2` - [What changes here] **Acceptance Criteria:**
- [ ] [Specific, testable criterion 1]
- [ ] [Specific, testable criterion 2]
- [ ] [Specific, testable criterion 3]
- [ ] [Specific, testable criterion 4] **Out of Scope:**
- [What is explicitly NOT included in this issue]
- [What will be handled separately]
``` ### WHAT Section Guidelines: **Scope:**
- Be specific and concrete
- Use action verbs (implement, fix, add, update, remove)
- Quantify where possible (e.g., "support up to 1000 concurrent users") **Acceptance Criteria:**
- Make them testable (can you write a test for it?)
- Use "Given/When/Then" format when helpful
- Include both functional and non-functional requirements
- Aim for 3-6 criteria (not too few, not too many) **Example - Good vs Bad:** ❌ Bad:
```
- Make the app faster
- Fix the bug
``` ✅ Good:
```
- [ ] API response time reduced to <200ms for 95th percentile
- [ ] Error handler catches and logs all database connection failures
- [ ] User sees clear error message when upload exceeds 10MB
``` --- ## STEP 3: CREATE WHY SECTION ### Template: ```markdown
## WHY **Business Value:**
[Why this matters to users or the business] **Problem Statement:**
[Current limitation, pain point, or issue that exists today] **Expected Impact:**
- User experience: [How UX improves]
- Performance: [How performance improves]
- Technical debt: [How this reduces debt]
- Risk mitigation: [How this reduces risk]
- [Other relevant impacts] **Context:**
[Any relevant background, history, related issues, or dependencies]
- Related to: #[issue-number]
- Depends on: #[issue-number]
- Blocks: #[issue-number]
``` ### WHY Section Guidelines: **Business Value:**
- Answer: "Why should anyone care?"
- Connect to user needs or business goals
- Be honest about priority **Problem Statement:**
- Describe current state clearly
- Explain consequences of NOT fixing
- Use concrete examples **Expected Impact:**
- Be realistic, not aspirational
- Quantify when possible
- Consider multiple dimensions (UX, performance, maintainability) **Example:** ```markdown
## WHY **Business Value:**
Users currently wait 3-5 seconds for search results, causing 40% to abandon
before seeing results. Fast search (<500ms) will improve engagement and
increase conversion by estimated 15-20%. **Problem Statement:**
Current search implementation does full-text scan on every request without
caching or indexing. As data grows (now 2M records), search becomes unusable. **Expected Impact:**
- User experience: Sub-second search results
- Performance: 10x faster queries via indexing
- Scalability: Can handle 10M+ records
- Cost reduction: Lower database load **Context:**
Related to #234 (database migration). Must be completed before Q2 launch.
``` --- ## STEP 4: CREATE HOW SECTION ### Template: ```markdown
## HOW **Technical Approach:**
[High-level strategy in 2-3 sentences] **Implementation Details:** ### 1. [Component/Area 1]
**File:** `path/to/file1.ext`
**Changes:**
- Add function `functionName()` to handle [specific task]
- Modify `existingFunction()` to [specific change]
- Add new class `ClassName` for [purpose] **New Code:**
- `class/function/method names` **Modified Code:**
- `existing class/function/method names` ### 2. [Component/Area 2]
**File:** `path/to/file2.ext`
**Changes:**
- [Specific changes] ### 3. Testing Strategy
**Unit Tests:**
- Test `function1()` with valid/invalid inputs
- Test `function2()` error handling
- Test edge case: [describe] **Integration Tests:**
- Test end-to-end flow: [describe]
- Test integration with [external service/component] **Test Files:**
- `tests/unit/test_file1.py`
- `tests/integration/test_integration.py` ### 4. Documentation Updates
- Update `README.md` section [which section]
- Add docstrings to new functions
- Update `docs/api.md` with new endpoints ### 5. Configuration Changes
**New Dependencies:**
- `package-name==version` - [why needed] **Environment Variables:**
- `NEW_VAR` - [purpose, default value] **Config Files:**
- Update `config/settings.yaml` - [what changes] **Files to Create:**
- `src/new_module.py` - [purpose]
- `tests/test_new_module.py` - [purpose] **Files to Modify:**
- `src/existing_file.py` - [what changes]
- `config/app.yaml` - [what changes] **Potential Risks:**
- **Risk:** [Describe risk] **Mitigation:** [How to handle]
- **Risk:** [Describe risk] **Mitigation:** [How to handle] **Estimated Complexity:** [Low / Medium / High]
**Reasoning:** [Why this complexity level]
``` ### HOW Section Guidelines: **Technical Approach:**
- Explain the "big picture" strategy
- Reference existing patterns in codebase
- Justify why this approach over alternatives **Implementation Details:**
- Be specific about file paths
- List exact function/class names to create/modify
- Specify what changes in each file
- Think through the implementation sequence **Testing Strategy:**
- Cover happy path, error cases, edge cases
- Specify what to test, not just "write tests"
- Consider integration points **Risks & Mitigation:**
- Be honest about technical risks
- Provide concrete mitigation strategies
- Consider backward compatibility **Complexity Assessment:**
- **Low:** Single file, <100 lines, no dependencies, clear pattern
- **Medium:** Multiple files, <500 lines, some new patterns, few integrations
- **High:** Many files, >500 lines, new architecture, complex integrations **Example:** ```markdown
## HOW **Technical Approach:**
Implement Elasticsearch for search indexing. Add background job to index
existing records. Replace current SQLite FTS with Elasticsearch queries. **Implementation Details:** ### 1. Search Service
**File:** `src/services/search_service.py`
**Changes:**
- Create `ElasticsearchClient` class with connection pooling
- Add `index_document(doc_id, content)` method
- Add `search(query, filters, limit)` method
- Add retry logic with exponential backoff **New Code:**
- `ElasticsearchClient` class
- `SearchQuery` dataclass ### 2. Background Indexing
**File:** `src/jobs/indexer.py`
**Changes:**
- Create `IndexerJob` to batch index existing records
- Process 1000 records at a time
- Add progress logging every 10k records ### 3. API Integration
**File:** `src/api/search.py`
**Changes:**
- Modify `search_endpoint()` to use `ElasticsearchClient`
- Remove old SQLite FTS code
- Add response caching (5 min TTL) ### 4. Testing Strategy
**Unit Tests:**
- Mock Elasticsearch client, test query building
- Test error handling when ES unavailable
- Test pagination logic **Integration Tests:**
- Test actual ES queries against test index
- Test indexing job with sample data
- Load test with 10k concurrent searches ### 5. Configuration
**New Dependencies:**
- `elasticsearch==8.10.0` - Official ES client
- `redis==5.0.0` - For response caching **Environment Variables:**
- `ELASTICSEARCH_URL` - ES cluster endpoint
- `SEARCH_CACHE_TTL=300` - Cache duration in seconds **Risks:**
- **Risk:** ES cluster downtime breaks search **Mitigation:** Add fallback to basic SQLite search, circuit breaker pattern
- **Risk:** Initial indexing takes hours **Mitigation:** Run during off-peak, show progress, make resumable **Estimated Complexity:** High
**Reasoning:** New infrastructure (ES), data migration, significant code changes
``` --- ## STEP 5: UPDATE GITHUB ISSUE **CRITICAL DISTINCTION:**
- **Issue refinement** → Update issue BODY using `gh issue edit --body`
- **Supplementary notes** → Post as comment using `gh issue comment` For issue refinement, ALWAYS update the issue body directly, NOT as a comment. 1. **Format the refinement:** - Combine WHAT, WHY, and HOW sections - Use clear markdown formatting - Include code blocks where helpful 2. **Update the issue body:** **Method 1: Using temporary file (recommended):** ```bash ISSUE_NUMBER=<issue-number> TEMP_FILE="/tmp/refined_issue_${ISSUE_NUMBER}.md" # Fetch current body to preserve original description CURRENT_BODY=$(gh issue view $ISSUE_NUMBER --json body -q '.body') # Create updated body with refinement appended cat > "$TEMP_FILE" << 'EOF'
${CURRENT_BODY} ---
## 🔍 ISSUE REFINEMENT [Insert WHAT section] [Insert WHY section] [Insert HOW section] ---
**Refinement completed.** Ready for implementation.
EOF # Update the issue body gh issue edit $ISSUE_NUMBER --body-file "$TEMP_FILE" # Verify gh issue view $ISSUE_NUMBER --json body -q '.body' | grep "ISSUE REFINEMENT" # Clean up rm "$TEMP_FILE" ``` **Method 2: Direct inline update:** ```bash CURRENT_BODY=$(gh issue view <issue-number> --json body -q '.body') UPDATED_BODY="$CURRENT_BODY --- ## 🔍 ISSUE REFINEMENT [Insert WHAT/WHY/HOW sections] --- **Refinement completed.** Ready for implementation. " gh issue edit <issue-number> --body "$UPDATED_BODY" ``` 3. **Add labels:** - `refined` - `complexity:low` OR `complexity:medium` OR `complexity:high` - Any other relevant labels (feature, bug, etc.) ```bash gh issue edit <issue-number> \ --add-label refined \ --add-label complexity:<assessed-level> ``` 4. **Refinement body format:**
```markdown
---

## 🔍 ISSUE REFINEMENT This issue has been analyzed and refined with implementation details. [Insert WHAT section] [Insert WHY section] [Insert HOW section] ---
**Refinement completed.** Ready for implementation.
``` **Remember:** Use `gh issue edit --body` for refinement, NOT `gh issue comment`. --- ## STEP 6: REQUEST USER PERMISSION **Stop here and ask:** ```markdown
✅ **Issue refinement complete!** I've updated GitHub issue #[number] with detailed WHAT, WHY, and HOW sections. **Summary:**
- **Scope:** [Brief 1-line summary]
- **Complexity:** [Low/Medium/High]
- **Estimated time:** ~[X] minutes (based on similar issues)
- **Components affected:** [Count]
- **Risks:** [None / Low / Medium / High] **Next steps - your choice:** **Option 1: Start implementation now**
I'll proceed directly to code generation and complete the full workflow
(code → RL loop → PR ready for review). Estimated completion: ~[X] minutes. **Option 2: Wait for later**
I'll stop here. You can review the refinement and tell me when to start
implementation. **Please respond:**
- Type `1` or `start` or `implement now` to begin implementation
- Type `2` or `wait` or `later` to pause here What would you like me to do?
``` **CRITICAL:** Do NOT proceed to Phase 2 without explicit user permission. --- ## QUALITY CHECKLIST Before updating the issue, verify: **WHAT Section:**
- [ ] Clear one-line summary
- [ ] Specific scope items (not vague)
- [ ] 3-6 testable acceptance criteria
- [ ] Out-of-scope items clarified
- [ ] All affected components listed **WHY Section:**
- [ ] Business value clearly stated
- [ ] Problem statement describes current state
- [ ] Expected impact is realistic and specific
- [ ] Context includes related issues/dependencies **HOW Section:**
- [ ] Technical approach explained clearly
- [ ] All files to create/modify listed with paths
- [ ] Specific function/class names provided
- [ ] Testing strategy covers unit + integration
- [ ] Dependencies and config changes specified
- [ ] Risks identified with mitigations
- [ ] Complexity accurately assessed **Overall:**
- [ ] Someone else could implement this from the HOW section alone
- [ ] Acceptance criteria map to implementation details
- [ ] No ambiguity remains
- [ ] Realistic and achievable scope --- ## COMMON MISTAKES TO AVOID ❌ **Vague acceptance criteria**
- Bad: "Make it faster"
- Good: "API response time <200ms for 95th percentile" ❌ **Missing file paths**
- Bad: "Update the config"
- Good: "Update `config/settings.yaml` line 45 to change timeout to 30s" ❌ **No test strategy**
- Bad: "Write tests"
- Good: "Test happy path, test 404 error, test rate limit exceeded" ❌ **Ignoring risks**
- Always identify at least 1-2 potential risks and mitigations ❌ **Wrong complexity**
- Be honest: Simple refactor = Low, New API integration = High ❌ **Not asking permission**
- Always stop and ask user if they want to implement now or later --- ## TIPS FOR BETTER REFINEMENT 1. **Look at merged PRs** - See how similar features were implemented
2. **Check project conventions** - Follow existing patterns strictly
3. **Be specific with names** - `UserAuthService` not "auth thing"
4. **Think through dependencies** - What packages/services are needed?
5. **Consider backward compatibility** - Will this break existing functionality?
6. **Estimate realistically** - Better to overestimate than underestimate
7. **Make it implementable** - Could another developer implement from HOW alone? --- ## RETURN TO CORE WORKFLOW After completing refinement and getting user permission:
- Return to core workflow
- Proceed to Phase 2: Code Generation
- Read `.claude/skills/phase2_codegen.md` before starting
