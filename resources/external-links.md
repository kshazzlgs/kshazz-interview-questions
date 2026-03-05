# External Resources & Further Reading

A curated list of external resources for .NET and ASP.NET Core interview preparation. These sites offer practice questions, tutorials, and real-world examples beyond what is covered in this repository.

> **Note:** No verbatim content has been copied from any of these sources. The summaries below are original descriptions. Interview topics covered by these resources have been used as inspiration for the original questions written in this repository.

---

## .NET & C# General

### amitpnk – Interview Questions

**URL:** <https://amitpnk.github.io/interview-questions>

A GitHub Pages site maintained by Amit Kumar that aggregates interview questions across multiple technology stacks including C#, .NET, SQL, and Azure. Useful as a quick-reference checklist before interviews, covering both conceptual questions and practical scenarios at beginner-to-intermediate level.

**Topics covered:** C# fundamentals (delegates, events, generics, collections), .NET framework comparison, OOP concepts, LINQ, async/await, design patterns, SQL basics, and Azure services.

---

## ASP.NET Core & Web API

### DotNetTutorials – ASP.NET Core Web API Interview Questions

**URL:** <https://dotnettutorials.net/lesson/asp-net-core-web-api-interview-questions/>

DotNetTutorials is a well-known tutorial site with long-form articles on .NET topics. This lesson covers frequently-asked Web API interview questions with explanations of concepts like routing, action results, model binding, content negotiation, and API security. Suitable for intermediate to advanced candidates.

**Topics covered:** Model binding sources (`[FromBody]`, `[FromQuery]`, etc.), action result types, content negotiation, global error handling, middleware pipeline, dependency injection, and API versioning.

### WeCreateProblems – .NET Core Interview Questions

**URL:** <https://www.wecreateproblems.com/interview-questions/dot-net-core-interview-questions>

A dedicated interview-prep platform with categorized .NET Core questions covering fundamentals, dependency injection, middleware, deployment, and more. Questions are presented with concise answers, making it a good last-minute review resource.

**Topics covered:** .NET Core vs .NET Framework differences, CLR and managed code, garbage collection, health checks, CORS configuration, middleware vs filters, SignalR basics, and deployment strategies.

### InterviewBit – ASP.NET Interview Questions

**URL:** <https://www.interviewbit.com/asp-net-interview-questions/>

InterviewBit's ASP.NET section covers a broad range of questions spanning classic ASP.NET and ASP.NET Core. Topics include the page lifecycle, state management, authentication, MVC architecture, and Web API design. Particularly useful for candidates who need to know both legacy ASP.NET and modern Core concepts.

**Topics covered:** ASP.NET Core fundamentals, `AddMvc` vs `AddControllers` vs `AddRazorPages`, Tag Helpers, Razor syntax, authentication & authorization, gRPC, SignalR, service lifetimes, and configuration management.

---

## GitHub Repositories

### aam9063/dotnet-interview-questions

**URL:** <https://github.com/aam9063/dotnet-interview-questions>

75 ASP.NET Core questions with detailed Markdown answers authored by Albert Alarcón Martínez. Topics include middleware, JWT auth, minimal APIs, caching, rate limiting, API versioning, clean architecture, EF Core, and CI/CD. Content from this repository has been imported and reorganized into this repo's `dotnet/` folder (see [third-party notices](../LICENSES/third-party-notices.md)).

### kumarvna/interview-questions

**URL:** <https://github.com/kumarvna/interview-questions>

A repository of interview questions on various .NET-related topics (GitHub user: kumarvna). The repository was not publicly accessible at time of import; linked here for reference.

---

## Official Documentation

- **Microsoft ASP.NET Core Documentation:** <https://learn.microsoft.com/en-us/aspnet/core/>
- **C# Language Reference:** <https://learn.microsoft.com/en-us/dotnet/csharp/>
- **.NET API Browser:** <https://learn.microsoft.com/en-us/dotnet/api/>
- **Entity Framework Core Documentation:** <https://learn.microsoft.com/en-us/ef/core/>
- **SignalR Documentation:** <https://learn.microsoft.com/en-us/aspnet/core/signalr/>
- **gRPC in .NET:** <https://learn.microsoft.com/en-us/aspnet/core/grpc/>

---

## Tips for Interview Preparation

- Practice writing code by hand (whiteboard-style) for common algorithm questions.
- Be able to explain the *why* behind design decisions (e.g., why choose Scoped vs Singleton).
- Review the official Microsoft documentation: <https://learn.microsoft.com/en-us/aspnet/core/>
- Follow the [.NET Blog](https://devblogs.microsoft.com/dotnet/) for the latest features and best practices.
- Understand the difference between .NET Framework, .NET Core, and .NET 5+ — this is a common opener.
- Know the middleware pipeline order and be ready to whiteboard it.
- Be prepared to discuss trade-offs (e.g., REST vs gRPC, controllers vs Minimal APIs, in-memory vs distributed cache).
