---
name: phase2-codegen
description: Generate production-quality code following acceptance criteria from refined issue, with comprehensive tests and documentation. Use after issue refinement (Phase 1) is complete.
argument-hint: "[issue-number-or-description]"
--- # PHASE 2: CODE GENERATION SKILL **Purpose:** Generate complete, production-quality implementation based on refined issue. --- ## PREREQUISITES Before starting Phase 2:
- ✅ Issue contains WHAT, WHY, and HOW sections
- ✅ User has approved implementation (if issue was just refined) --- ## AGENT DELEGATION (RECOMMENDED) For complex implementations, delegate to the appropriate developer agent based on the service being modified: ### Service Detection | File Pattern | Service | Developer Agent |
|--------------|---------|-----------------|
| `backend service/**`, `*.go`, `internal/**`, `cmd/**` | Backend API | `backend-developer` |
| `frontend/**`, `src/components/**`, `src/pages/**` | Frontend | `frontend-developer` |
| `cli/**`, `src/commands/**` | CLI | `cli-developer` |
| `website/**` | Marketing Website | `website-developer` |
| `docs-site/**`, `docs/**/*.md` | Documentation Site | `docs-site-developer` | ### Delegation Template ```
Task tool with:
- subagent_type: [detected-developer-agent]
- prompt: | Implement the following feature based on the refined GitHub issue. ## WHAT (Acceptance Criteria) {paste WHAT section} ## WHY (Business Context) {paste WHY section} ## HOW (Implementation Details) {paste HOW section} Follow all developer guidelines and return when complete.
``` ### When to Delegate - **Always delegate** for service-specific changes (Go backend, React frontend, etc.)
- **Self-implement** for simple cross-cutting changes (config updates, small docs)
- **Sequential delegation** for multi-service changes (backend first, then frontend) ### Benefits of Delegation - Developer agents follow service-specific guidelines automatically
- Better code quality through domain expertise
- Reduced context usage in main workflow
- Consistent patterns across similar tasks --- ## STEP 1: REVIEW REFINED ISSUE **Re-read completely:**
1. WHAT section - Understand ALL acceptance criteria
2. WHY section - Understand the business value (helps with decisions)
3. HOW section - This is your implementation blueprint **Extract key information:**
- List of files to create
- List of files to modify
- List of functions/classes to implement
- List of edge cases to handle
- List of tests to write **Create mental checklist:**
- Map each acceptance criterion to code you'll write
- Identify which HOW items satisfy which WHAT criteria --- ## STEP 2: IMPLEMENT CORE FUNCTIONALITY ### Follow HOW Section Exactly **For each component in HOW section:** 1. **Create or open the file**
2. **Implement exactly as specified:** - Create classes/functions with exact names from HOW - Follow the described logic - Handle edge cases mentioned - Add error handling 3. **Match project conventions:** - Follow existing code style - Use same naming patterns - Match file organization - Follow architectural patterns ### Code Quality Standards **Required in ALL code:** ✅ **Proper imports**
- Group: stdlib, third-party, local
- Alphabetize within groups
- No unused imports ✅ **Type hints** (Python) or types (TypeScript)
```python
def calculate_total(items: List[Item], tax_rate: float) -> Decimal: ...
``` ✅ **Docstrings/comments**
```python
def complex_algorithm(data: dict) -> Result: """ Process data using XYZ algorithm. Args: data: Input data containing keys 'x', 'y', 'z' Returns: Result object with processed data Raises: ValueError: If required keys missing """
``` ✅ **Error handling**
- Catch specific exceptions, not generic
- Log errors appropriately
- Provide helpful error messages
- Clean up resources (use context managers) ✅ **No magic numbers/strings**
```python
# Bad
if status == 200: # Good
HTTP_OK = 200
if status == HTTP_OK:
``` ✅ **Clear variable names**
```python
# Bad
d = get_data()
x = process(d) # Good
user_data = get_user_data()
validated_user = validate_user_data(user_data)
``` ✅ **Single responsibility**
- Functions do one thing
- Classes have one purpose
- Avoid god functions/classes ✅ **DRY (Don't Repeat Yourself)**
- Extract common logic to functions
- Use inheritance/composition appropriately
- Avoid copy-paste code ### Language-Specific Standards **Python:**
- Follow PEP 8
- Use type hints (Python 3.9+ syntax)
- Use dataclasses for data structures
- Use context managers for resources
- Prefer pathlib over os.path **Go:**
- Follow Go conventions
- Use error wrapping (`fmt.Errorf`)
- Defer cleanup
- Use interfaces for abstractions **JavaScript/TypeScript:**
- Use const/let, not var
- Prefer async/await over callbacks
- Use TypeScript types strictly
- Follow project's ESLint rules --- ## STEP 3: WRITE COMPREHENSIVE TESTS ### Test Coverage Requirements **For EVERY new function/method:** 1. **Happy path test**
```python
def test_calculate_total_with_valid_items(): items = [Item(price=10), Item(price=20)] total = calculate_total(items, tax_rate=0.1) assert total == Decimal('33.00')
``` 2. **Edge cases**
```python
def test_calculate_total_with_empty_list(): total = calculate_total([], tax_rate=0.1) assert total == Decimal('0.00') def test_calculate_total_with_zero_tax(): items = [Item(price=10)] total = calculate_total(items, tax_rate=0) assert total == Decimal('10.00')
``` 3. **Error cases**
```python
def test_calculate_total_with_negative_tax_raises_error(): items = [Item(price=10)] with pytest.raises(ValueError, match="Tax rate cannot be negative"): calculate_total(items, tax_rate=-0.1)
``` ### Test Organization **Unit Tests:**
- One test file per source file
- Name: `test_[source_file].py`
- Test individual functions in isolation
- Mock external dependencies
- Fast (milliseconds) **Integration Tests:**
- Test component interactions
- Test with real dependencies where safe (local DB, file system)
- Mock external APIs/services
- Test end-to-end workflows **Test Structure (AAA Pattern):**
```python
def test_user_registration(): # Arrange - Set up test data user_data = {"email": "test@example.com", "password": "secure123"} # Act - Execute the function result = register_user(user_data) # Assert - Verify the outcome assert result.success is True assert result.user.email == "test@example.com"
``` ### Testing Checklist For each acceptance criterion from WHAT section:
- [ ] At least one test validates this criterion
- [ ] Happy path covered
- [ ] Edge cases covered
- [ ] Error handling covered Overall:
- [ ] All new functions have tests
- [ ] All modified functions have tests updated
- [ ] Integration tests cover main workflows
- [ ] Tests are deterministic (no flaky tests)
- [ ] Tests run fast (<5 seconds for unit tests) --- ## STEP 4: UPDATE DOCUMENTATION ### Code Documentation **Every public function/method:**
```python
def search_users(query: str, limit: int = 10) -> List[User]: """ Search for users by name or email. Args: query: Search string to match against name/email limit: Maximum number of results (default: 10) Returns: List of User objects matching the query, sorted by relevance Raises: ValueError: If limit is negative or exceeds 100 DatabaseError: If database connection fails Example: >>> users = search_users("john", limit=5) >>> len(users) 3 """
``` **Complex logic:**
```python
# Calculate compound interest using formula: A = P(1 + r/n)^(nt)
# where P=principal, r=rate, n=compounds per year, t=years
final_amount = principal * (1 + rate / compounds_per_year) ** (compounds_per_year * years)
``` ### Project Documentation **README.md - Update if:**
- Public API changes
- New features added
- Installation steps change
- Configuration options added **Example README addition:**
```markdown
## New Feature: User Search Search for users by name or email: ```python
from myapp import search_users results = search_users("john")
for user in results: print(user.name)
``` **Configuration**
- `SEARCH_LIMIT_MAX`: Max results per query (default: 100)
``` **API Documentation - Update if:**
- New endpoints added
- Request/response formats change
- Authentication requirements change **Configuration Documentation:**
```markdown
## New Environment Variables - `ELASTICSEARCH_URL` (required): Elasticsearch cluster URL - Example: `http://localhost:9200` - Used for search indexing - `SEARCH_CACHE_TTL` (optional): Cache duration in seconds - Default: 300 - Range: 0-3600
``` --- ## STEP 5: UPDATE DEPENDENCIES & CONFIGURATION ### Dependencies **When adding new packages:** 1. **Add to requirements file:**
```txt
# requirements.txt
elasticsearch==8.10.0 # For search indexing
redis==5.0.0 # For response caching
``` 2. **Document why needed:**
- Add comment explaining purpose
- Pin to specific version
- Check for security vulnerabilities 3. **Update lock files:**
```bash
# Python
pip freeze > requirements.txt # Node
npm install package-name --save
``` ### Configuration **When adding new config:** 1. **Environment variables:**
```python
# config.py
ELASTICSEARCH_URL = os.getenv('ELASTICSEARCH_URL', 'http://localhost:9200')
SEARCH_CACHE_TTL = int(os.getenv('SEARCH_CACHE_TTL', '300'))
``` 2. **Config files:**
```yaml
# config/settings.yaml
search: elasticsearch_url: ${ELASTICSEARCH_URL} cache_ttl: 300 max_results: 100
``` 3. **Example/template files:**
```bash
# .env.example
ELASTICSEARCH_URL=http://localhost:9200
SEARCH_CACHE_TTL=300
``` 4. **Document defaults:**
- Provide sensible defaults
- Document valid ranges
- Explain what each setting does --- ## STEP 6: VALIDATE AGAINST ACCEPTANCE CRITERIA **Before considering Phase 2 complete:** Go through EACH acceptance criterion from WHAT section: ```markdown
Acceptance Criteria:
- [ ] API response time <200ms for 95th percentile → Verify: Performance test written and passing - [ ] Error handler catches database failures → Verify: Error handling code exists, test covers this - [ ] User sees clear error message for large uploads → Verify: Error message implemented, test validates message
``` **Self-check questions:**
- Can I demo this acceptance criterion?
- Is there a test that would fail if this criterion wasn't met?
- Does the code actually implement what the criterion describes? **If ANY criterion is not met:**
- Do NOT proceed to Phase 3
- Implement the missing functionality
- Add the missing tests --- ## QUALITY GATES (Self-Check Before Phase 3) Run these checks locally: ### 1. Code Runs Without Errors
```bash
# Python
python -m myapp # Node
npm start # Go
go run main.go
``` ### 2. Tests Pass Locally
```bash
# Python
pytest # Node
npm test # Go
go test ./...
``` ### 3. Linting Passes
```bash
# Python
ruff check .
mypy . # Node
npm run lint # Go
golangci-lint run
``` ### 4. Type Checking Passes
```bash
# Python
mypy src/ # TypeScript
tsc --noEmit
``` ### 5. No Obvious Issues
- No print statements left in code
- No commented-out code blocks
- No TODO comments (move to issues)
- No debug logging in production code --- ## COMMON MISTAKES TO AVOID ❌ **Incomplete implementation**
- Implementing 80% and hoping RL loop will catch it
- Skipping "minor" acceptance criteria ❌ **No error handling**
- Happy path only
- Not handling expected failures ❌ **Insufficient tests**
- Only testing happy path
- Not testing edge cases
- No integration tests ❌ **Ignoring existing patterns**
- Implementing differently than rest of codebase
- Not following project conventions ❌ **Over-engineering**
- Adding features not in WHAT section
- Making it "future-proof" beyond requirements ❌ **Under-engineering**
- Quick hacks instead of proper solutions
- Ignoring risks from HOW section ❌ **Poor naming**
- Vague function names like `process()`, `handle()`, `do_thing()`
- Abbreviations that aren't obvious ❌ **Missing documentation**
- No docstrings
- No README updates
- Undocumented config changes --- ## TIPS FOR BETTER CODE 1. **Read similar code first** - See how others solved similar problems
2. **Start with tests** - Write test skeleton first (TDD)
3. **Implement incrementally** - One acceptance criterion at a time
4. **Run tests frequently** - After each function/feature
5. **Commit logically** - Small, focused commits (not one giant commit)
6. **Review your own code** - Read through before moving to Phase 3
7. **Check acceptance criteria** - Validate each one is satisfied --- ## EXIT CRITERIA CHECKLIST Before proceeding to Phase 3: **Code:**
- [ ] All functions from HOW section implemented
- [ ] All classes from HOW section implemented
- [ ] All edge cases from HOW section handled
- [ ] Error handling for all failure modes
- [ ] Code follows project conventions
- [ ] No linting errors locally
- [ ] Type checking passes locally **Tests:**
- [ ] Unit tests for all new functions
- [ ] Integration tests for main workflows
- [ ] All acceptance criteria have tests
- [ ] Tests pass locally
- [ ] Code coverage maintained/improved **Documentation:**
- [ ] Docstrings on all public functions
- [ ] Comments for complex logic
- [ ] README updated (if needed)
- [ ] API docs updated (if needed)
- [ ] Config changes documented **Configuration:**
- [ ] Dependencies added to requirements file
- [ ] Environment variables documented
- [ ] Config files updated
- [ ] Example configs provided **Validation:**
- [ ] Each WHAT acceptance criterion satisfied
- [ ] Implementation matches HOW approach
- [ ] No deviations from plan (or documented why)
- [ ] Ready for automated quality checks --- ## PROCEED TO PHASE 3 Once all exit criteria are met:
- Return to core workflow
- Proceed to Phase 3: RL Loop
- Read `.claude/skills/phase3_rl_loop.md` before starting **Note:** For detailed quality gate definitions, see `.claude/skills/common/quality-gates.md`
