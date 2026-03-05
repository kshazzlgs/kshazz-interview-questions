# .NET Interview Questions

This directory organizes .NET interview questions by topic area.

## Contents

| Topic | File | Description |
|-------|------|-------------|
| C# Language | [csharp/questions.md](csharp/questions.md) | C# language features: type system, OOP, LINQ, async/threading, delegates & events, tuples, dynamic keyword, pattern matching, modern language features |
| .NET Fundamentals | [dotnet-fundamentals/questions.md](dotnet-fundamentals/questions.md) | .NET platform comparison, CLR, garbage collection, assemblies, DI lifetimes, configuration, options pattern, hosting, Generic Host, background services, logging |
| ASP.NET Core | [aspnet-core/questions.md](aspnet-core/questions.md) | Routing, middleware, security/JWT, minimal APIs, caching, rate limiting, API versioning, EF Core, architecture, model binding, error handling, health checks, CORS, SignalR, gRPC, testing |

## Quick Reference

### Most Asked Topics

**C# Language**
- `const` vs `readonly`, `ref` vs `out`, boxing/unboxing
- `interface` vs `abstract class`, inheritance vs composition
- Async/await, `Task`, `CancellationToken`
- LINQ deferred execution, `IEnumerable` vs `IQueryable`
- Records, nullable reference types, pattern matching
- Delegates vs events, `Func`/`Action`/`Predicate`
- Tuples, deconstruction, `dynamic` keyword

**.NET Fundamentals**
- .NET Framework vs .NET Core vs .NET 5+ comparison
- CLR, managed code, and JIT compilation
- Garbage collection: generations, LOH, `IDisposable`
- DI lifetimes: Singleton, Scoped, Transient
- `IHostedService` and `BackgroundService`
- Options pattern (`IOptions`, `IOptionsSnapshot`, `IOptionsMonitor`)
- Configuration layering and validation
- Structured logging with `ILogger<T>`
- Generic Host vs Web Host
- Assemblies, versioning, and loading contexts

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
- Model binding, action results, content negotiation
- Global exception handling and ProblemDetails
- Health checks (liveness & readiness probes)
- CORS configuration and policies
- SignalR for real-time communication
- gRPC services in .NET
- `AddControllers` vs `AddMvc` vs `AddRazorPages`
- Tag Helpers in Razor views

---

*See [resources/external-links.md](../resources/external-links.md) for additional study resources.*
