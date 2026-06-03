---
name: logging-enforcer
description: Enforce structured logging in .NET with ILogger<T> — message templates (not interpolation), correct log levels, correlation-ID scopes, business/lifecycle/failure events, Application Insights integration, and no sensitive data. Use when the user asks to review or fix logging, add structured logging, or integrate Application Insights.
---

# Logging Enforcer

Drive logging toward structured, leveled, correlated, and secret-safe.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   to choose the right Application Insights wiring (SDK vs. WorkerService vs. Functions
   host).
2. **Audit** against
   [logging-standard](../../../docs/standards/logging-standard.md):
   `Console.WriteLine`/`Trace` for app logging, string interpolation in log templates,
   `LogError(ex.Message)` instead of passing the exception, wrong levels, missing
   scope keys (`CorrelationId` **and** `InstanceId`), `InstanceId` not resolved once at
   startup or with no fallback, logged secrets/PII, missing App Insights, and hot-path
   log calls that should use `[LoggerMessage]`.
3. **Report** `file:line`, anti-pattern, fix, severity.
4. **Propose**: a logging scope around request/operation entry points carrying both
   `CorrelationId` (see
   [correlation-id-standard](../../../docs/standards/correlation-id-standard.md)) and
   `InstanceId` (host/replica id resolved once at startup —
   `WEBSITE_INSTANCE_ID`/`HOSTNAME`/`Environment.MachineName` with a GUID fallback);
   App Insights registration for detected Azure types, with both surfaced as custom
   dimensions via the scope and/or an `ITelemetryInitializer`; and source-generated
   logging for hot paths.
5. **Offer to apply** mechanical fixes (interpolation→templates, add exception arg).

## Guardrails

- Never introduce logging of secrets/PII; redact identifiers where needed.
- Keep level semantics and scope keys consistent across all projects.
- **Build gate:** after applying changes, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
