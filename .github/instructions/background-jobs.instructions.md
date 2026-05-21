---
applyTo: "**/*.cs"
---

# Background Job Processing

Long-running operations (document extraction, batch processing, data sync) must run outside HTTP request cycles. Use `IHostedService` (built-in .NET) to host background workers that consume queue messages and process jobs with resilience, observability, and graceful shutdown.

## Core Principles

- **Hosted Service Pattern**: Implement `IHostedService` or inherit `BackgroundService` for worker lifecycle management.
- **Job State Machine**: Track job progress through discrete states (Pending → Processing → Completed/Failed).
- **Graceful Shutdown**: Honor cancellation tokens, complete in-flight jobs, and drain queues on shutdown.
- **Observability**: Log state transitions, durations, errors with correlation IDs for tracing.
- **Idempotency**: Tolerate duplicate message delivery; process should succeed even if partially complete.

## Key Patterns Summary

| Pattern | Purpose | Example |
|---------|---------|---------|
| **BackgroundService** | Host long-running workers | DocumentProcessingWorker inherits BackgroundService |
| **State Machine** | Track job lifecycle | Pending → Processing → Completed/Failed |
| **Correlation ID** | End-to-end tracing | Same ID in logs, messages, database records |
| **Exponential Backoff** | Resilient retry | 1s, 2s, 4s, 8s, then DLQ |
| **Graceful Shutdown** | Clean termination | Honor CancellationToken, drain queue on stop |
| **Idempotency** | Tolerate duplicates | Reprocess safely without side effects |
