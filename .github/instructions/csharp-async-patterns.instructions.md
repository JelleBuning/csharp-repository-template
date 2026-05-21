---
applyTo: "**/*.cs"
---

# Async/Await Patterns for Background Jobs

Modern C# async/await enables scalable, non-blocking operations. In background job systems (queuing workers, batch processing), correct async usage is critical for resource efficiency and correctness.

## Core Principles

- **Async All the Way**: Don't block on async operations (`Wait()`, `Result`, `.GetAwaiter().GetResult()`). It causes deadlocks and thread pool starvation.
- **ConfigureAwait**: Use `ConfigureAwait(false)` in libraries and background jobs to avoid unnecessary UI context capturing.
- **Cancellation Tokens**: Accept `CancellationToken` everywhere async, allowing graceful shutdown and timeout enforcement.
- **Explicit Composition**: Use `Task.WhenAll()` / `Task.WhenAny()` instead of waiting sequentially.
- **No Async Void (except events)**: Async void methods can't be awaited, making error handling impossible.

## Key Patterns Summary

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Sequential Await** | One operation depends on previous | Extract text, then index it |
| **Task.WhenAll** | Multiple independent operations | Fetch text, metadata, preview in parallel |
| **Task.WhenAny** | Wait for first completion | Poll multiple endpoints, use first response |
| **ConfigureAwait(false)** | Library/background code | Skip context capture for performance |
| **CancellationToken** | Enable graceful shutdown | Pass token through the entire call stack |
| **Factory Method** | Async initialization | `CreateAsync()` returns initialized instance |
| **IAsyncDisposable** | Cleanup async resources | `await using` for proper cleanup |
