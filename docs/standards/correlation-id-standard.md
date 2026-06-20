# Standard: Correlation ID

> Canonical correlation-ID rules for end-to-end tracing. Consumed by the
> `correlation-id-enforcer` skill.

## Inbound contract (MANDATORY — every HTTP entry point)

**Every** ASP.NET Core Web API, MVC app, Blazor endpoint, Azure Function **HTTP
trigger**, and HTTP-triggered Durable starter **MUST accept** a correlation ID on the
request header and **MUST guarantee** one exists for the rest of the request:

1. **Read** the `X-Correlation-Id` request header.
2. **If present and valid → use the caller's value** (so the trace spans the caller too).
3. **If absent, empty, or invalid → generate `Guid.NewGuid().ToString("N")` and assign
   it.** No request ever proceeds without a correlation ID — this is non-negotiable.
4. **Echo it back** on the response header so the caller can record the value used.

> **"Valid"** = non-empty after trim, within a sane length cap (≤128 chars), and limited
> to a safe character set (`[A-Za-z0-9._-]`). Reject/replace anything else to prevent log
> forging and header-injection — fall back to a generated GUID. Never echo an
> unsanitized client value straight into logs or the response.

This rule applies identically to **non-HTTP** entry points (queue / Service Bus / Event
Hub / Timer triggers, worker message loops): read the ID from the message property; if
absent or invalid, generate one. The transport differs; the *accept-or-generate
guarantee* does not.

## Principles

1. **Header name:** `X-Correlation-Id` (one configurable constant — never a magic string
   scattered across the code).
2. **Accept-or-generate at the edge** per the inbound contract above — generate **once**,
   at the first entry point; downstream layers only read and propagate it.
3. **Echo it back** on the response (`X-Correlation-Id`) so callers can record it.
4. **Put it in scope immediately** so all logs for the request carry it
   (`ILogger.BeginScope` with `CorrelationId`) — see
   [logging-standard](./logging-standard.md).
5. **Attach to `Activity`.** Set it as a baggage/tag on `Activity.Current` so
   distributed tracing (W3C `traceparent`) and Application Insights correlate.
6. **Propagate outbound.** Add the header to every outbound HTTP call (via a
   `DelegatingHandler` registered on `IHttpClientFactory` clients) and to every
   message published to queues/topics/Service Bus/Event Hubs (as a message property).
7. **Flow across async boundaries.** For Functions/queue/Service Bus triggers, read the
   correlation ID from the message property; if absent, generate one. Durable
   orchestrations carry it through `input`/custom status so all activities share it.
8. **Include in error responses.** Every `ProblemDetails`/error payload exposes the
   correlation ID (see [exception-handling-standard](./exception-handling-standard.md)).
9. **Include in telemetry.** Correlation ID becomes a custom dimension on App Insights
   telemetry for end-to-end query.

## Implementation building blocks (by host type)

- **Web API / MVC / Blazor Server:** `CorrelationIdMiddleware` early in the pipeline
  (before routing) that reads/creates the ID, stores it in `HttpContext.Items` and a
  scoped `ICorrelationContext`, sets the logging scope, the `Activity` tag, and the
  response header.
- **Azure / Durable Functions (isolated):** an `IFunctionsWorkerMiddleware` that does
  the same from the trigger metadata (HTTP headers or message properties).
- **Outbound HTTP:** `CorrelationIdDelegatingHandler` added to all typed/named clients.
- **Messaging:** helper that stamps/reads the property on publish/consume.
- **Worker / Console:** create an ID per unit of work / message and set the scope.

## Reference implementation (accept-or-generate)

The exact types/namespaces adapt to the repo; the **logic must match**: read header →
validate → use or generate → expose to the request → set logging scope + `Activity` →
echo on response.

**Shared constant + validation (one place):**

```csharp
public static class CorrelationConstants
{
    public const string HeaderName = "X-Correlation-Id";

    /// Returns the caller's value when safe; otherwise a fresh GUID. Never returns null/empty.
    public static string Normalize(string? incoming)
    {
        var value = incoming?.Trim();
        var isValid = !string.IsNullOrEmpty(value)
            && value.Length <= 128
            && value.All(c => char.IsLetterOrDigit(c) || c is '.' or '_' or '-');
        return isValid ? value! : Guid.NewGuid().ToString("N");
    }
}
```

**ASP.NET Core Web API / MVC / Blazor Server — middleware (register early, before routing):**

```csharp
public sealed class CorrelationIdMiddleware(RequestDelegate next, ILogger<CorrelationIdMiddleware> logger)
{
    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = CorrelationConstants.Normalize(
            context.Request.Headers[CorrelationConstants.HeaderName]);

        context.Items[CorrelationConstants.HeaderName] = correlationId; // available to the request
        Activity.Current?.SetTag("correlation.id", correlationId);       // distributed tracing / App Insights

        // Echo back even if the pipeline later throws.
        context.Response.OnStarting(() =>
        {
            context.Response.Headers[CorrelationConstants.HeaderName] = correlationId;
            return Task.CompletedTask;
        });

        using (logger.BeginScope(new Dictionary<string, object> { ["CorrelationId"] = correlationId }))
        {
            await next(context); // every log in the request now carries CorrelationId
        }
    }
}

// Program.cs — as early as possible:
app.UseMiddleware<CorrelationIdMiddleware>();
```

**Azure Functions (isolated worker) — HTTP/message trigger middleware:**

```csharp
public sealed class CorrelationIdMiddleware : IFunctionsWorkerMiddleware
{
    public async Task Invoke(FunctionContext context, FunctionExecutionDelegate next)
    {
        string? incoming = null;
        if (await context.GetHttpRequestDataAsync() is { } req &&
            req.Headers.TryGetValues(CorrelationConstants.HeaderName, out var values))
        {
            incoming = values.FirstOrDefault();
        }
        // For queue/Service Bus triggers, read from context.BindingContext.BindingData instead.

        var correlationId = CorrelationConstants.Normalize(incoming);
        context.Items[CorrelationConstants.HeaderName] = correlationId;
        Activity.Current?.SetTag("correlation.id", correlationId);

        await next(context);
        // Echo onto the HttpResponseData in the function or a response-writing step.
    }
}

// Program.cs (isolated host):
// .ConfigureFunctionsWorkerDefaults(b => b.UseMiddleware<CorrelationIdMiddleware>())
```

**Outbound propagation — delegating handler on every typed/named `HttpClient`:**

```csharp
public sealed class CorrelationIdDelegatingHandler(IHttpContextAccessor accessor) : DelegatingHandler
{
    protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken ct)
    {
        var id = accessor.HttpContext?.Items[CorrelationConstants.HeaderName] as string;
        if (!string.IsNullOrEmpty(id) && !request.Headers.Contains(CorrelationConstants.HeaderName))
            request.Headers.Add(CorrelationConstants.HeaderName, id);
        return base.SendAsync(request, ct);
    }
}
// services.AddHttpClient<TClient>().AddHttpMessageHandler<CorrelationIdDelegatingHandler>();
```

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| HTTP entry point that does **not read** `X-Correlation-Id` from the request | add the accept-or-generate middleware/worker-middleware |
| Header missing and **no GUID generated** (request runs with no ID) | generate `Guid.NewGuid().ToString("N")` and assign |
| Echoing the **raw client header value** unsanitized into logs/response | normalize/validate first (length + charset cap) |
| No correlation handling on an HTTP entry point | add middleware/worker-middleware |
| ID read but not logged | wrap operation in logging scope |
| ID not propagated to outbound HTTP/messages | add delegating handler / message stamp |
| Each layer generating its own ID | generate once at the edge, propagate |
| ID absent from error responses | add to ProblemDetails extension |

## Enforcement output

- Report each HTTP/message entry point lacking correlation handling, the propagation
  gaps (outbound HTTP/messaging), and the building block to add for the detected type.
