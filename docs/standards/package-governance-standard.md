# Standard: Dependency & Package Governance

> Canonical package-governance rules. Consumed by the `package-governance` skill and
> the `docs/governance/package-governance.md` policy.

## Principles

1. **Central Package Management (CPM).** Use `Directory.Packages.props` at the repo
   root with `<ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>`
   and `<PackageVersion>` entries. Individual `.csproj` files reference packages
   **without** versions.
2. **Shared MSBuild settings.** Use `Directory.Build.props` (and optionally
   `Directory.Build.targets`) for solution-wide properties: `TargetFramework`/
   `LangVersion`, `Nullable=enable`, `ImplicitUsings=enable`,
   `TreatWarningsAsErrors`, `EnableNETAnalyzers`, `AnalysisLevel`,
   deterministic build, and source-link settings.
3. **One version per package** across the solution (CPM enforces this). Resolve
   conflicting transitive versions explicitly.
4. **No floating versions** (`*`, `1.2.*`) in committed manifests — pin exact versions
   for reproducible builds.
5. **Vulnerability scanning.** No known-vulnerable packages. Check with
   `dotnet list package --vulnerable --include-transitive`. Treat moderate+ as blocking.
6. **Outdated detection.** Periodically run
   `dotnet list package --outdated --include-transitive`; schedule upgrades, prioritizing
   security and LTS alignment.
7. **Remove unused dependencies.** Detect package references not used by any code and
   propose removal.
8. **Deprecated packages.** `dotnet list package --deprecated` → replace.
9. **License/source hygiene.** Prefer well-maintained, appropriately-licensed packages;
   avoid abandoned libraries. Restore only from trusted feeds (`nuget.config`).

## Migration to CPM (when not yet adopted)

1. Add root `Directory.Packages.props` with all distinct `PackageReference` versions
   hoisted to `<PackageVersion>`.
2. Strip `Version=` attributes from every `.csproj`.
3. Add/normalize `Directory.Build.props` with shared properties.
4. Build to confirm parity; resolve any version conflicts surfaced by CPM.

## Enforcement / report output

- Inventory: package, current version, latest, vulnerable?, deprecated?, used?,
  projects referencing it.
- Findings grouped: **Vulnerable (blocking)**, **Deprecated**, **Outdated**,
  **Unused**, **Not centralized**.
- Concrete remediation: exact `Directory.Packages.props`/`.csproj` edits and CLI
  commands. Flag major-version upgrades as potentially breaking.
