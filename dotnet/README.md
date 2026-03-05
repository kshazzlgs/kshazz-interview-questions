# .NET Interview Questions

This directory organizes .NET interview questions by topic area.

## Contents

| Topic | File | Description |
|-------|------|-------------|
| C# Language | [csharp/questions.md](csharp/questions.md) | C# language features: type system, OOP, LINQ, async/threading, modern language features |
| .NET Fundamentals | [dotnet-fundamentals/questions.md](dotnet-fundamentals/questions.md) | CLR, GC, DI lifetimes, configuration, options pattern, hosting, background services, logging |
| ASP.NET Core | [aspnet-core/questions.md](aspnet-core/questions.md) | Routing, middleware, security/JWT, minimal APIs, caching, rate limiting, API versioning, EF Core, architecture, testing |

## Quick Reference

### Most Asked Topics

**C# Language**
- `const` vs `readonly`, `ref` vs `out`, boxing/unboxing
- `interface` vs `abstract class`, inheritance vs composition
- Async/await, `Task`, `CancellationToken`
- LINQ deferred execution, `IEnumerable` vs `IQueryable`
- Records, nullable reference types, pattern matching

**.NET Fundamentals**
- DI lifetimes: Singleton, Scoped, Transient
- `IHostedService` and `BackgroundService`
- Options pattern (`IOptions`, `IOptionsSnapshot`, `IOptionsMonitor`)
- Configuration layering and validation
- Structured logging with `ILogger<T>`

**ASP.NET Core**
- Middleware pipeline order and short-circuiting
- JWT authentication and refresh tokens
- Minimal APIs vs MVC Controllers
- Output caching, `IMemoryCache`, `IDistributedCache`
- Rate limiting strategies
- API versioning with `Asp.Versioning`
- Clean Architecture, CQRS, Repository pattern
- EF Core migrations, concurrency, seeding
- Integration testing with `WebApplicationFactory`

---

*See [resources/external-links.md](../resources/external-links.md) for additional study resources.*
