---
name: configuration-pattern-enforcer
description: Enforce the .NET Options Pattern with strongly-typed, validated configuration — remove raw IConfiguration access from business code, add startup validation, and wire appsettings.json / local.settings.json / environment variables / Azure App Configuration correctly. Use when the user asks to review or fix configuration, adopt the options pattern, or stop using IConfiguration directly.
---

# Configuration Pattern Enforcer

Move configuration to strongly-typed, validated options bound at startup.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   to know the correct configuration sources (Web/Worker/Console vs. Functions).
2. **Audit** against
   [configuration-standard](../../../docs/standards/configuration-standard.md): raw
   `IConfiguration`/indexer access in non-startup code, `IConfiguration` injected into
   domain/app services, missing startup validation, missing `SectionName` constants,
   secrets committed in `appsettings.json`/`local.settings.json`, and missing layered
   sources / App Configuration wiring.
3. **Report** `file:line`, the issue, the proposed `<Area>Options` class, and the
   `AddOptions<T>().Bind(...).ValidateDataAnnotations().ValidateOnStart()` registration.
4. **Flag committed secrets as high severity** and recommend User Secrets / Key Vault.
5. **Offer to apply**: create options classes, replace raw accesses with `IOptions<T>`,
   and add validation registrations.

## Guardrails

- Never move secret values into source; reference Key Vault / App Settings instead.
- Do not change runtime configuration values — only the access pattern and validation.
- **Build gate:** after applying changes, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
