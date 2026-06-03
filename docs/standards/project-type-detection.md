# Standard: Project Type Detection

> Canonical reference. Every skill, subagent, and workflow MUST run this detection
> **first** so the same standards adapt to whatever .NET project is being analyzed.
> This framework is repository-agnostic: never assume a project layout — detect it.

## Goal

Given any .NET solution, classify each project into one of the supported types and
expose that classification to downstream standards (logging, configuration,
correlation IDs, etc.) which behave slightly differently per type.

## Supported project types

| Type | Primary signals |
|---|---|
| **ASP.NET Core Web API** | `.csproj` uses `Microsoft.NET.Sdk.Web`; `Program.cs`/`Startup.cs` calls `AddControllers`/`MapControllers` or Minimal API `MapGet/MapPost`; no Razor/Blazor markup |
| **Azure Functions (isolated/in-proc)** | `host.json` present; package `Microsoft.Azure.Functions.Worker` (isolated) or `Microsoft.NET.Sdk.Functions` (in-proc); `[Function]`/`[FunctionName]` attributes; `local.settings.json` |
| **Durable Functions** | Azure Functions signals **plus** package `Microsoft.Azure.Functions.Worker.Extensions.DurableTask` (or `Microsoft.Azure.WebJobs.Extensions.DurableTask`); `[OrchestrationTrigger]`, `[ActivityTrigger]`, `IDurableOrchestrationContext`/`TaskOrchestrationContext` |
| **ASP.NET MVC** | `Microsoft.NET.Sdk.Web`; `AddControllersWithViews`/`AddMvc`; `Views/` folder with `.cshtml`; `_ViewImports.cshtml` |
| **Blazor (Server / WASM / Web App)** | `Microsoft.NET.Sdk.Web` or `Microsoft.NET.Sdk.BlazorWebAssembly`; `App.razor`, `_Imports.razor`, `.razor` components; `AddServerSideBlazor`/`AddRazorComponents`/WASM `wwwroot/index.html` |
| **Worker Service** | `Microsoft.NET.Sdk.Worker` or `Host.CreateDefaultBuilder` with `AddHostedService<>`; classes implementing `BackgroundService`/`IHostedService`; no web SDK |
| **Console Application** | `Microsoft.NET.Sdk` with `<OutputType>Exe</OutputType>`; `static Main`/top-level statements; no web/worker/functions markers |

## Detection procedure

1. **Enumerate projects.** Read the `.sln` (or fall back to all `*.csproj`). Record
   each project's SDK attribute (`<Project Sdk="...">`), `<OutputType>`, and
   `<TargetFramework(s)>`.
2. **Inspect package references.** Collect `<PackageReference>` items (and, if
   central package management is used, resolve versions from `Directory.Packages.props`).
3. **Inspect host/config files.** Note presence of `host.json`, `local.settings.json`,
   `appsettings*.json`, `Program.cs`, `Startup.cs`, `App.razor`, `_Imports.razor`,
   `wwwroot/`, `Views/`.
4. **Scan entry points.** Identify `Main`, `CreateHostBuilder`/`CreateDefaultBuilder`,
   `WebApplication.CreateBuilder`, `[Function]`/`[FunctionName]`, `BackgroundService`.
5. **Classify** using the signal table above, preferring the most specific match
   (Durable Functions > Azure Functions; Blazor/MVC > generic Web API only when
   Razor/Blazor markup exists).
6. **Emit a detection summary** (used as the header of every report):

   ```
   Detected projects:
   - <ProjectName> → <Type>  (TFM: net8.0, isolated worker)
   ```

## How downstream standards consume this

- **Configuration:** Web/MVC/Blazor → `appsettings.json` + env vars + Azure App
  Configuration; Functions → `local.settings.json` + App Settings; Console/Worker →
  `appsettings.json` + env vars.
- **Correlation IDs:** Web API/MVC/Blazor/HTTP-triggered Functions → enforce
  `X-Correlation-Id` middleware/filter; Worker/Console → propagate via `Activity`/
  message headers instead of HTTP headers.
- **Logging:** all types use `ILogger<T>`; Functions use the Functions logging host;
  Application Insights wiring differs (SDK vs. `Microsoft.ApplicationInsights.WorkerService`).
- **Async:** entry-point shape differs (Minimal API handlers, Function methods,
  `ExecuteAsync`) — enforcement rules are identical.

## Edge cases

- **Multi-targeted projects:** report each TFM; apply standards to all.
- **Mixed solutions:** classify per-project; a single solution may contain Web API +
  Worker + Functions. Apply the matching variant to each.
- **Class libraries / test projects:** classify as `Library`/`Test` and exempt from
  host-specific rules (correlation middleware, startup config validation) but still
  subject to naming, async, exception, and logging standards.
