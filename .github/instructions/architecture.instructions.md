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

### Entities vs. Value Objects

**Entities**
- Have a unique identity (Id).
- Mutable (but only via domain methods, never public setters).
- Example: `User`, `Order`, `Product`.

```csharp
public class User
{
    public UserId Id { get; }
    public Email Email { get; private set; }
    
    public void UpdateEmail(Email newEmail) => Email = newEmail;
}
```

**Value Objects**
- No identity; equality is based on value.
- Immutable (`record` or `init`-only properties).
- Example: `Money`, `Address`, `Email`.

```csharp
public record Email(string Value)
{
    public static ErrorOr<Email> Create(string value)
    {
        if (string.IsNullOrWhiteSpace(value) || !value.Contains("@"))
            return Error.Validation("Email.Invalid", "Invalid format");
        return new Email(value);
    }
}
```

### Aggregates
- **Aggregate Root**: Single entity that is the entry point for the aggregate.
- Protect invariants (business rules) within the aggregate.
- Aggregates communicate **only** through their root.

```csharp
public class Order
{
    private readonly List<OrderLine> _lines = [];
    public IReadOnlyList<OrderLine> Lines => _lines.AsReadOnly();
    public OrderStatus Status { get; private set; }
    
    public void AddLine(ProductId productId, Quantity quantity)
    {
        if (quantity.Value <= 0)
            throw new InvalidOperationException("Quantity must be > 0");
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Cannot add lines to confirmed orders");
        
        _lines.Add(new OrderLine(productId, quantity));
    }
}
```

### Domain Services
Use when logic doesn't fit naturally in an entity or value object:

```csharp
public class UserRegistrationService
{
    private readonly IUserRepository _users;
    private readonly IEmailNotifier _notifier;
    
    public async Task RegisterAsync(Email email, string password)
    {
        if (await _users.ExistsByEmailAsync(email))
            return Error.Conflict("User.EmailTaken", "Email already registered");
        
        var user = User.Register(email, password);
        await _users.AddAsync(user);
        await _notifier.SendWelcomeAsync(user.Email);
    }
}
```

### Domain Events
Publish events when domain state changes; other modules react asynchronously.

```csharp
public class User
{
    private readonly List<DomainEvent> _events = [];
    public IReadOnlyList<DomainEvent> DomainEvents => _events.AsReadOnly();
    
    public static User Register(Email email, string password)
    {
        var user = new User { Email = email };
        user.AddEvent(new UserRegistered(user.Id, email));
        return user;
    }
    
    private void AddEvent(DomainEvent @event) => _events.Add(@event);
}
```

Dispatch events in Application layer after persisting:
```csharp
var user = User.Register(email, password);
await _repo.AddAsync(user);
await _eventPublisher.PublishAsync(user.DomainEvents);
```

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

## Entity Framework Core

### DbContext Organization
- **One DbContext per module** to maintain boundaries.
- Name it after the module: `UsersDbContext`, `OrdersDbContext`.
- Keep in `Infrastructure/Persistence/`.

```csharp
public class UsersDbContext : DbContext
{
    public UsersDbContext(DbContextOptions<UsersDbContext> options) : base(options) { }
    
    public DbSet<User> Users { get; init; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(UsersDbContext).Assembly);
    }
}
```

### Entity Configuration
Use **IEntityTypeConfiguration<T>** for fluent configurations:

```csharp
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.HasKey(u => u.Id);
        builder.Property(u => u.Email).IsRequired().HasMaxLength(255);
        builder.HasIndex(u => u.Email).IsUnique();
    }
}
```

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

### Correlation IDs
Track requests end-to-end via correlation IDs:
```csharp
using (LogContext.PushProperty("CorrelationId", correlationId))
{
    await next();  // All logs in this request include CorrelationId
}
```

### No Secrets in Logs
Never log passwords, tokens, API keys, or sensitive data.
