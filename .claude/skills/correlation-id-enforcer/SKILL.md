---
name: correlation-id-enforcer
description: Enforce end-to-end request tracing via X-Correlation-Id across .NET Web APIs, Azure Functions, and HTTP triggers — read or generate the header, put it in logging scope and Activity, propagate it to outbound HTTP and messaging, and include it in error responses and Application Insights. Use when the user asks to add correlation IDs, enable distributed tracing, or propagate request context.
---

# Correlation ID Enforcer

Establish a single correlation ID per request and propagate it everywhere.

**Inbound contract first (non-negotiable):** every Web API / MVC / Blazor endpoint and
every Azure Function **HTTP trigger** (and non-HTTP triggers via message property) MUST
**read** the `X-Correlation-Id` request header, **use it when present and valid**, and
**generate `Guid.NewGuid().ToString("N")` when it is missing/invalid** — then echo it on
the response. No request may proceed without a correlation ID. Validate the incoming value
(trim, ≤128 chars, `[A-Za-z0-9._-]`) before using it; otherwise fall back to a generated GUID.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   to choose middleware vs. Functions worker-middleware vs. worker/message handling.
2. **Audit** against
   [correlation-id-standard](../../../docs/standards/correlation-id-standard.md), starting
   with the **inbound contract**: any HTTP/message entry point that does not read the header
   or does not generate-and-assign when it is absent (**highest priority** — the request
   would run with no ID). Then: IDs read but not logged or not attached to `Activity`, IDs
   not propagated to outbound HTTP / messaging, multiple IDs generated per flow, raw
   unsanitized client values echoed into logs/responses, and IDs missing from error
   responses/telemetry.
3. **Report** each entry point and propagation gap, with the building block to add for
   the detected host type.
4. **Apply the building blocks** directly (do not stop at suggestions): `CorrelationIdMiddleware` or
   `IFunctionsWorkerMiddleware`, a `CorrelationIdDelegatingHandler` on
   `IHttpClientFactory` clients, message-property stamping for queues/Service Bus, the
   logging scope (see [logging-standard](../../../docs/standards/logging-standard.md)),
   and the ProblemDetails extension (see
   [exception-handling-standard](../../../docs/standards/exception-handling-standard.md)).

## Guardrails

- Generate the ID **once** at the edge; downstream layers only read/propagate it.
- Use a single configurable header-name constant; default `X-Correlation-Id`.
- **Build gate:** after applying changes, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
