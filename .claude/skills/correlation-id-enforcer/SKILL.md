---
name: correlation-id-enforcer
description: Enforce end-to-end request tracing via X-Correlation-Id across .NET Web APIs, Azure Functions, and HTTP triggers — read or generate the header, put it in logging scope and Activity, propagate it to outbound HTTP and messaging, and include it in error responses and Application Insights. Use when the user asks to add correlation IDs, enable distributed tracing, or propagate request context.
---

# Correlation ID Enforcer

Establish a single correlation ID per request and propagate it everywhere.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   to choose middleware vs. Functions worker-middleware vs. worker/message handling.
2. **Audit** against
   [correlation-id-standard](../../../docs/standards/correlation-id-standard.md):
   HTTP/message entry points without correlation handling, IDs read but not logged or
   not attached to `Activity`, IDs not propagated to outbound HTTP / messaging, multiple
   IDs generated per flow, and IDs missing from error responses/telemetry.
3. **Report** each entry point and propagation gap, with the building block to add for
   the detected host type.
4. **Propose / apply building blocks**: `CorrelationIdMiddleware` or
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
