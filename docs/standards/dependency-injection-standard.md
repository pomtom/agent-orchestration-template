# Standard: Dependency Injection

> Canonical DI rules. Consumed by the `dependency-injection-enforcer` skill.
> **Enforce/apply** companion to [architecture-standard §5](./architecture-standard.md)
> (which *assesses* DI). Keep the two consistent.

## Principles

1. **Inject abstractions through the constructor.** Services depend on interfaces passed in
   via the constructor; the container builds the graph. No `new`-ing of injectable
   collaborators across layers, and no `static` service singletons.
2. **No service location.** Do not inject `IServiceProvider` (or call
   `provider.GetService<T>()`, `ActivatorUtilities`, or a static `ServiceLocator`) to pull
   dependencies on demand. Declare them as constructor parameters. The narrow exceptions —
   factory registrations, `IServiceScopeFactory` for explicit child scopes in
   workers/consumers, and framework-required resolution — must be deliberate and localized.
3. **Correct lifetimes.**
   - **Transient** — lightweight, stateless, cheap to create.
   - **Scoped** — per request / per unit of work; `DbContext`, repositories, unit of work,
     and request-tied state ([repository-pattern-standard](./repository-pattern-standard.md)).
   - **Singleton** — stateless shared services, caches, `IOptionsMonitor<T>` consumers,
     clients that are themselves thread-safe.
4. **No captive dependencies.** A longer-lived service must never capture a shorter-lived
   one: a **singleton must not depend on a scoped/transient** service (the scoped instance
   is captured for the app's lifetime). Resolve scoped work inside a created scope via
   `IServiceScopeFactory`, or push the dependency to the call site.
5. **`DbContext` is scoped — never singleton.** `DbContext` is not thread-safe; a singleton
   (or a singleton capturing it) causes concurrency corruption. In workers/consumers that
   run as singletons, create a scope per message and resolve the context inside it.
6. **`HttpClient` via `IHttpClientFactory`.** Never `new HttpClient()` per call (socket
   exhaustion) and never hold a single `HttpClient` as a long-lived singleton (stale DNS).
   Register typed/named clients with `AddHttpClient<T>()`; the factory manages handler
   lifetimes.
7. **Group registrations by module.** Each module/feature exposes an
   `IServiceCollection AddXxx(this IServiceCollection, IConfiguration)` extension; `Program`
   composes them. No 300-line `Program.cs` registration wall; no registrations scattered
   across unrelated files.
8. **Register against the interface, with the right shape.** `AddScoped<IFoo, Foo>()`.
   Prefer `TryAdd*` in library/module extensions to avoid clobbering host overrides; use
   `TryAddEnumerable` for multi-implementation sets. Bind options via the Options pattern
   ([configuration-standard](./configuration-standard.md)), not by registering raw config.
9. **Validate the container in development.** Enable scope validation and build-on-startup
   validation so captive/missing dependencies fail fast at boot, not at first request.

## Recommended building blocks (introduce only what's missing)

- Per-module `Add<Module>()` extension methods over `IServiceCollection`.
- Typed `HttpClient` registrations via `AddHttpClient<TClient>()`.
- Scope-validation enabled (`ValidateScopes`/`ValidateOnBuild`) in the dev host build.

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| Singleton injecting a scoped service (`DbContext`, repository) | make the consumer scoped, or resolve per-scope via `IServiceScopeFactory` |
| `IServiceProvider`/`GetService<T>()` used as a service locator | declare the dependency in the constructor |
| `new HttpClient()` per call | `IHttpClientFactory` / typed client |
| `DbContext` registered/captured as singleton | register **scoped**; scope per message in workers |
| `new ConcreteService(...)` across a layer boundary | inject the abstraction |
| Wrong lifetime (stateful service as singleton; stateless re-created per call) | match lifetime to state/cost |
| Massive `Program.cs` registration block | per-module `AddXxx` extensions |
| Library `Add*` clobbering host registrations | use `TryAdd*` / `TryAddEnumerable` |

## Project-type notes (after [detection](./project-type-detection.md))

- **Web API / MVC / Blazor:** request scope is the natural scope; controllers/components
  resolve scoped services automatically.
- **Azure / Durable Functions (isolated):** register in the worker's `ConfigureServices`;
  a function invocation is the scope — don't capture per-invocation state in singletons.
- **Worker / Console:** the host runs the worker as a **singleton**; create an explicit
  scope (`IServiceScopeFactory.CreateScope()`) per message/iteration to resolve scoped
  services like `DbContext`/repositories.

## Enforcement output

- Report `file:line`, the anti-pattern (especially captive dependencies and service
  location), the fix, and severity (captive `DbContext`/service location = high).
- Apply mechanical, behavior-preserving fixes directly (constructor injection, lifetime
  corrections, module extraction, `IHttpClientFactory`); pause for approval where a public
  constructor signature changes.
- **Build gate:** rebuild after changes per
  [build-verification-standard](./build-verification-standard.md).
