# Code Smells Checklist

## Function-Level Smells

- [ ] Long functions (>50 lines — investigate, >100 lines — always flag)
- [ ] Multiple responsibilities in one function
- [ ] Too many parameters (>4)
- [ ] Boolean flag parameters (split into two functions instead)
- [ ] Deep nesting (>3 levels of indentation)
- [ ] Output parameters (returning via pointer args instead of return values)
- [ ] Functions that mix abstraction levels (high-level orchestration + low-level details)
- [ ] Inconsistent return patterns (sometimes error, sometimes panic, sometimes log-and-continue)

## Class / Type-Level Smells

- [ ] Large classes/structs (>10 fields or >300 lines — may need decomposition)
- [ ] Too many methods on one type
- [ ] Class/struct used as a grab-bag (unrelated fields grouped together)
- [ ] Exported/public fields that should be private
- [ ] Missing zero-value / default safety (usable without constructor?)
- [ ] Feature envy (methods that mostly use another type's data)

## Package / Module-Level Smells

- [ ] Circular dependencies between packages/modules
- [ ] Too many files in one module (>15 — consider splitting)
- [ ] Package/module name doesn't reflect contents
- [ ] `utils`, `helpers`, `common` packages (usually a sign of poor organization)
- [ ] Mixed abstraction levels in one package/module
- [ ] Package/module exports more than it should

## Naming Smells

- [ ] Inconsistent naming conventions across the codebase
- [ ] Abbreviations without context used inconsistently
- [ ] Names that lie (function name doesn't match what it does)
- [ ] Generic names (`data`, `result`, `item`, `temp`) in non-trivial scope
- [ ] Stuttering (`user.UserName` instead of `user.Name`)

## Duplication Smells

- [ ] Copy-pasted code blocks (>5 lines identical)
- [ ] Similar functions that differ in one parameter (should be parameterized)
- [ ] Repeated error handling patterns that should be middleware
- [ ] Duplicated validation logic across handlers
- [ ] Same type/struct defined in multiple places

## Error Handling Smells

- [ ] Swallowed errors (empty catch blocks or ignored error returns)
- [ ] Errors without context (re-thrown without wrapping/message)
- [ ] Panic/exception used for recoverable errors
- [ ] Inconsistent error types (string errors vs typed errors vs sentinel values)
- [ ] Error messages that don't help debugging (no context about what failed)
- [ ] Logging an error AND re-throwing it (double-reporting)

## Test Smells

- [ ] Tests coupled to implementation details (break on refactor)
- [ ] Missing edge case coverage
- [ ] Test names don't describe the scenario
- [ ] Test setup is >50% of the test function
- [ ] Shared mutable test state between test cases
- [ ] No parameterized/table-driven tests where patterns repeat
- [ ] Tests that pass but don't actually assert anything meaningful

## Concurrency Smells

- [ ] Background goroutines/threads without lifecycle management (fire-and-forget)
- [ ] Shared state without synchronization
- [ ] Channel/async operations without timeout or cancellation
- [ ] Lock/mutex held across I/O operations
- [ ] Potential race conditions on shared state

## Dependency Smells

- [ ] Concrete types where interfaces would allow testing
- [ ] Dependencies created inside functions instead of injected
- [ ] Too many dependencies in one constructor (>5 — component does too much)
- [ ] Unused dependencies (imported but not meaningfully used)
- [ ] Pinned to old versions without reason
