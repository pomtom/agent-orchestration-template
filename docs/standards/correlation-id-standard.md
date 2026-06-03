# Standard: Correlation ID

> Canonical correlation-ID rules for end-to-end tracing. Consumed by the
> `correlation-id-enforcer` skill.

## Principles

1. **Header name:** `X-Correlation-Id` (configurable constant in one place).
2. **Require / generate.** For every inbound request to a Web API, MVC app, Blazor
   endpoint, Azure Function **HTTP trigger**, or HTTP-triggered Durable starter:
   - If `X-Correlation-Id` is present and valid, use it.
   - If missing, **generate** a new GUID (`Guid.NewGuid().ToString("N")`).
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

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| No correlation handling on an HTTP entry point | add middleware/worker-middleware |
| ID read but not logged | wrap operation in logging scope |
| ID not propagated to outbound HTTP/messages | add delegating handler / message stamp |
| Each layer generating its own ID | generate once at the edge, propagate |
| ID absent from error responses | add to ProblemDetails extension |

## Enforcement output

- Report each HTTP/message entry point lacking correlation handling, the propagation
  gaps (outbound HTTP/messaging), and the building block to add for the detected type.
