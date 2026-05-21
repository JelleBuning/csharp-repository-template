---
applyTo: "**/*.cs"
---

# Architecture: Modular Monolith
A single deployable unit organized into vertical slices (modules) with clear boundaries, independent of each other but sharing infrastructure.

## Core Principles
- **Vertical Slices**: Each module owns a complete feature (API, domain logic, persistence, tests).
- **Module Independence**: Minimal coupling between modules; communicate via events or through application services.
- **Shared Kernel**: Common utilities, value objects, and infrastructure in a `Shared` or `Common` module.
- **Single Deployment**: Ship as one artifact, but structured for future extraction if needed.

## Module Structure (example: `Users` module)
```
Users/
  ├── Application/           (Commands, queries, handlers, orchestration)
  ├── Domain/                (Entities, value objects, domain services, events)
  ├── Infrastructure/        (EF Core DbContext, repositories, external APIs)
  ├── Presentation/          (Controllers, DTOs, request/response models)
  └── Tests/
```

## Domain-Driven Design (DDD)

### Ubiquitous Language
- Use domain terms consistently across code, discussions, and documentation.
- Avoid generic names: use `Order`, `Invoice`, `Shipment` instead of `Entity`, `Item`.
- Domain experts and developers should speak the same language.

### Bounded Contexts
Each module is a bounded context with clear boundaries:
- **Anti-Corruption Layer (ACL)**: Translate external models at boundaries.
- **Shared Kernel**: Minimal shared types (rarely needed).

## Dependency Injection

### Container Choice
Use **Microsoft.Extensions.DependencyInjection** (built into .NET):
- Built-in, no additional NuGet dependency for basic scenarios.
- Sufficient for modular monolith with vertical slices.
- Use **Scrutor** for assembly scanning if needed.

### Module Registration Pattern
Create an extension method per module:

```csharp
public static class DependencyInjection
{
    public static IServiceCollection AddUsersModule(this IServiceCollection services)
    {
        services.AddScoped<IUserRepository, UserRepository>();
        services.AddScoped<CreateUserHandler>();
        return services;
    }
}
```

In `Program.cs`:
```csharp
builder.Services
    .AddUsersModule()
    .AddOrdersModule()
    .AddSharedModule();
```

### Lifetime Guidelines
- **Transient**: Stateless utilities, factories.
- **Scoped**: Application handlers, repositories, DbContext (per HTTP request).
- **Singleton**: Configuration, caches, logging factories, event publishers.

### Anti-Patterns
- Never inject `IServiceProvider` to resolve dependencies on demand (Service Locator).
- Never use static factories like `SomeFactory.Create()` in production code.
- If modules need bidirectional communication, use **Domain Events** or a mediator.

### Migrations
- Generate: `dotnet ef migrations add MigrationName --context UsersDbContext`
- Never edit migrations manually (except fixing typos).
- Always commit migrations to source control.

### Query Performance
**N+1 Prevention**: Always `.Include()` related data in a single query:
```csharp
var users = await _context.Users
    .Include(u => u.Roles)
    .ToListAsync();
```

**Lazy Loading**: Disable by default. Use explicit eager loading with `.Include()`.

**Projections**: Use `.Select()` to return only needed columns.

## Logging & Observability

### Structured Logging
Use **Serilog** with structured properties for queryable logs:
```csharp
_logger.LogInformation("User {UserId} created successfully", user.Id);
```

### Key Decision Points
Log at important domain/application events without cluttering code:
- User registration, login, permission changes.
- Domain invariant violations (conflicts, validation failures).
- External service calls (API calls, database operations).
- Error conditions with context.

### No Secrets in Logs
Never log passwords, tokens, API keys, or sensitive data.
