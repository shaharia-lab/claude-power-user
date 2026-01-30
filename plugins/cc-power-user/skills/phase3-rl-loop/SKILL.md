---
name: phase3-rl-loop
description: Iteratively improve code through CI/CD feedback until all quality gates pass. Use after code generation (Phase 2) to perfect implementation through automated testing and review.
argument-hint: "[branch-name]"
--- # PHASE 3: REINFORCEMENT LEARNING LOOP SKILL **Purpose:** Iteratively improve code until ALL quality gates pass through automated feedback. --- ## LOOP OVERVIEW This is where good code becomes great code. You'll iterate through:
1. Pre-commit validation
2. Commit and push
3. CI/CD monitoring
4. PR review analysis
5. Fix generation **Repeat until perfect** (typically 2-4 iterations, max 10). --- ## SPECIALIZED AGENT DELEGATION During the RL loop, delegate to specialized agents for complex fixes: ### Go Linting Issues When golangci-lint fails with multiple issues: ```
Task tool with:
- subagent_type: golang-lint-fixer
- prompt: | Fix all golangci-lint issues in the backend service. Current errors: {paste linting output} Run `make lint` and fix all issues systematically.
``` **When to delegate:**
- More than 5 linting issues
- Complex issues (ineffassign, errcheck, govet)
- Issues spanning multiple files ### OpenAPI Schema Updates When API endpoints are added or modified: ```
Task tool with:
- subagent_type: openapi-schema-manager
- prompt: | Update OpenAPI schema for the following endpoint changes: - Added: POST /api/v1/users - Modified: GET /api/v1/prompts (added filter param) Analyze handlers and update schema accordingly.
``` **When to delegate:**
- New API endpoints added
- Request/response schemas changed
- After CI fails on OpenAPI validation ### Security Issues When security scans find vulnerabilities: ```
Task tool with:
- subagent_type: security-guardian
- prompt: | Review and fix security vulnerabilities found in CI: {paste security scan output}
``` --- ## LOOP INITIALIZATION ```python
iteration_count = 0
max_iterations = 10
all_checks_passed = False # Tracking
failed_checks = []
review_comments = []
``` --- ## ITERATION STRUCTURE ### Step 1: Pre-commit Hooks **Run ALL pre-commit hooks:**
```bash
pre-commit run --all-files
``` **What pre-commit checks:**
- Code formatting (black, prettier, gofmt)
- Linting (ruff, eslint, golangci-lint)
- Type checking (mypy, tsc)
- Unit tests
- Import sorting
- Trailing whitespace
- Large file detection **If ANY hook fails:** 1. **Read error output carefully**
```
Example error:
- hook id: ruff
- exit code: 1
main.py:45:80: E501 Line too long (95 > 88 characters)
main.py:67:1: F401 'os' imported but unused
``` 2. **Fix ALL issues** (not just the first one)
```python
# Fix line length
result = some_function( arg1, arg2, arg3
) # Split across lines # Remove unused import
# import os ← Remove this
``` 3. **Re-run hooks to verify**
```bash
pre-commit run --all-files
``` 4. **CONTINUE to next iteration** (do NOT commit yet) **If all hooks pass:**
- Log: `[RL-LOOP-{N}] [RESULT] Pre-commit hooks passed ✅`
- Proceed to Step 2 --- ### Step 2: Commit & Push **Create descriptive commit message:**
```bash
# Format: <type>: <description>
#
# Types: feat, fix, refactor, test, docs, chore git commit -m "feat: implement elasticsearch search with caching - Add ElasticsearchClient with connection pooling
- Implement search endpoint with query parsing
- Add Redis caching layer with 5min TTL
- Include unit and integration tests Closes #123"
``` **Push to feature branch:**
```bash
git push origin feature/issue-123-search-implementation
``` **Log:**
```
[RL-LOOP-{N}] [ACTION] Committed and pushed to feature/issue-123
[RL-LOOP-{N}] [RESULT] Commit SHA: abc123def
``` --- ### Step 3: Monitor CI/CD Pipeline **Wait for GitHub Actions to trigger:**
- Don't immediately move on
- Watch for pipeline to start
- Monitor all jobs **Collect outputs from ALL jobs:** 1. **Integration Tests**
```
✅ test_user_search_returns_results - PASSED
✅ test_search_with_filters - PASSED
❌ test_search_pagination - FAILED AssertionError: Expected 10 results, got 100
``` 2. **Security Scans**
```
✅ No vulnerabilities found in dependencies
⚠️ Warning: Unused dependency 'requests' in requirements.txt
``` 3. **Performance Benchmarks**
```
✅ API latency: 145ms (target: <200ms)
✅ Memory usage: 250MB (target: <500MB)
``` 4. **Code Coverage**
```
✅ Coverage: 87% (target: 85%) Missing coverage in src/utils.py lines 45-67
``` **Parse results carefully:**
- Which jobs passed?
- Which jobs failed?
- What are the error messages?
- Are there warnings to address? --- ### Step 4: Automated PR Review **Wait for Claude Code review action:**
- This runs after CI completes
- Analyzes code quality, logic, best practices **Parse review comments:** Example comments:
```
1. [BLOCKER] src/search.py:45 "Missing error handling for Elasticsearch connection failure" 2. [SUGGESTION] src/api.py:120 "Consider using dependency injection for SearchClient" 3. [NITPICK] tests/test_search.py:30 "Test name should be more descriptive"
``` **Categorize comments:**
- **BLOCKER**: Must fix before merge
- **SUGGESTION**: Should fix if reasonable
- **NITPICK**: Nice to have, low priority --- ### Step 5: Evaluation **Check if all gates passed:** ```python
all_gates_passed = ( precommit_passed and all_ci_jobs_passed and no_blocker_comments and coverage_target_met and no_security_issues and performance_benchmarks_ok
) if all_gates_passed: log("[RL-LOOP-{N}] All quality gates passed! ✅") break # Exit loop
``` **If ANY gate failed:**
- Proceed to Step 6 (Fix Issues) --- ### Step 6: Fix Issues **Prioritize fixes:** 1. **Critical** (must fix): - Failing tests - Security vulnerabilities - Blocker review comments - Breaking changes 2. **Important** (should fix): - Performance issues - Code quality suggestions - Coverage gaps - Warnings 3. **Nice to have**: - Style nitpicks - Minor refactoring suggestions **Generate comprehensive fixes:** Example process for failing test:
```
Failed test: test_search_pagination
Error: Expected 10 results, got 100 Analysis:
- Function not respecting limit parameter
- Default limit not applied Fix:
def search(query: str, limit: int = 10) -> List[Result]: results = elasticsearch.search(query) return results[:limit] # Apply limit Updated test to verify:
def test_search_pagination(): results = search("test", limit=10) assert len(results) == 10
``` **Validate fixes locally:**
```bash
# Run specific failing test
pytest tests/test_search.py::test_search_pagination # Run all tests
pytest # Run pre-commit
pre-commit run --all-files
``` **Log fixes:**
```
[RL-LOOP-{N}] [DECISION] Fixing 3 issues: 1. Add error handling for ES connection 2. Fix pagination limit bug 3. Improve test_search_pagination
``` --- ### Step 7: Next Iteration **Prepare for next loop:**
- All fixes applied
- Code ready for commit
- Return to Step 1 (Pre-commit Hooks) **Iteration complete log:**
```
[RL-LOOP-{N}] [SUMMARY] - Pre-commit: PASSED - CI/CD: 2 failures - Review: 1 blocker - Fixes applied: 3 - Ready for iteration {N+1}
``` --- ## ERROR HANDLING STRATEGIES ### When Pre-commit Fails Repeatedly **Symptom:** Same linting error after 2+ attempts **Root causes:**
- Not understanding the error
- Fixing symptom, not cause
- Config mismatch with CI **Solutions:** 1. **Read the error completely**
```
Don't just see "E501 Line too long"
Read the FULL output: File: src/main.py Line: 45 Column: 88 Actual: 95 characters Expected: Max 88 Code: result = function(arg1, arg2, arg3, arg4, arg5)
``` 2. **Understand the rule**
```bash
# Why does this rule exist?
# How do others handle it? # Look at similar code:
grep -r "similar_function" src/
``` 3. **Fix properly**
```python
# Bad fix: Add noqa comment to ignore
result = function(arg1, arg2, arg3, arg4) # noqa: E501 # Good fix: Actually make it shorter
result = function( arg1, arg2, arg3, arg4
)
``` 4. **Verify fix**
```bash
# Run just that check
pre-commit run ruff --files src/main.py
``` --- ### When Tests Fail in CI But Pass Locally **Common causes:** 1. **Environment differences**
```python
# Local: Python 3.11
# CI: Python 3.9 # Fix: Use compatible syntax
data = dict | None # Python 3.10+
data = Union[dict, None] # Python 3.9+
``` 2. **Timezone issues**
```python
# Bad: Assumes local timezone
now = datetime.now() # Good: Explicit UTC
now = datetime.now(timezone.utc)
``` 3. **Race conditions**
```python
# Bad: Non-deterministic
results = fetch_async() # Order not guaranteed # Good: Sorted for consistency
results = sorted(fetch_async(), key=lambda x: x.id)
``` 4. **File path assumptions**
```python
# Bad: Assumes Unix paths
path = "data/file.txt" # Good: Cross-platform
path = Path("data") / "file.txt"
``` **Debug strategy:**
```bash
# Check CI logs for exact error
# Look for environment info
# Reproduce CI environment locally using Docker
docker run -it python:3.9 bash
``` --- ### When Review Comments Are Unclear **Example unclear comment:**
```
"This could be better"
``` **How to interpret:** 1. **Look at context** - What line is commented on? - What pattern is used elsewhere? 2. **Check similar code**
```bash
# Find similar patterns in codebase
grep -r "similar_pattern" src/
``` 3. **Make best judgment** - Choose safest, most maintainable option - Follow project conventions - Prefer explicit over clever 4. **Document decision**
```python
# Using explicit loop instead of list comprehension
# for better readability and debugging
results = []
for item in items: if item.valid: results.append(process(item))
``` --- ### When Stuck After 5+ Iterations **Red flag:** Same error loop or regression **Debug checklist:** 1. **Review ALL previous iteration logs**
```
What changed between iteration 2 and 3?
Did we introduce a new error fixing an old one?
``` 2. **Re-read the refined issue**
```
Am I solving the right problem?
Did I misunderstand a requirement?
``` 3. **Check assumptions**
```python
# Am I assuming X is always true?
# What if X is None, empty, or unexpected?
``` 4. **Simplify approach**
```
Can I break this into smaller pieces?
Can I implement a simpler version first?
``` 5. **Look for systemic issues**
```
Is my test environment different from CI?
Is there a config issue?
Did I miss a dependency?
``` 6. **Compare with working code**
```bash
# Find similar feature that works
# See how they handled similar challenges
git log --all --grep="search implementation"
``` --- ## SPECIFIC ERROR PATTERNS ### Pattern: Import Errors **Error:**
```
ModuleNotFoundError: No module named 'elasticsearch'
``` **Fix:**
```bash
# Add to requirements.txt
echo "elasticsearch==8.10.0" >> requirements.txt # Update in CI (often needs cache clear)
# Check GitHub Actions uses correct requirements file
``` --- ### Pattern: Type Errors **Error:**
```
Argument 1 has incompatible type "str | None"; expected "str"
``` **Fix:**
```python
# Bad: Passing potentially None value
result = process(value) # Good: Handle None case
if value is not None: result = process(value)
else: result = default_value
``` --- ### Pattern: Test Flakiness **Error:**
```
Sometimes passes, sometimes fails
``` **Causes:**
- Timing dependencies
- Random data
- External service calls
- Shared state between tests **Fix:**
```python
# Bad: Timing dependent
time.sleep(1) # Hope it's done
assert result is ready # Good: Explicit wait
wait_for(lambda: result.is_ready, timeout=5) # Bad: Random data
value = random.randint(1, 100) # Good: Fixed seed or deterministic
random.seed(42)
value = random.randint(1, 100)
``` --- ### Pattern: Performance Regressions **Error:**
```
Benchmark failed: 450ms (target: <200ms)
``` **Debug:**
```python
# Add timing
import time
start = time.time()
result = expensive_operation()
print(f"Took: {time.time() - start}s") # Profile
import cProfile
cProfile.run('expensive_operation()')
``` **Common fixes:**
- Add caching
- Optimize queries (N+1 problem)
- Use batch operations
- Add indexes --- ## ITERATION EFFICIENCY TIPS **Iteration 1:**
- Focus on basic checks passing
- Get linting and formatting right
- Ensure tests run (even if some fail) **Iteration 2-3:**
- Fix failing tests thoroughly
- Address review feedback
- Improve coverage **Iteration 4+:**
- Fine-tune edge cases
- Polish error messages
- Optimize performance **If you're at iteration 7+:**
- STOP and reassess
- Review your approach
- Consider asking for clarification
- Check if you misunderstood requirements --- ## SUCCESS EXIT CRITERIA **All must be TRUE to exit loop:** ✅ **Pre-commit hooks:** All passing
✅ **Unit tests:** 100% passing
✅ **Integration tests:** 100% passing
✅ **Linting:** No errors, no warnings
✅ **Type checking:** No errors
✅ **Code coverage:** Meets or exceeds target (usually 85%+)
✅ **Security scans:** No vulnerabilities
✅ **Performance benchmarks:** All within targets
✅ **Review comments:** Zero blockers, addressed all reasonable suggestions
✅ **Acceptance criteria:** All from WHAT section demonstrably met **Once ALL are true:**
```
[RL-LOOP-{N}] [SUCCESS] All quality gates passed!
[RL-LOOP-{N}] [COMPLETE] Exiting RL loop after {N} iterations
``` **Proceed to Phase 4** --- ## FAILURE EXIT **Only if:** iteration_count >= max_iterations (10) **Actions:**
1. **Document remaining issues:**
```
After 10 iterations, unable to resolve:
1. Integration test X fails with error Y
2. Performance benchmark Z exceeds target by 50ms Potential causes:
- May need architectural change
- May require external dependency update
- May need requirements clarification
``` 2. **Request human help:**
```markdown
⚠️ **Attention Required** After 10 RL loop iterations, I was unable to resolve all quality gates. **Remaining issues:**
1. [Issue description]
2. [Issue description] **What I tried:**
- [Attempt 1]
- [Attempt 2] **Current state:**
- Code commits: [link to branch]
- Test results: [summary]
- Review feedback: [unresolved items] **Recommendation:**
[Your analysis of what might be blocking progress] I need human intervention to proceed.
``` 3. **STOP - Do not continue** --- ## COMMON MISTAKES TO AVOID ❌ **Exiting early** - "95% passing is good enough"
❌ **Ignoring warnings** - "It's just a warning, not an error"
❌ **Superficial fixes** - Silencing errors instead of fixing
❌ **Not reading full error** - Only looking at first line
❌ **Repeating same fix** - Doing same thing expecting different result
❌ **Disabling checks** - Removing pre-commit hook that fails
❌ **Scope creep** - Adding features not in requirements
❌ **Giving up** - Requesting help before trying thoroughly --- ## BEST PRACTICES ✅ **Read errors completely** - Don't skim
✅ **Fix all instances** - Not just the first occurrence
✅ **Test fixes locally** - Before committing
✅ **Commit logically** - Group related fixes
✅ **Log thoroughly** - Help future debugging
✅ **Learn from patterns** - Similar errors likely have similar fixes
✅ **Stay focused** - One issue at a time
✅ **Be systematic** - Work through errors in order --- ## PROCEED TO PHASE 4 Once all success exit criteria are met:
- Return to core workflow
- Proceed to Phase 4: Human Review Handoff
- Read `.claude/skills/phase4_pr.md` before starting **Note:** For detailed quality gate definitions, see `.claude/skills/common/quality-gates.md`
