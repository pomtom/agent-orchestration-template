# GitHub Copilot Instructions

These instructions make GitHub Copilot follow the **same** engineering standards as the
Claude Code framework in this repository. The canonical standards live under
[`docs/standards/`](../docs/standards/) and are the single source of truth — when a rule
here is ambiguous, defer to the matching `docs/standards/*.md` file.

> **Scope:** This repository hosts a repository-level automation framework. Do **not**
> scaffold new .NET projects, Web APIs, Function Apps, Web Apps, or sample/business code.
> Generate business code only when explicitly asked. (Creating a missing **test** project
> is permitted.)

## Always detect the project type first

Before suggesting changes, identify the project type (ASP.NET Core Web API, Azure
Functions, Durable Functions, MVC, Blazor, Worker Service, Console) using
[`project-type-detection`](../docs/standards/project-type-detection.md). Tailor
suggestions to the detected type.

## Standards Copilot must follow

- **Async/await** — async all the way; never `.Result`, `.Wait()`,
  `.GetAwaiter().GetResult()`, or `async void` (except event handlers); flow
  `CancellationToken`; `ConfigureAwait(false)` in libraries.
  ([async-standard](../docs/standards/async-standard.md))
- **Naming** — Microsoft/.NET conventions; `I`-prefixed interfaces; `Async` suffix;
  `Service`/`Repository`/`Dto`/`Request`/`Response`/`Command`/`Query`/`Handler`/`Event`/
  `Options` patterns. ([naming-standard](../docs/standards/naming-standard.md))
- **Exception handling** — centralized middleware; meaningful domain exceptions; no empty
  catches; `throw;` not `throw ex;`; standardized `ProblemDetails` with a correlation ID.
  ([exception-handling-standard](../docs/standards/exception-handling-standard.md))
- **Configuration** — Options Pattern with validated, strongly-typed classes; no raw
  `IConfiguration` in business code; secrets in User Secrets/Key Vault, never in source.
  ([configuration-standard](../docs/standards/configuration-standard.md))
- **Logging** — structured `ILogger<T>` with message templates (no interpolation);
  correct levels; a logging scope carrying both `CorrelationId` and `InstanceId`
  (host/replica id resolved once at startup, surfaced as App Insights custom
  dimensions); pass the exception object; no secrets/PII; Application Insights where
  Azure is used. ([logging-standard](../docs/standards/logging-standard.md))
- **Correlation IDs** — `X-Correlation-Id` read/generated at every HTTP/message entry
  point, logged, attached to `Activity`, propagated outbound, and returned in errors.
  ([correlation-id-standard](../docs/standards/correlation-id-standard.md))
- **Packages** — Central Package Management (`Directory.Packages.props`) +
  `Directory.Build.props`; pinned versions; no vulnerable/deprecated packages.
  ([package-governance-standard](../docs/standards/package-governance-standard.md))
- **Testing** — match the existing framework; Arrange-Act-Assert;
  `Method_Scenario_ExpectedOutcome`; mock external dependencies; cover success/failure/
  edge cases. ([testing-standard](../docs/standards/testing-standard.md))
- **Architecture** — Clean Architecture layering, SOLID, CQRS/Repository/DI done right.
  ([architecture-standard](../docs/standards/architecture-standard.md))

## When generating code

- Prefer async, validated, observable, and correlated implementations by default.
- Never hardcode secrets or connection strings.
- Keep suggestions consistent with the patterns already present in the file/solution.
