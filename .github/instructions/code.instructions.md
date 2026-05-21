---
applyTo: "**/*.cs"
---

# Coding Standards

- **Modern C#**: Use records, pattern matching, and init-only properties where appropriate.
- **Immutability**: Use `record`/`record struct` for DTOs and Value Objects. Use `init`-only setters on entities. Never expose public setters on domain objects.
- **Collections**: Use `IReadOnlyList<T>` or `IReadOnlyCollection<T>` for public API return types.
- **Nullable**: Enable nullable reference types. Never return `null` — use `Result<T>`, `Option<T>`, or empty collections instead.
- **Guard Clauses**: Validate all arguments using `ArgumentNullException.ThrowIfNull()` or `ArgumentException`. Never silently ignore invalid input.

# Patterns & Error Handling

## Design Patterns

Apply patterns intentionally with a clear reason:
- Behavior varies by type/condition → **Strategy**
- Object creation is complex or conditional → **Factory**
- Adding behavior without modifying a class → **Decorator**

## Result<T> and ErrorOr<T>

Expected failures (validation, business rule violations, not found) should be modeled in the type system using `ErrorOr<T>`, not thrown as exceptions.

**Why ErrorOr?**
- Exceptions are for truly exceptional, unrecoverable conditions (null reference, system crash).
- Using exceptions for control flow is slow and makes error handling implicit.
- `ErrorOr<T>` makes error handling explicit and composable.

### Handler Pattern

Implement handlers that return `ErrorOr<T>` from domain/application operations.

### Error Types

- `Error.Validation(code, description)` — Input validation failure
- `Error.NotFound(code, description)` — Resource not found
- `Error.Conflict(code, description)` — Business rule violation
- `Error.Unauthorized(code, description)` — Authentication required
- `Error.Forbidden(code, description)` — Authenticated but no permission
- `Error.Failure(code, description)` — Generic error

### Pattern Matching

Handle both success and failure cases using pattern matching (.Match() or MatchAsync() methods).

### Exception Handling

- Throw exceptions only for truly exceptional, unrecoverable conditions.
- Never use exceptions for control flow (validation, not found, conflicts).

## General Guidelines

- Do not introduce new NuGet packages without explicit approval.
- Do not use `static` classes or methods unless absolutely necessary.

# Performance

- No micro-optimization unless profiling has identified a bottleneck.
- Use `Span<T>` only when justified by measurable performance needs and it remains readable.
- For async streams, see `.github/skills/csharp-async/SKILL.md`.