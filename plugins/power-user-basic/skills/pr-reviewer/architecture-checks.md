# PR Architecture Checks

Apply these checks when the PR touches multiple modules, introduces new patterns, or changes data flow.

## Layer Violations
- [ ] HTTP handlers/controllers don't contain business logic (delegated to services)
- [ ] Services don't import HTTP-specific types (request/response objects)
- [ ] Data access layer doesn't contain business logic
- [ ] Configuration layer doesn't import service or API packages
- [ ] Dependency flow follows the project's established direction (never reverse it)

## Module Boundaries
- [ ] Changes respect existing package/module boundaries
- [ ] No new circular dependencies introduced
- [ ] New packages/modules have clear, single responsibility
- [ ] Interfaces used at module boundaries (not concrete types)
- [ ] Internal implementation details not leaked through exports

## Pattern Consistency
- [ ] New code follows existing patterns for similar operations
- [ ] Error handling consistent with rest of codebase
- [ ] Naming conventions match existing code
- [ ] New endpoints/routes follow existing patterns
- [ ] New data access operations follow existing patterns

## Abstraction
- [ ] New abstractions are justified (used in 2+ places, or clear future need)
- [ ] No premature abstraction for single-use code
- [ ] No under-abstraction (copy-pasted logic that should be shared)
- [ ] Abstraction level consistent within a function/module

## Testability
- [ ] New code is testable (dependencies injectable)
- [ ] No global state that makes testing hard
- [ ] Side effects isolated and mockable
- [ ] Complex logic separated from I/O for unit testing
