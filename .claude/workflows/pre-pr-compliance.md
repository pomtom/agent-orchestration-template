# Workflow: Pre-PR Compliance

Run before opening a pull request to ensure changes meet the engineering standards.

## When to use

You've made changes and want them to pass review (and the `standards-ci.yml` checks)
the first time.

## Steps

1. **Detect project types** (`docs/standards/project-type-detection.md`).

2. **Scope to the diff.** Prefer auditing changed files/projects; widen if changes are
   cross-cutting.

3. **Baseline build gate.** Restore + build (+ test) per
   `docs/standards/build-verification-standard.md`. If red, stop and report; if green,
   record the known-good point.

4. **Run the enforcers, one at a time, building after each** (each reports `file:line` +
   fixes; apply mechanical, behavior-preserving fixes, flag breaking ones; **rebuild
   after each step** and fix-forward/revert on a red build before continuing):
   - **`async-await-enforcer`** — no `.Result`/`.Wait()`/`async void`; tokens flow.
   - **`naming-convention-enforcer`** — names match the standard; breaking renames listed.
   - **`exception-handling-enforcer`** — centralized handling; no swallowed/`throw ex;`.
   - **`configuration-pattern-enforcer`** — Options pattern; no raw `IConfiguration`;
     validated; no committed secrets.
   - **`logging-enforcer`** — structured templates, levels, `CorrelationId`+`InstanceId`
     scope, no PII.
   - **`correlation-id-enforcer`** — entry points + propagation covered.

5. **Tests.** Invoke **`unit-test-generator`** for new/changed public behavior; ensure
   success/failure/edge coverage and that the suite builds and passes.

6. **Packages.** Run **`package-governance`** if dependencies changed — no vulnerable/
   deprecated packages; versions centralized; **rebuild after version/CPM changes**.

7. **Final verification & gate.** Run a full solution `dotnet build` + `dotnet test`.
   Summarize remaining violations by severity. **Block** on: a red final build,
   failing tests, deadlock-risk async, committed secrets, vulnerable packages, or
   missing centralized exception handling.

## Output

- Applied fixes summary + list of flagged breaking changes awaiting approval
- Per-step build status + **explicit final `dotnet build` + `dotnet test` result**
- A green/▲red compliance gate verdict mapped to the PR template checklist
