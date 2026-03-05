# .NET Fundamentals Interview Questions

> **Source attribution:** Content adapted from [aam9063/dotnet-interview-questions](https://github.com/aam9063/dotnet-interview-questions) by Albert Alarcón Martínez. Used with attribution. See [third-party notices](../../LICENSES/third-party-notices.md).

## Table of Contents

- [Dependency Injection](#dependency-injection)
  - [Q: What Is the Difference Between Singleton, Scoped, and Transient Services?](#q-what-is-the-difference-between-singleton-scoped-and-transient-services)
  - [Q: How to Use a Scoped Service Inside a Singleton Service?](#q-how-to-use-a-scoped-service-inside-a-singleton-service)
- [Application Lifecycle & Hosting](#application-lifecycle--hosting)
  - [Q: How to Execute Code When the Application Is Starting and Stopping?](#q-how-to-execute-code-when-the-application-is-starting-and-stopping)
  - [Q: What Is a Background Service in ASP.NET Core?](#q-what-is-a-background-service-in-aspnet-core)
  - [Q: What Is `HostedService` Used For in ASP.NET Core?](#q-what-is-hostedservice-used-for-in-aspnet-core)
  - [Q: Explain the Difference Between `PeriodicTimer` and `await Task.Delay()`](#q-explain-the-difference-between-periodictimer-and-await-taskdelay)
  - [Q: Difference Between `IHostedService`, `BackgroundService`, and `IAsyncDisposable`](#q-difference-between-ihostedservice-backgroundservice-and-iasyncdisposable)
  - [Q: How Can You Deploy an ASP.NET Core Application?](#q-how-can-you-deploy-an-aspnet-core-application)
- [Configuration & Options](#configuration--options)
  - [Q: Name a Few Ways to Read Data from `appsettings.json` Configuration](#q-name-a-few-ways-to-read-data-from-appsettingsjson-configuration)
  - [Q: What Is the Options Pattern in ASP.NET Core?](#q-what-is-the-options-pattern-in-aspnet-core)
  - [Q: Name the Use Cases for `IOptionsSnapshot` and `IOptionsMonitor`](#q-name-the-use-cases-for-ioptionssnapshot-and-ioptionsmonitor)
  - [Q: How to Validate Configuration in ASP.NET Core?](#q-how-to-validate-configuration-in-aspnet-core)
  - [Q: What Is the Difference Between DataAnnotations and FluentValidation?](#q-what-is-the-difference-between-dataannotations-and-fluentvalidation)
- [Logging](#logging)
  - [Q: How to Configure Logging in ASP.NET Core?](#q-how-to-configure-logging-in-aspnet-core)

---

## Dependency Injection

### Q: What Is the Difference Between Singleton, Scoped, and Transient Services?

**A:**

In ASP.NET Core, you register services in the **Dependency Injection (DI) container**. When registering services, you choose their **lifetime**, which controls **how and when** instances are created.

---

#### Singleton

- **One instance** is created for the **entire application lifetime**.
- The same instance is **shared across all requests and users**.

```csharp
builder.Services.AddSingleton<IMyService, MyService>();
```

Use when:
- The service is **stateless** or **expensive to create**.
- You want to **cache** or **share resources**.

Be careful: avoid using `HttpContext` or per-request data; not thread-safe by default.

---

#### Scoped

- A **new instance is created per HTTP request**.
- All components within that request share the same instance.

```csharp
builder.Services.AddScoped<IMyService, MyService>();
```

Use when:
- You need to maintain **state within a single request** (e.g., database context).
- Safe to use services that depend on the current request (e.g., `HttpContextAccessor`).

---

#### Transient

- A **new instance is created every time** the service is requested.

```csharp
builder.Services.AddTransient<IMyService, MyService>();
```

Use when:
- The service is **lightweight** and **stateless**.
- You don't need to share state at all.

---

#### Practical Example

```csharp
public interface IOperationService
{
    Guid Id { get; }
}

public class OperationService : IOperationService
{
    public Guid Id { get; } = Guid.NewGuid();
}
```

Register services:

```csharp
builder.Services.AddSingleton<IOperationService, OperationService>();   // Singleton
builder.Services.AddScoped<IOperationService, OperationService>();      // Scoped
builder.Services.AddTransient<IOperationService, OperationService>();   // Transient
```

Inject into controller:

```csharp
public class DemoController : Controller
{
    private readonly IOperationService _service1;
    private readonly IOperationService _service2;

    public DemoController(IOperationService service1, IOperationService service2)
    {
        _service1 = service1;
        _service2 = service2;
    }

    public IActionResult Index()
    {
        return Content($"Service1 ID: {_service1.Id} | Service2 ID: {_service2.Id}");
    }
}
```

- **Singleton**: Both IDs are **the same** across all requests.
- **Scoped**: Same ID within one request, different across requests.
- **Transient**: Different ID every time it is injected.

---

#### Summary Table

| Lifetime  | Instance Per | Shared Between | Typical Use Case                              |
| --------- | ------------ | -------------- | --------------------------------------------- |
| Singleton | Application  | All requests   | Logging, caching, config, DI container itself |
| Scoped    | HTTP Request | Same request   | DbContext, business logic                     |
| Transient | Every call   | No sharing     | Lightweight services, mapping                 |

---

### Q: How to Use a Scoped Service Inside a Singleton Service?

**A:**

Directly injecting a **scoped service into a singleton** is **not allowed** and will cause a runtime exception or incorrect behavior.

> ❌ Why? Because scoped services are **tied to the HTTP request**, and singleton services **live for the entire app lifetime**. Mixing them can cause memory leaks or unexpected results.

---

#### Recommended Solution: Use `IServiceScopeFactory`

```csharp
public interface IScopedService
{
    string GetRequestId();
}

public class ScopedService : IScopedService
{
    private readonly string _requestId = Guid.NewGuid().ToString();

    public string GetRequestId() => _requestId;
}
```

Singleton that depends on scoped service:

```csharp
public class MySingletonService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public MySingletonService(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    public void DoWork()
    {
        using var scope = _scopeFactory.CreateScope();
        var scopedService = scope.ServiceProvider.GetRequiredService<IScopedService>();
        Console.WriteLine($"Scoped Request ID: {scopedService.GetRequestId()}");
    }
}
```

Register in `Program.cs`:

```csharp
builder.Services.AddScoped<IScopedService, ScopedService>();
builder.Services.AddSingleton<MySingletonService>();
```

---

#### Alternative: Using `IServiceProvider` directly (less preferred)

```csharp
public class MySingletonService
{
    private readonly IServiceProvider _serviceProvider;

    public MySingletonService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public void DoWork()
    {
        using var scope = _serviceProvider.CreateScope();
        var scoped = scope.ServiceProvider.GetRequiredService<IScopedService>();
        Console.WriteLine(scoped.GetRequestId());
    }
}
```

> ⚠️ Warning: Avoid storing `IServiceProvider` long-term inside the singleton. Only use it temporarily within the method that needs the scoped dependency.

---

#### What NOT to do

This will **compile** but cause **runtime issues** or unexpected behavior:

```csharp
// BAD PRACTICE: Injecting a scoped service directly into singleton
public class BadSingleton
{
    public BadSingleton(IScopedService scoped) { }
}
```

---

#### Summary Table

| Approach                       | Safe? | Description                                     |
| ------------------------------ | ----- | ----------------------------------------------- |
| Inject `IServiceScopeFactory`  | ✅     | Recommended: short-lived scope inside singleton |
| Inject `IServiceProvider`      | ✅     | Acceptable: create scope manually               |
| Inject scoped service directly | ❌     | Dangerous: breaks DI lifetime rules             |

---

## Application Lifecycle & Hosting

### Q: How to Execute Code When the Application Is Starting and Stopping?

**A:**

In ASP.NET Core, you can hook into application **lifetime events** to run custom logic **on startup** and **on shutdown**.

ASP.NET Core provides `IHostApplicationLifetime` to access:

- `ApplicationStarted` – app has fully started
- `ApplicationStopping` – app is shutting down (graceful)
- `ApplicationStopped` – app has fully stopped

---

#### Using `Program.cs` in .NET 6 / 7 / 8

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var lifetime = app.Lifetime;

lifetime.ApplicationStarted.Register(() =>
{
    Console.WriteLine("Application Started");
});

lifetime.ApplicationStopping.Register(() =>
{
    Console.WriteLine("Application Stopping");
});

lifetime.ApplicationStopped.Register(() =>
{
    Console.WriteLine("Application Stopped");
});

app.Run();
```

---

#### Using `IHostedService` for Long-Running Background Tasks

You can create a class that implements `IHostedService` to run logic on app start and shutdown:

```csharp
public class StartupTask : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Startup logic here");
        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Shutdown logic here");
        return Task.CompletedTask;
    }
}
```

Register it:

```csharp
builder.Services.AddHostedService<StartupTask>();
```

---

#### Summary Table

| Action               | Best Approach                                                  |
| -------------------- | -------------------------------------------------------------- |
| Run code on startup  | `ApplicationStarted.Register()` or `StartAsync()`             |
| Run code on shutdown | `ApplicationStopping` / `ApplicationStopped` or `StopAsync()` |
| For background logic | Implement `IHostedService`                                     |

---

### Q: What Is a Background Service in ASP.NET Core?

**A:**

A **Background Service** is a long-running task that executes in the background independently of incoming HTTP requests.

In ASP.NET Core, background services are typically implemented using the **`BackgroundService`** base class, which is part of the **`Microsoft.Extensions.Hosting`** namespace.

---

#### Use Cases

- Processing queued messages
- Periodic jobs (e.g., cleanup, reporting)
- Polling external APIs
- Sending emails
- Listening to events from a queue (e.g., RabbitMQ, Kafka)

---

#### How to Create a Background Service

Step 1 — Create the service:

```csharp
public class MyBackgroundService : BackgroundService
{
    private readonly ILogger<MyBackgroundService> _logger;

    public MyBackgroundService(ILogger<MyBackgroundService> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Background service is starting.");

        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Background service is working...");
            await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
        }

        _logger.LogInformation("Background service is stopping.");
    }
}
```

Step 2 — Register the service in `Program.cs`:

```csharp
builder.Services.AddHostedService<MyBackgroundService>();
```

---

#### Lifecycle

A `BackgroundService` starts **when the application starts**, and it stops **gracefully** when the app is shutting down. Implement logic in `ExecuteAsync(CancellationToken stoppingToken)` and always **listen to the cancellation token** to stop gracefully.

---

#### Best Practices

- Always monitor the `stoppingToken` to shut down gracefully.
- Avoid infinite loops without `await` — they can freeze the thread.
- Use `try/catch` inside the loop to prevent crashes.
- Offload heavy work using channels or queues.

---

#### Summary Table

| Option              | Use Case                                        |
| ------------------- | ----------------------------------------------- |
| `IHostedService`    | Base interface for background workers           |
| `BackgroundService` | Abstract class for long-running background task |
| `Task.Run()` in startup | ❌ Not recommended for long-term processing  |
| Timer-based tasks   | Use `System.Threading.Timer` in hosted service  |

---

### Q: What Is `HostedService` Used For in ASP.NET Core?

**A:**

A **Hosted Service** in ASP.NET Core is a background task that runs **alongside the web application** — outside the request/response cycle.

It implements the **`IHostedService`** interface or derives from **`BackgroundService`**, and is managed by the ASP.NET Core **generic host**.

---

#### Common Use Cases

| Use Case                         | Description                        |
| -------------------------------- | ---------------------------------- |
| Long-running background tasks    | e.g., queue processing, workers    |
| Periodic jobs                    | e.g., cleanup tasks, status checks |
| Polling external services        | e.g., API watchers, RSS fetchers   |
| Scheduled notifications          | e.g., email reminders, alerts      |
| Message queue consumers          | e.g., RabbitMQ, Kafka consumers    |

---

#### Example: Basic Hosted Service

```csharp
public class MyBackgroundWorker : BackgroundService
{
    private readonly ILogger<MyBackgroundWorker> _logger;

    public MyBackgroundWorker(ILogger<MyBackgroundWorker> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Background service is starting.");

        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Working at: {time}", DateTimeOffset.Now);
            await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
        }

        _logger.LogInformation("Background service is stopping.");
    }
}
```

Register the service in `Program.cs`:

```csharp
builder.Services.AddHostedService<MyBackgroundWorker>();
```

---

#### Lifecycle Methods in `IHostedService`

If you implement `IHostedService` manually (not via `BackgroundService`):

```csharp
public class MyWorker : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken)
    {
        // Initialization logic
        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        // Cleanup logic
        return Task.CompletedTask;
    }
}
```

---

#### Hosted Service with Dependency Injection

You can inject any service into the constructor:

```csharp
public MyWorker(ILogger<MyWorker> logger, IMyService service) { ... }
```

> The hosted service itself is **registered as singleton**, but injected services can be **scoped** via `IServiceScopeFactory`.

---

#### Best Practices

- Always check `stoppingToken.IsCancellationRequested` in loops.
- Use `try/catch` inside `ExecuteAsync` to avoid crashing the host.
- For scoped dependencies (e.g., `DbContext`), use `IServiceScopeFactory`.

---

#### Summary Table

| Feature             | Explanation                                       |
| ------------------- | ------------------------------------------------- |
| `IHostedService`    | Interface to implement background tasks           |
| `BackgroundService` | Base class with `ExecuteAsync()`                  |
| Lifetime            | Runs from app start to shutdown                   |
| Common use cases    | Background jobs, queue consumers, scheduled tasks |
| Registration        | Via `AddHostedService<T>()` in DI                 |

---

### Q: Explain the Difference Between `PeriodicTimer` and `await Task.Delay()`

**A:**

Both `PeriodicTimer` and `Task.Delay()` are used for **delaying execution in asynchronous code**, especially in **background services**, but they differ significantly in **precision**, **control**, and **design purpose**.

---

#### `await Task.Delay(...)`

A general-purpose method that delays execution for a **specific duration**, commonly used in loops to simulate periodic tasks.

```csharp
while (!cancellationToken.IsCancellationRequested)
{
    DoWork();
    await Task.Delay(TimeSpan.FromSeconds(5), cancellationToken);
}
```

**Problem:** The delay happens **after** `DoWork()`, so total interval = `DoWork duration + delay`, causing **drift** over time.

---

#### `PeriodicTimer`

Introduced in **.NET 6**, a **specialized timer** for running periodic tasks with **accurate intervals**, regardless of task duration.

```csharp
var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));

while (await timer.WaitForNextTickAsync(cancellationToken))
{
    DoWork();
}
```

Automatically maintains a **steady interval** between ticks. If `DoWork()` takes too long, ticks may be **skipped**, avoiding backlog.

---

#### Side-by-Side Comparison

| Feature              | `Task.Delay()`                        | `PeriodicTimer`                             |
| -------------------- | ------------------------------------- | ------------------------------------------- |
| Delay behavior       | Manual delay after work               | Waits for next tick                         |
| Drift control        | ❌ Drift increases with task duration  | ✅ Maintains accurate intervals              |
| Skips missed ticks   | ❌ No — just delays again              | ✅ Yes — does not queue missed ticks         |
| Cancellation support | ✅ via `CancellationToken`             | ✅ via `WaitForNextTickAsync(token)`         |
| Best use case        | Simple delays or one-off waits        | Accurate recurring tasks in background jobs |
| Introduced in        | .NET 4.5+                             | .NET 6 and later                            |

---

#### When to Use What?

| Scenario                           | Recommended Option |
| ---------------------------------- | ------------------ |
| Run a task every fixed interval    | `PeriodicTimer`    |
| Delay after a task finishes        | `Task.Delay()`     |
| Need precise scheduling            | `PeriodicTimer`    |
| Need compatibility with .NET 5/4.8 | `Task.Delay()`     |

---

#### Example in a BackgroundService

Using `Task.Delay` (includes work time in interval):

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        DoWork();
        await Task.Delay(5000, stoppingToken);
    }
}
```

Using `PeriodicTimer` (task duration doesn't affect next tick):

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));

    while (await timer.WaitForNextTickAsync(stoppingToken))
    {
        DoWork();
    }
}
```

---

### Q: Difference Between `IHostedService`, `BackgroundService`, and `IAsyncDisposable`

**A:**

| Type                | Description                                               | Use Case                   |
| ------------------- | --------------------------------------------------------- | -------------------------- |
| `IHostedService`    | Interface for background tasks during app lifetime        | Custom hosted logic        |
| `BackgroundService` | Abstract class that simplifies `IHostedService`           | Long-running jobs          |
| `IAsyncDisposable`  | Cleanup logic for async resources (e.g. streams, sockets) | Graceful resource disposal |

---

#### `BackgroundService` Example

```csharp
public class Worker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine("Running task...");
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

Register in DI:

```csharp
builder.Services.AddHostedService<Worker>();
```

---

#### `IAsyncDisposable` Example

```csharp
public class MyDisposableService : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await _connection.DisposeAsync();
    }
}
```

---

### Q: How Can You Deploy an ASP.NET Core Application?

**A:**

ASP.NET Core is **cross-platform**, so you can deploy it on Linux, Windows Server, cloud services, or Docker containers.

---

#### Deployment Targets

| Target                | Description                              |
| --------------------- | ---------------------------------------- |
| **Azure App Service** | PaaS for hosting .NET apps in minutes    |
| **IIS on Windows**    | Host on-prem apps on Windows Server      |
| **Linux + Nginx**     | Host via reverse proxy (Kestrel + Nginx) |
| **Docker**            | Containerized deployment                 |
| **Kubernetes**        | Scalable container orchestration         |

---

#### Deployment Options

**Self-Contained Deployment (SCD):** Publishes the app with the entire .NET runtime (no need to install .NET on host).

```bash
dotnet publish -c Release -r linux-x64 --self-contained true -o ./publish
```

**Framework-Dependent Deployment (FDD):** Smaller, but requires .NET runtime on the server.

```bash
dotnet publish -c Release -o ./publish
```

---

#### Azure App Service

```bash
az webapp up --name MyAspApp --resource-group MyGroup --runtime "DOTNET|8.0"
```

---

#### Linux + Nginx + Kestrel

1. Publish your app:

```bash
dotnet publish -c Release -o /var/www/myapp
```

2. Create a systemd service:

```ini
[Unit]
Description=My ASP.NET Core App

[Service]
WorkingDirectory=/var/www/myapp
ExecStart=/usr/bin/dotnet MyApp.dll
Restart=always
User=www-data

[Install]
WantedBy=multi-user.target
```

3. Configure **Nginx** reverse proxy:

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

#### Deploy via Docker

Create a `Dockerfile`:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

Build and run:

```bash
docker build -t myapp .
docker run -d -p 8080:80 myapp
```

---

#### Summary Table

| Option            | Best For                                 |
| ----------------- | ---------------------------------------- |
| Azure App Service | Fast deployment with scaling + CI/CD     |
| IIS on Windows    | Enterprises using Windows infrastructure |
| Linux + Nginx     | Lightweight and flexible                 |
| Docker            | Cloud-native and reproducible            |
| Kubernetes        | Microservices or large-scale apps        |

---

## Configuration & Options

### Q: Name a Few Ways to Read Data from `appsettings.json` Configuration

**A:**

In ASP.NET Core, the `appsettings.json` file is commonly used to store configuration data such as connection strings, custom settings, feature toggles, etc.

Example `appsettings.json`:

```json
{
  "AppSettings": {
    "SiteName": "MyApp",
    "MaxItems": 10
  },
  "ConnectionStrings": {
    "Default": "Server=.;Database=MyDb;Trusted_Connection=True;"
  }
}
```

---

#### 1. Using `IConfiguration` Interface (Key-Based Access)

```csharp
public class HomeController : Controller
{
    private readonly IConfiguration _config;

    public HomeController(IConfiguration config)
    {
        _config = config;
    }

    public IActionResult Index()
    {
        var siteName = _config["AppSettings:SiteName"];
        var connection = _config.GetConnectionString("Default");

        return Content($"Site: {siteName} | DB: {connection}");
    }
}
```

Use colon (`:`) for nested sections. Use `GetConnectionString()` for the `ConnectionStrings` section.

---

#### 2. Binding to a POCO (Plain C# Class)

```csharp
public class AppSettings
{
    public string SiteName { get; set; }
    public int MaxItems { get; set; }
}
```

Register it in `Program.cs`:

```csharp
builder.Services.Configure<AppSettings>(builder.Configuration.GetSection("AppSettings"));
```

Inject using `IOptions<T>`:

```csharp
public class HomeController : Controller
{
    private readonly AppSettings _settings;

    public HomeController(IOptions<AppSettings> options)
    {
        _settings = options.Value;
    }

    public IActionResult Index()
    {
        return Content($"Site: {_settings.SiteName}, Max: {_settings.MaxItems}");
    }
}
```

---

#### 3. Using `GetSection().Get<T>()` for Manual Binding

No need for `IOptions`, useful in non-DI contexts:

```csharp
var settings = builder.Configuration.GetSection("AppSettings").Get<AppSettings>();
Console.WriteLine(settings.SiteName);
```

---

#### 4. Using `IOptionsSnapshot<T>` or `IOptionsMonitor<T>`

- `IOptionsSnapshot<T>` – for **scoped** lifetime, gets updated per request.
- `IOptionsMonitor<T>` – for **singleton** services, detects **real-time changes** if `reloadOnChange` is enabled.

```csharp
public class MyService
{
    private readonly AppSettings _currentSettings;

    public MyService(IOptionsMonitor<AppSettings> monitor)
    {
        _currentSettings = monitor.CurrentValue;
    }
}
```

---

#### Summary Table

| Method                                 | Use Case                           | Lifetime         |
| -------------------------------------- | ---------------------------------- | ---------------- |
| `IConfiguration["Key"]`                | Simple key access                  | Any              |
| `IConfiguration.GetSection().Get<T>()` | Bind to class manually             | Any              |
| `IOptions<T>`                          | Auto-bind class via DI             | Singleton/Scoped |
| `IOptionsSnapshot<T>`                  | Updated on each request            | Scoped           |
| `IOptionsMonitor<T>`                   | Live-updating, background services | Singleton        |

---

### Q: What Is the Options Pattern in ASP.NET Core?

**A:**

The **Options Pattern** in ASP.NET Core is a way to **bind configuration sections** (like from `appsettings.json`) to **strongly-typed classes** and inject them using **dependency injection (DI)**.

It helps keep configuration code clean, type-safe, and maintainable.

---

#### Why Use the Options Pattern?

- Strongly-typed access to config values
- Supports validation and reload on change
- Follows dependency injection principles

---

#### Step-by-Step Implementation

Example `appsettings.json`:

```json
{
  "AppSettings": {
    "SiteTitle": "MyApp",
    "MaxItems": 10
  }
}
```

Step 1 — Create a POCO class:

```csharp
public class AppSettings
{
    public string SiteTitle { get; set; }
    public int MaxItems { get; set; }
}
```

Step 2 — Register the configuration in `Program.cs`:

```csharp
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));
```

Step 3 — Inject using `IOptions<T>` in a service or controller:

```csharp
public class HomeController : Controller
{
    private readonly AppSettings _settings;

    public HomeController(IOptions<AppSettings> options)
    {
        _settings = options.Value;
    }

    public IActionResult Index()
    {
        return Content($"Title: {_settings.SiteTitle}, Max: {_settings.MaxItems}");
    }
}
```

---

#### Advanced Variants

**`IOptionsSnapshot<T>`** — for scoped lifetimes (e.g., per HTTP request), captures configuration as it was at the start of the request:

```csharp
public HomeController(IOptionsSnapshot<AppSettings> options)
```

**`IOptionsMonitor<T>`** — for singleton services, automatically **detects changes** in config if `reloadOnChange: true`:

```csharp
public class BackgroundWorker
{
    public BackgroundWorker(IOptionsMonitor<AppSettings> monitor)
    {
        var settings = monitor.CurrentValue;
    }
}
```

---

#### Common Mistakes

- Forgetting to register `.Configure<T>()` in `Program.cs`
- Using `IOptionsSnapshot` in singleton services (not allowed)
- Accessing config directly with strings (`_config["SomeKey"]`) — less safe

---

#### Summary Table

| Interface             | Lifetime  | Reload on Change | Use Case                     |
| --------------------- | --------- | ---------------- | ---------------------------- |
| `IOptions<T>`         | Singleton | ❌                | Basic usage in most services |
| `IOptionsSnapshot<T>` | Scoped    | ✅ (per request)  | Web apps (controllers)       |
| `IOptionsMonitor<T>`  | Singleton | ✅ (real-time)    | Background workers, caching  |

---

### Q: Name the Use Cases for `IOptionsSnapshot` and `IOptionsMonitor`

**A:**

ASP.NET Core provides `IOptions<T>`, `IOptionsSnapshot<T>`, and `IOptionsMonitor<T>` to access configuration data. Each serves a different purpose depending on the **lifetime** of the service and the **need for runtime updates**.

---

#### `IOptionsSnapshot<T>`

Retrieves a **fresh configuration instance per HTTP request** (scoped lifetime). Suitable for **web applications** where config may change between requests.

| Use Case                    | Description                                                                         |
| --------------------------- | ----------------------------------------------------------------------------------- |
| Per-request configuration   | Inject in a controller or scoped service to get updated values per request          |
| Testing new features        | Useful when config values are toggled frequently (e.g., A/B testing, feature flags) |
| Multi-tenant apps           | Each request might load a different tenant's configuration                          |

Not suitable for singleton services (throws error).

---

#### `IOptionsMonitor<T>`

**Singleton-safe** and supports **real-time config changes**. Monitors the underlying configuration file (e.g., `appsettings.json`) and triggers a **callback when config changes**.

| Use Case                    | Description                                                                      |
| --------------------------- | -------------------------------------------------------------------------------- |
| Background services         | Use in long-running or singleton services (e.g., workers, hosted services)       |
| Live configuration reloads  | Auto-reloads values when `appsettings.json` changes and `reloadOnChange` is true |
| On-change notifications     | Subscribe to `OnChange()` for reacting to config updates                         |
| Caching and tuning          | Useful for changing thresholds or feature toggles without restarting the app     |

Example: reacting to changes with `IOptionsMonitor`:

```csharp
public class FeatureToggleService
{
    public FeatureToggleService(IOptionsMonitor<AppSettings> monitor)
    {
        monitor.OnChange(settings =>
        {
            Console.WriteLine($"Config updated: FeatureX = {settings.EnableFeatureX}");
        });
    }
}
```

---

#### Summary Table

| Feature                  | `IOptionsSnapshot<T>`         | `IOptionsMonitor<T>`                          |
| ------------------------ | ----------------------------- | --------------------------------------------- |
| Lifetime                 | Scoped                        | Singleton                                     |
| Refresh on change        | Yes (per request)             | Yes (real-time)                               |
| Use in controllers       | ✅                             | ✅                                             |
| Use in singletons        | ❌                             | ✅                                             |
| Detect changes via event | ❌                             | ✅ `.OnChange()`                               |
| Common use case          | Per-request config (web apps) | Live monitoring (background services, caches) |

---

### Q: How to Validate Configuration in ASP.NET Core?

**A:**

When using the **Options pattern** to bind config sections, it's important to ensure the values are not null or empty, fall within an expected range, and meet required business rules. If misconfigured, the app should **fail fast** at startup rather than throw runtime exceptions.

---

#### 1. Data Annotations + `ValidateDataAnnotations()`

POCO with annotations:

```csharp
public class AppSettings
{
    [Required]
    public string SiteTitle { get; set; }

    [Range(1, 100)]
    public int MaxItems { get; set; }
}
```

Register and validate in `Program.cs`:

```csharp
builder.Services.AddOptions<AppSettings>()
    .Bind(builder.Configuration.GetSection("AppSettings"))
    .ValidateDataAnnotations();
```

---

#### 2. Custom Validation with `.Validate(...)`

```csharp
builder.Services.AddOptions<AppSettings>()
    .Bind(builder.Configuration.GetSection("AppSettings"))
    .Validate(settings => settings.MaxItems > 0 && settings.MaxItems < 100,
              "MaxItems must be between 1 and 99");
```

---

#### 3. Fail Fast on Startup with `.ValidateOnStart()`

By default, validation happens **only when the option is first accessed**. To **fail at app startup**, add `.ValidateOnStart()`:

```csharp
builder.Services.AddOptions<AppSettings>()
    .Bind(builder.Configuration.GetSection("AppSettings"))
    .ValidateDataAnnotations()
    .Validate(settings => settings.SiteTitle.StartsWith("My"), "Must start with 'My'")
    .ValidateOnStart();
```

---

#### Advanced: Use `IValidateOptions<T>`

For reusable, injectable validators:

```csharp
public class AppSettingsValidator : IValidateOptions<AppSettings>
{
    public ValidateOptionsResult Validate(string name, AppSettings settings)
    {
        if (string.IsNullOrWhiteSpace(settings.SiteTitle))
            return ValidateOptionsResult.Fail("SiteTitle is required.");

        if (settings.MaxItems <= 0)
            return ValidateOptionsResult.Fail("MaxItems must be positive.");

        return ValidateOptionsResult.Success;
    }
}
```

Register it:

```csharp
builder.Services.AddSingleton<IValidateOptions<AppSettings>, AppSettingsValidator>();
```

---

#### Summary Table

| Method                      | Description                           | Fails on startup?            |
| --------------------------- | ------------------------------------- | ---------------------------- |
| `ValidateDataAnnotations()` | Uses `[Required]`, `[Range]`, etc.    | ❌ (unless `ValidateOnStart`) |
| `Validate(...)`             | Custom logic with lambda              | ❌ (unless `ValidateOnStart`) |
| `ValidateOnStart()`         | Forces validation on app startup      | ✅                            |
| `IValidateOptions<T>`       | Full custom reusable validation logic | ✅                            |

---

### Q: What Is the Difference Between DataAnnotations and FluentValidation?

**A:**

Both **Data Annotations** and **FluentValidation** are used to validate input data in ASP.NET Core. They differ in **flexibility, maintainability, and separation of concerns**.

---

#### Data Annotations (Built-in)

Validation attributes added **directly to model properties** using `[Required]`, `[StringLength]`, `[Range]`, etc.

```csharp
public class Product
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }

    [Range(1, 999)]
    public decimal Price { get; set; }
}
```

**Pros:**
- Built-in to .NET
- Simple for small models
- Recognized by MVC, Blazor, Swagger, etc.

**Cons:**
- Validation logic is **mixed with your model**
- Not ideal for **complex rules or conditions**
- Less flexible and hard to unit test

---

#### FluentValidation (External Library)

A **separate class-based validation library** using a fluent API — highly customizable and testable.

Install via NuGet:

```bash
dotnet add package FluentValidation.AspNetCore
```

Example:

```csharp
public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class ProductValidator : AbstractValidator<Product>
{
    public ProductValidator()
    {
        RuleFor(p => p.Name)
            .NotEmpty()
            .MaximumLength(100);

        RuleFor(p => p.Price)
            .InclusiveBetween(1, 999);
    }
}
```

Register it in `Program.cs`:

```csharp
builder.Services.AddValidatorsFromAssemblyContaining<ProductValidator>();
builder.Services.AddFluentValidationAutoValidation();
```

**Pros:**
- **Separation of concerns** (model != validation logic)
- Fluent and readable API
- Easily unit testable
- Conditional validation (`When(...)`)
- Complex rules and rule sets

**Cons:**
- Requires additional NuGet package
- Slightly more setup for small apps

---

#### Comparison Table

| Feature                    | Data Annotations   | FluentValidation               |
| -------------------------- | ------------------ | ------------------------------ |
| Built-in support           | ✅ Yes              | ❌ Requires NuGet package       |
| External dependency        | ❌ None             | ✅ Yes (`FluentValidation`)     |
| Separation of concerns     | ❌ Mixed with model | ✅ Separate validator classes   |
| Conditional validation     | ❌ Limited          | ✅ Powerful `When(...)` support |
| Unit testable              | ❌ Hard             | ✅ Easily testable              |
| Complex rules              | ❌ Not ideal        | ✅ Fully supported              |
| Suitable for large apps    | ⚠️ Limited         | ✅ Highly recommended           |

---

#### Summary

Use **Data Annotations** when:
- You have **simple** validations
- You want to **keep things minimal**
- You work in **Blazor or MVC with metadata**

Use **FluentValidation** when:
- You need **advanced or conditional** logic
- You want **testable, clean code**
- You're working on **scalable projects**

---

## Logging

### Q: How to Configure Logging in ASP.NET Core?

**A:**

ASP.NET Core provides a **built-in logging framework** that is lightweight and extensible, supports **structured logging**, and works with **multiple providers** (Console, Debug, File, Seq, Serilog, etc.).

---

#### Built-in Logging Providers

| Provider      | Description                        |
| ------------- | ---------------------------------- |
| Console       | Logs to terminal/console           |
| Debug         | Logs to Visual Studio Debug output |
| EventSource   | Windows EventSource                |
| EventLog      | Windows Event Log                  |
| Azure Monitor | Cloud logging                      |

---

#### Configure Logging in `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

// Clear default providers (optional)
builder.Logging.ClearProviders();

// Add desired providers
builder.Logging.AddConsole();
builder.Logging.AddDebug();

// Set minimum level globally
builder.Logging.SetMinimumLevel(LogLevel.Information);
```

---

#### Configure Logging Levels in `appsettings.json`

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "MyAppNamespace": "Debug"
    }
  }
}
```

---

#### Use `ILogger<T>` in Services/Controllers

```csharp
public class UserService
{
    private readonly ILogger<UserService> _logger;

    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }

    public void CreateUser(User user)
    {
        _logger.LogInformation("Creating user: {Name}", user.Name);

        try
        {
            // Logic
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating user");
        }
    }
}
```

---

#### Logging Levels

| Level         | Description                       |
| ------------- | --------------------------------- |
| `Trace`       | Most detailed (diagnostic)        |
| `Debug`       | Development-only info             |
| `Information` | High-level application events     |
| `Warning`     | Unexpected but recoverable issues |
| `Error`       | Failed operations                 |
| `Critical`    | System is unusable                |
| `None`        | No logging                        |

---

#### Add Third-Party Logging (e.g., Serilog)

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
```

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateLogger();

builder.Host.UseSerilog();
```

---

#### Structured Log Output

```csharp
_logger.LogInformation("User {UserId} logged in at {Time}", user.Id, DateTime.UtcNow);
```

Output:

```
info: UserService[0]
      User 42 logged in at 2025-06-24T14:00:00Z
```

---

#### Summary Table

| Task                      | How to Do It                                 |
| ------------------------- | -------------------------------------------- |
| Add basic logging         | Use `ILogger<T>` and `AddConsole()`          |
| Configure per environment | Use `appsettings.{Environment}.json`         |
| Customize per category    | Set `LogLevel` in `appsettings.json`         |
| Use structured logging    | Use `{placeholder}` format                   |
| Add file/cloud providers  | Use Serilog, NLog, Seq, Application Insights |
