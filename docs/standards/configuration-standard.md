# Standard: Configuration Management

> Canonical configuration rules. Consumed by the `configuration-pattern-enforcer` skill.

## Principles

1. **Options Pattern everywhere.** Bind configuration sections to strongly-typed
   POCOs via `IOptions<T>` / `IOptionsSnapshot<T>` / `IOptionsMonitor<T>`. Register
   with `services.AddOptions<TOptions>().Bind(config.GetSection("Section"))`.
2. **No `IConfiguration` injection in business code.** `IConfiguration` may only be
   read during composition (startup). Application/domain services depend on typed
   options, never on `IConfiguration` or `config["Key"]`.
3. **Validate at startup.** Use `.ValidateDataAnnotations()` and/or
   `.Validate(...)` plus `.ValidateOnStart()` so misconfiguration fails fast at boot,
   not at first request. Add `[Required]`/`[Range]`/`[Url]` annotations on options.
4. **One options class per concern**, named `<Area>Options` (see
   [naming-standard](./naming-standard.md)), with a `public const string SectionName`.
5. **Layered sources** (later overrides earlier), resolved by host type via
   [detection](./project-type-detection.md):
   - **Web / MVC / Blazor / Worker / Console:** `appsettings.json` →
     `appsettings.{Environment}.json` → User Secrets (dev) → environment variables →
     command line → **Azure App Configuration** (+ **Key Vault** for secrets).
   - **Azure / Durable Functions:** `local.settings.json` (local only, not deployed) →
     Function App **Application Settings** (env vars) → App Configuration / Key Vault.
6. **Secrets never in source.** No connection strings, keys, or passwords in
   `appsettings.json`/`local.settings.json` committed to the repo. Use User Secrets
   locally and Key Vault / App Settings in deployed environments. Reference Key Vault
   via Managed Identity where possible.
7. **Azure App Configuration** for centralized, environment-specific, and
   feature-flagged settings when present; wire with sentinel-key refresh.

## Anti-patterns to flag

| Anti-pattern | Fix |
|---|---|
| `_config["ConnectionStrings:Db"]` in a service | inject `IOptions<DbOptions>` |
| `IConfiguration` injected into domain/app services | bind typed options at startup |
| no startup validation | add `.ValidateDataAnnotations().ValidateOnStart()` |
| secrets in `appsettings.json` | move to User Secrets / Key Vault |
| magic strings for section names | `const string SectionName` on the options class |
| `new HttpClient` with inline base URL from config string | typed options + `IHttpClientFactory` |

## Enforcement output

- Report file:line for each raw `IConfiguration`/indexer access in non-startup code,
  the proposed options class, and the binding + validation registration to add.
- List any secret found in committed config as **high severity**.
