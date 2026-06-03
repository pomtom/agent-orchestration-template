---
name: package-governance
description: Govern NuGet dependencies for a .NET solution — adopt Central Package Management (Directory.Packages.props) and Directory.Build.props, detect vulnerable/deprecated/outdated/unused packages, and recommend upgrades and removals. Use when the user asks to centralize package management, audit dependencies, check for vulnerable packages, or clean up NuGet references.
---

# Package Governance

Centralize and harden dependency management across the solution.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   and enumerate all `.csproj` package references.
2. **Assess CPM adoption.** Check for root `Directory.Packages.props`
   (`ManagePackageVersionsCentrally`) and `Directory.Build.props`. If absent, prepare
   the migration described in
   [package-governance-standard](../../../docs/standards/package-governance-standard.md).
3. **Run governance scans** (report commands even if you can't execute them):
   - `dotnet list package --vulnerable --include-transitive`
   - `dotnet list package --deprecated`
   - `dotnet list package --outdated --include-transitive`
   - Static check for package references not used in code (unused dependencies).
4. **Report** an inventory + findings grouped as **Vulnerable (blocking)**,
   **Deprecated**, **Outdated**, **Unused**, **Not centralized**, each with concrete
   `Directory.Packages.props`/`.csproj` edits and CLI remediation.
5. **Offer to apply**: create/normalize `Directory.Packages.props` &
   `Directory.Build.props`, strip inline versions, remove unused refs, and bump
   versions (flagging major upgrades as potentially breaking).

## Guardrails

- Pin exact versions; no floating ranges.
- Major-version bumps require explicit approval (breaking-change risk).
- **Build gate:** after any version/CPM change, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
