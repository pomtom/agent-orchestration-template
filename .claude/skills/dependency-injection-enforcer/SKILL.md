---
name: dependency-injection-enforcer
description: Enforce correct .NET dependency injection — constructor injection over service location, correct service lifetimes (transient/scoped/singleton), no captive dependencies (singleton capturing scoped DbContext), IHttpClientFactory instead of new HttpClient, per-module registration extensions, TryAdd in libraries, and container validation at startup. Use when the user asks to review or fix DI, fix lifetime bugs/captive dependencies, remove IServiceProvider service location, or organize service registrations.
---

# Dependency Injection Enforcer

Drive the service graph toward constructor-injected abstractions with correct lifetimes and
no captive dependencies — the apply/fix companion to the architecture review's DI section.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   — the natural scope differs (request scope for web; per-invocation for Functions;
   explicit `IServiceScopeFactory` scopes for Worker/Console singletons).
2. **Audit** against
   [dependency-injection-standard](../../../docs/standards/dependency-injection-standard.md):
   **captive dependencies** (singleton capturing scoped/transient, especially `DbContext`),
   `IServiceProvider`/`GetService<T>()` service location, `new HttpClient()`, `DbContext`
   registered or captured as singleton, `new`-ing injectable services across a layer
   boundary, wrong lifetimes, monolithic `Program.cs` registration walls, and library
   `Add*` clobbering host registrations (should use `TryAdd*`).
3. **Report** `file:line`, the anti-pattern, the fix, and severity — captive `DbContext`
   and service location are **high**.
4. **Apply** mechanical, behavior-preserving fixes directly: convert service location to
   constructor injection, correct lifetimes, swap `new HttpClient()` for typed
   `IHttpClientFactory` clients, extract per-module `AddXxx` extensions, and enable scope
   validation in the dev host. Pause for approval where a **public constructor signature**
   changes.

## Guardrails

- Never make `DbContext` (or anything capturing it) a singleton — it is not thread-safe;
  in Worker/Console singletons create a scope per message via `IServiceScopeFactory`.
- Bind configuration via the Options pattern
  ([configuration-standard](../../../docs/standards/configuration-standard.md)), not raw
  config registrations.
- **Build gate:** after applying changes, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
