---
name: csharp-async
description: 'Get best practices for C# async programming'
---

# C# Async Programming Best Practices

Your goal is to help me follow best practices for asynchronous programming in C#.

## Naming Conventions

- Use the `Async` suffix for all async methods.
- Match method names with their synchronous counterparts when applicable (e.g., `GetDataAsync()` for `GetData()`).

## Return Types

- Return `Task<T>` when the method returns a value.
- Return `Task` when the method doesn't return a value.
- Use `ValueTask<T>` only in hot-path, high-performance scenarios to reduce allocations — not by default.
- Never return `void` from async methods except for event handlers.

## Exception Handling

- Throw exceptions normally inside `async` methods — they are captured and propagated when the `Task` is awaited.
- Use `Task.FromException()` only when implementing a `Task`-returning method **without** `async/await` (e.g., interface stub implementations).
- Use try/catch around `await` expressions where recovery or logging is needed.
- Never swallow exceptions silently in async methods.

## ConfigureAwait

- **Do not use `ConfigureAwait(false)` in application code** (ASP.NET Core has no `SynchronizationContext`, making it redundant).
- Only use `ConfigureAwait(false)` in **library code** that targets multiple frameworks and must avoid capturing a synchronization context.

## Performance

- Use `Task.WhenAll()` to execute multiple independent tasks in parallel.
- Use `Task.WhenAny()` for timeouts or taking the first completed result.
- Always pass and forward `CancellationToken` in long-running or I/O-bound operations.
- You may elide `async/await` on simple pass-through methods **only when** there is no try/catch and no `using` block — removing it changes exception propagation behavior.
- Use `IAsyncEnumerable<T>` with `await foreach` for processing async streams; avoid buffering entire sequences into memory.

## Common Pitfalls
f
- Never use `.Wait()`, `.Result`, or `.GetAwaiter().GetResult()` — these block the thread and can cause deadlocks.
- Never mix blocking and async code.
- Never create `async void` methods except for event handlers.
- Always `await` Task-returning methods; never fire-and-forget unless explicitly intentional and logged.

## Patterns

- Use the Task-based Asynchronous Pattern (TAP) for all public async APIs.
- For long-running operations, implement a cancellable async command pattern.

---

When reviewing C# code, identify violations of the above and suggest corrections with a brief explanation of why the change matters.