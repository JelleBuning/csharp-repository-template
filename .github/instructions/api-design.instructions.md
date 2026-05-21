---
applyTo: "**/*.cs"
---

# API Design

## RESTful Conventions

### Resource Naming

Use **nouns** (not verbs) for resources, actions via HTTP methods:

```csharp
// ✅ Good
POST   /api/v1/users              // Create user
GET    /api/v1/users/{id}         // Get user
PUT    /api/v1/users/{id}         // Replace user
PATCH  /api/v1/users/{id}         // Partial update
DELETE /api/v1/users/{id}         // Delete user
GET    /api/v1/users/{id}/orders  // Get orders for user

// ❌ Bad
POST   /api/v1/createUser
GET    /api/v1/getUser
POST   /api/v1/deleteUser
```

### HTTP Methods

- **GET**: Retrieve data (safe, idempotent)
- **POST**: Create new resource (not idempotent)
- **PUT**: Replace entire resource (idempotent)
- **PATCH**: Partial update (not always idempotent)
- **DELETE**: Remove resource (idempotent)
- **HEAD**: Like GET, but no body (safe, idempotent)
- **OPTIONS**: Describe communication options

### Status Codes

- **2xx Success**
  - 200 OK: Successful GET, PUT, PATCH
  - 201 Created: POST created resource
  - 204 No Content: Successful DELETE, or POST with no response body
- **3xx Redirection**
  - 301 Moved Permanently
  - 307 Temporary Redirect
- **4xx Client Error**
  - 400 Bad Request: Validation failure
  - 401 Unauthorized: Authentication required
  - 403 Forbidden: Authenticated but no permission
  - 404 Not Found: Resource doesn't exist
  - 409 Conflict: Business rule violation (duplicate, etc.)
- **5xx Server Error**
  - 500 Internal Server Error: Unhandled exception
  - 503 Service Unavailable: Maintenance, downstream dependency down