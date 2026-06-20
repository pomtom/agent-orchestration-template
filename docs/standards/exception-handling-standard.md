# Standard: Exception Handling

> Canonical exception-handling rules. Consumed by the `exception-handling-enforcer` skill.

## Principles

1. **Centralize.** Use a single global handler per host; do not scatter try/catch for
   cross-cutting concerns.
   - **Web API / MVC / Blazor Server:** exception-handling middleware
     (`UseExceptionHandler` with a handler, or a custom
     `ExceptionHandlingMiddleware`) producing **ProblemDetails** (RFC 7807).
   - **Azure / Durable Functions (isolated):** a function **middleware**
     (`IFunctionsWorkerMiddleware`) that catches, logs, and shapes the response;
     for in-proc, a try/catch wrapper or filter.
   - **Worker / Console:** catch at the top of the unit of work / message loop; let
     truly fatal errors stop the host.
2. **Throw specific, meaningful exceptions.** Prefer domain exceptions
   (`OrderNotFoundException`, `InsufficientInventoryException`) over `Exception`,
   `ApplicationException`, or `InvalidOperationException` for domain conditions.
3. **Never swallow.** No empty `catch {}`. If you catch, you must handle, rethrow
   (`throw;` — never `throw ex;`), or wrap with an inner exception.
4. **Catch narrowly.** Avoid `catch (Exception)` except at the centralized boundary.
5. **Standardize API error responses.** All error responses use `ProblemDetails`
   with: `type`, `title`, `status`, `detail`, `instance`, and a `correlationId`
   extension. Map domain exceptions → HTTP status (validation→400, not-found→404,
   conflict→409, unauthorized→401/403, unexpected→500).
6. **Log every handled exception** at the boundary with structured context (see
   [logging-standard](./logging-standard.md)), including the correlation ID (see
   [correlation-id-standard](./correlation-id-standard.md)). Do not double-log the same
   exception at every layer.
7. **Do not leak internals.** Return generic messages + correlation ID to clients;
   keep stack traces and sensitive detail in logs only. Never include secrets/PII in
   messages.
8. **Validation vs. exceptions.** Prefer result/validation objects for expected
   business rule failures on hot paths; reserve exceptions for exceptional conditions.

## Recommended building blocks

- A small domain exception hierarchy (e.g., `DomainException` base →
  `NotFoundException`, `ValidationException`, `ConflictException`).
- An `ExceptionToProblemDetailsMapper` mapping exception types → status + title.
- Correlation ID surfaced in every error payload and log scope.

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| `catch (Exception) {}` | remove or handle/rethrow with logging |
| `throw ex;` | `throw;` (preserve stack) |
| `catch { throw new Exception("failed"); }` | wrap with inner: `throw new XException(msg, ex)` |
| returning 200 on error | map to correct status via ProblemDetails |
| try/catch in every method | move to centralized boundary |
| exception messages with PII/secrets | sanitize; log detail separately |

## Where exceptions get silently missed (escape hatches)

A centralized handler only catches what actually reaches it. These constructs let an
exception **bypass the boundary and disappear** — the failure mode behind "an exception
got swallowed/missed." Hunt for each:

| Escape hatch | Why it's missed | Fix |
|---|---|---|
| `async void` (non event-handler) | the exception is raised on the sync context with no `Task` to observe → **crashes the process / is lost**, never reaches the handler | return `async Task`; only event handlers may be `async void`, and they must `try/catch` internally |
| Fire-and-forget `Task` (`_ = DoAsync();`, unawaited call) | a faulted `Task` nobody awaits is observed by no one; the exception is dropped | `await` it, or run it through a supervised runner that logs faults (e.g. `ContinueWith` logging on `OnlyOnFaulted`, or a tracked background job) |
| `Task.WhenAll(...)` with a single `await` | `await` rethrows only the **first** exception; the rest are hidden on the aggregate | inspect `task.Exception` / iterate the tasks, or use the `WhenAll` result and log every faulted task's exception |
| `AggregateException` not unwrapped | logging `ex.Message` on an aggregate shows "One or more errors occurred" and hides the real cause(s) | unwrap with `ex.Flatten().InnerExceptions` (or `ex.InnerException`) and log each |
| `BackgroundService.ExecuteAsync` throwing | behavior depends on TFM: **pre-.NET 6** swallows it silently (the host keeps running with a dead worker); **.NET 6+** defaults to `BackgroundServiceExceptionBehavior.StopHost` (the whole host crashes) — which is sometimes downgraded back to `Ignore`. Either way the *common real bug* is a `try/catch` **inside** the loop that swallows per-iteration errors with no log | wrap the loop body in try/catch that **logs** and decides stop-vs-continue per iteration; leave the default `StopHost` for genuinely fatal startup/loop failures so they're loud, not silent |
| `catch { }` / `catch (Exception) { /* log nothing */ }` | the exception is consumed with no record | remove, or log + handle/rethrow; never an empty body |
| Exception thrown inside `finally` / `Dispose`/`DisposeAsync` | it **replaces/masks** the original in-flight exception | don't throw from `finally`/`Dispose`; guard cleanup with its own try/catch that logs |
| `OperationCanceledException`/`TaskCanceledException` caught by a broad `catch (Exception)` | cancellation gets logged as an error or swallowed as a "failure" | catch and **rethrow** `OperationCanceledException` when `ct.IsCancellationRequested` (expected shutdown), separate from real errors |
| Swallowing in event handlers / `Timer`/`ThreadPool` callbacks | no `Task`/await boundary observes them | wrap the callback body in try/catch that logs |
| `Task` discarded in LINQ (`.Select(async x => …)` not awaited) | each produced `Task` is unobserved | materialize and `await Task.WhenAll(...)`, then check every result |
| Return-code/`bool` swallowing of a thrown failure | the real exception is converted to a silent `false` | let domain exceptions propagate to the boundary, or return a result that carries the failure |

**Rule of thumb:** every `Task` is either `await`ed, returned, or explicitly supervised
with fault logging — never abandoned. Every `catch` either handles meaningfully, rethrows
with `throw;`, or wraps with an inner exception — never silently. Cancellation is not a
failure; distinguish it.

## Enforcement output

- Report file:line, the anti-pattern, the fix, and severity. Treat every **escape hatch
  above as high severity** — these are the paths where exceptions are missed entirely.
- Note where a centralized handler is missing for a detected host type and propose
  the appropriate middleware/wrapper.
- Verify the centralized handler is actually reached: trace that unhandled exceptions on
  each entry point (controller, Function trigger, worker loop) propagate to it rather than
  being absorbed by one of the escape hatches above.
