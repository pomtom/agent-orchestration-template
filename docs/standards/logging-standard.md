# Standard: Logging

> Canonical logging rules. Consumed by the `logging-enforcer` skill.

## Principles

1. **Use `ILogger<T>`.** Inject the generic logger; the category is the type. Never
   use `Console.WriteLine`/`Debug.WriteLine`/`Trace` for application logging.
2. **Structured logging only.** Use message templates with named placeholders, not
   string interpolation/concatenation:
   - ✅ `_logger.LogInformation("Order {OrderId} placed for {CustomerId}", id, customerId);`
   - ❌ `_logger.LogInformation($"Order {id} placed for {customerId}");`
   Interpolation destroys structured properties and hurts performance.
3. **Correct levels.**
   - `Trace`/`Debug`: developer diagnostics (off in prod).
   - `Information`: business/lifecycle events (started, processed, completed).
   - `Warning`: recoverable/abnormal but handled (retry, fallback, validation reject).
   - `Error`: a failure of the current operation (exception handled at boundary).
   - `Critical`: app/host-wide failure.
4. **Log meaningful business events**, application **lifecycle** events (startup,
   shutdown, config loaded, dependency health), and all **failures/exceptions** at the
   centralized boundary (see [exception-handling-standard](./exception-handling-standard.md)).
5. **Always include `CorrelationId` and `InstanceId` in a logging scope** so every log
   line for a request is both correlated *and* attributable to the host/replica that
   produced it (see [correlation-id-standard](./correlation-id-standard.md)). Both flow
   into Application Insights as custom dimensions, enabling you to query a request
   end-to-end **and** see which instance handled each hop:

   ```csharp
   using (_logger.BeginScope(new Dictionary<string, object>
   {
       ["CorrelationId"] = correlationId,
       ["InstanceId"]    = instanceId,
   }))
   {
       // all logs inside carry both dimensions
   }
   ```

6. **`InstanceId` identifies the running host/replica**, not the request. Resolve it
   **once at startup** and reuse it for the process lifetime. Source it by detected
   project type ([detection](./project-type-detection.md)):
   - **Azure App Service / Functions (Web API, MVC, Blazor, Azure & Durable Functions):**
     the `WEBSITE_INSTANCE_ID` environment variable (Azure sets it per instance); fall
     back to `Environment.MachineName`.
   - **Containers / AKS:** `HOSTNAME` (pod name) or `Environment.MachineName`.
   - **Worker / Console / local:** `Environment.MachineName`; if multiple processes run
     on one machine, append a per-process suffix (a short startup GUID).
   - **Always have a fallback:** if no host value is available, generate a process-scoped
     GUID at startup so the dimension is never empty.
   Register it as a singleton (e.g. an `IInstanceContext`/`InstanceInfo` holding the
   resolved id) and read it wherever the scope is opened. For App Insights you may also
   set it globally via an `ITelemetryInitializer` so it tags **all** telemetry
   (requests, dependencies, exceptions), not only `ILogger` traces.
7. **Pass the exception object**: `_logger.LogError(ex, "Failed to process {OrderId}", id);`
   — never `LogError(ex.Message)`.
8. **No sensitive data.** Never log secrets, tokens, passwords, full PII, or full
   payloads containing them. Redact/hash where identifiers are needed.
9. **Application Insights** for telemetry when Azure is in play:
   - Web/Worker/Console: `AddApplicationInsightsTelemetry()` /
     `AddApplicationInsightsTelemetryWorkerService()`.
   - Azure / Durable Functions: App Insights via the Functions host
     (`APPLICATIONINSIGHTS_CONNECTION_STRING`); use `ILogger` so telemetry flows
     automatically; preserve operation/correlation linkage for distributed tracing.
   - Surface `CorrelationId` and `InstanceId` as custom dimensions (via the logging
     scope and/or an `ITelemetryInitializer`) so they are queryable in App Insights, e.g.
     `traces | where customDimensions.CorrelationId == "..."` and
     `... | summarize count() by tostring(customDimensions.InstanceId)`.
10. **Consistent format across projects.** Same scope keys —
    `CorrelationId`, `InstanceId`, plus `UserId` (where applicable) and `Operation` —
    same level semantics, same enrichment.
11. **High-frequency log call sites** should use `LoggerMessage` source-generated
    delegates (`[LoggerMessage]`) to avoid allocations on hot paths.

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| `Console.WriteLine(...)` | `ILogger<T>` with appropriate level |
| `LogInformation($"...{x}...")` | message template with `{X}` placeholder |
| `LogError(ex.Message)` | `LogError(ex, "...", args)` |
| logging secrets/PII | redact / remove |
| no correlation scope | wrap operation in `BeginScope` with `CorrelationId` **and** `InstanceId` |
| `InstanceId` missing from scope/telemetry | resolve once at startup; add to scope and/or `ITelemetryInitializer` |
| `InstanceId` resolved per-request/per-log | resolve once at startup, reuse for process lifetime |
| swallowed exception with no log | log at boundary |

## Enforcement output

- Report file:line, anti-pattern, fix, severity.
- Note missing App Insights wiring for detected Azure project types and any
  request/operation entry point whose scope is missing `CorrelationId` or `InstanceId`.
- Confirm `InstanceId` is resolved once at startup (singleton/initializer) with a
  non-empty fallback, and recommend the host-appropriate source
  (`WEBSITE_INSTANCE_ID` / `HOSTNAME` / `Environment.MachineName` + GUID fallback).
