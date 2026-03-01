# Architecture Patterns Reference

## SOLID Principles

- **Single Responsibility:** Each module/class/function has one reason to change
- **Open/Closed:** Open for extension, closed for modification — use interfaces/abstractions
- **Liskov Substitution:** Implementations must be substitutable for their interfaces
- **Interface Segregation:** Small, focused interfaces — not kitchen-sink interfaces
- **Dependency Inversion:** Depend on abstractions (interfaces), not concrete types

## Patterns to Verify

### Dependency Injection
- Are dependencies injected via constructors or function parameters, not created internally?
- Are interfaces used at module boundaries?
- Is there a clear composition root (main/entry point)?

### Repository / Storage Pattern
- Is data access abstracted behind interfaces?
- Are storage implementations swappable?
- Is business logic free of storage-specific code?

### Service Layer
- Is business logic separated from HTTP handlers/controllers?
- Do services depend on interfaces, not concrete storage implementations?
- Are services testable without spinning up infrastructure?

### Middleware / Decorator
- Are cross-cutting concerns (logging, auth, recovery) handled via middleware?
- Is middleware composable and single-purpose?

### Factory / Builder
- Are complex objects created consistently?
- Is construction logic separated from business logic?

## Anti-Patterns to Flag

### God Object
A class/struct/package that does too much. Signs:
- >10 methods, >5 dependencies, >300 lines
- Name is vague ("Manager", "Handler", "Processor", "Utils", "Helper")

### Service Locator
Global registry that provides dependencies at runtime instead of injection. Signs:
- `GetService()`, `Resolve()`, global maps of interfaces
- Makes dependencies implicit and testing harder

### Anemic Domain Model
Data structures with no behavior — all logic lives in separate service functions.
- Not always bad, but flag if domain logic is scattered and inconsistent

### Spaghetti Architecture
No clear layering — controllers call data access directly, business logic in HTTP layer.
- Check: can you describe the dependency flow in one sentence?

### Lava Flow
Dead code, unused types, commented-out blocks left from previous iterations.
- Check: are all exported/public symbols actually used?

### Shotgun Surgery
One logical change requires touching many unrelated files.
- Check: are related concerns co-located?

### Feature Envy
A function that accesses data from another module more than its own.
- Signs: excessive cross-module imports for data access

## Language-Agnostic Patterns

### Interface Compliance
- Interfaces defined where consumed, not where implemented
- Small interfaces (1-3 methods preferred)
- Accept interfaces, return concrete types

### Error Handling
- Errors wrapped with context at each layer
- Typed/sentinel errors for expected failure cases
- No swallowed errors (empty catch/if-err blocks)

### Concurrency Safety
- Shared mutable state protected by locks or message passing
- Background workers have clear lifecycle (start, stop, cleanup)
- Cancellation/context propagation for long-running operations

### Package / Module Design
- No circular imports
- Internal packages/modules for implementation details
- Clear public API surface per module
