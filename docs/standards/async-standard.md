# Standard: Async / Await

> Canonical async rules. Consumed by the `async-await-enforcer` skill.

## Core rules

1. **Async all the way.** If a method performs I/O (DB, HTTP, file, queue, Azure SDK),
   it must be `async` and return `Task`/`Task<T>`/`ValueTask<T>`. Callers await it.
2. **No sync-over-async.** Never use `.Result`, `.Wait()`, `.GetAwaiter().GetResult()`,
   or `Task.Run(...).Result`. These deadlock and waste threads.
3. **No blocking I/O on async paths.** Replace synchronous APIs with async equivalents
   (`ReadAsync`, `SendAsync`, `ToListAsync`, `SaveChangesAsync`, etc.).
4. **Propagate `CancellationToken`.** Accept a `CancellationToken` (default last
   parameter) and pass it through to every async call that accepts one.
5. **`ConfigureAwait(false)`** in library code (non-UI, non-request-context libraries).
   Application/host code (ASP.NET Core, Functions) does not need it because there is no
   captured sync context, but library projects should use it.
6. **Async suffix.** Async methods end in `Async` (except framework-mandated signatures
   like controller actions where the convention is relaxed but still encouraged).
7. **Avoid `async void`.** Only event handlers may be `async void`; everything else
   returns `Task`. `async void` exceptions cannot be caught and crash the process.
8. **Return `Task` directly** when no `await` is needed (pass-through), unless you need
   a `using`/try-catch scope — then `await` inside.
9. **Don't wrap CPU-bound sync work in fake async.** Use `Task.Run` only to offload
   genuinely CPU-bound work off a request thread, not to "make things async."

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| `var x = GetAsync().Result;` | `var x = await GetAsync();` |
| `DoAsync().Wait();` | `await DoAsync();` |
| `Task.Run(() => Async()).GetAwaiter().GetResult()` | `await Async();` |
| `async void Save()` | `async Task SaveAsync()` |
| Missing token | thread `CancellationToken` through |
| `Thread.Sleep` in async | `await Task.Delay(..., ct)` |
| `foreach` + `await` when parallel-safe & bounded | consider `Task.WhenAll` with care |

## Project-type notes (after [detection](./project-type-detection.md))

- **Web API / MVC / Blazor:** controller actions / handlers / component lifecycle
  (`OnInitializedAsync`) are async; no `.Result` in middleware or filters.
- **Azure / Durable Functions:** function methods are `async Task`; **inside Durable
  orchestrators do not** use `Task.Delay`/`DateTime.Now`/non-deterministic async —
  use `context.CreateTimer`/durable-safe APIs.
- **Worker Services:** `ExecuteAsync(CancellationToken)` honors the token and awaits.

## Enforcement output

- Report each violation as: file:line, the offending pattern, the recommended fix, and
  severity (error for deadlock-risk `.Result`/`.Wait()`, warning for missing token).
- Offer to apply mechanical fixes; flag any change that alters a public signature.
