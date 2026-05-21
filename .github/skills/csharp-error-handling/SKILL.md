---
name: csharp-error-handling
description: 'Get best practices for error handling and result patterns in C#'
---

# C# Error Handling & Result Patterns

Your goal is to help me design robust error handling using the ErrorOr pattern, distinguishing between expected failures (modeled in the type system) and truly exceptional conditions (thrown as exceptions).

## Core Philosophy

- **Expected failures** (validation, not found, conflict) → model as `ErrorOr<T>`, never throw
- **Exceptions** → only for truly exceptional, unrecoverable conditions (null reference, system crash)
- **Explicit error handling** → type system makes errors visible at compile time, not runtime surprises

## ErrorOr<T> vs Exceptions

### When to Use ErrorOr<T>

Use `ErrorOr<T>` for **expected failures** in domain and application logic:

- ✅ Validation failures (invalid email, password too weak)
- ✅ Not found (user doesn't exist, document not found)
- ✅ Business rule violations (conflict, duplicate, insufficient balance)
- ✅ Authentication/authorization failures (unauthorized, forbidden)

### When to Throw Exceptions

Throw exceptions **only** for unrecoverable, truly exceptional conditions:

- ✅ Null reference (null argument passed, unexpected null from database)
- ✅ System resource exhaustion (out of memory, disk full)
- ✅ Unexpected external service failure (assuming it's temporary)
- ✅ Configuration errors (missing required setting at startup)

**Example**: Document not found → `ErrorOr<Document>` (expected). Database connection dropped → exception (unrecoverable).

## Error Types

Use semantic error types for clarity:

```csharp
// Validation failure
return Error.Validation("Email.Invalid", "Email must contain '@' character");

// Resource not found
return Error.NotFound("User.NotFound", $"User {userId} not found");

// Business rule violation
return Error.Conflict("Order.CannotCancel", "Cannot cancel a completed order");

// Permission denied
return Error.Forbidden("Document.NoAccess", "You don't have permission to view this document");

// Authentication required
return Error.Unauthorized("User.NotAuthenticated", "You must log in first");

// Generic error
return Error.Failure("Processing.Failed", "Document processing failed due to corrupted file");
```