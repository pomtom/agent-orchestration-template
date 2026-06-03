# Standard: Build Verification (the build gate)

> Canonical rule for keeping the solution **green at every step** of an automated review
> or enforcement run. Consumed by the `code-review` and `pre-pr-compliance` workflows and
> by every enforcer skill that modifies code. Run
> [project-type-detection](./project-type-detection.md) first.

## The rule

**Never leave the build broken.** After *any* code modification, rebuild. A step is only
"done" when the solution still builds (and previously-passing tests still pass). If a
change breaks the build, **fix-forward or revert that change before moving to the next
step.** No step may start on top of a red build.

## Discover the build/test commands

1. **Solution:** prefer the root `*.sln`. If none, build every `*.csproj`.
2. **SDK:** honor `global.json` if present (pinned SDK); otherwise use the installed SDK
   matching the detected `TargetFramework`(s).
3. **Commands** (Windows/PowerShell or cross-platform `dotnet` CLI):
   - Restore: `dotnet restore <sln|csproj>`
   - Build:   `dotnet build <sln|csproj> -c Debug --nologo`  *(warnings-as-errors honored
     if the repo sets `TreatWarningsAsErrors`)*
   - Test:    `dotnet test <sln|csproj> -c Debug --nologo`
4. **Functions/Workers:** a normal `dotnet build` is sufficient for verification; do **not**
   start hosts (`func start`, `dotnet run`) — building, not running, is the gate.

## The gate sequence

1. **Baseline build (pre-flight).** Before changing anything, run restore + build (+ test
   if a suite exists). 
   - ❌ **Baseline red:** stop. Report the pre-existing failure; do not begin
     enforcement on an already-broken solution unless the user asks you to fix the build
     first.
   - ✅ **Baseline green:** record it as the known-good point and proceed.
2. **Per-step build.** After each enforcement step that edits code, rebuild the affected
   project(s) (or the solution for cross-cutting changes).
   - ✅ Green → mark the step verified; this becomes the new known-good point.
   - ❌ Red → diagnose and **fix-forward** if trivial; otherwise **revert this step's
     changes** back to the last known-good point and record the step as *blocked* with
     the compiler error. Continue with the remaining independent steps.
3. **Test gate.** After steps that change behavior (and always at the end), run the test
   suite. Treat new test failures the same as build failures.
4. **Final verification.** End every run with a full solution `dotnet build` **and**
   `dotnet test`. The run is only "successful" if both are green.

## Reporting

- For each step record: **build = pass/fail**, **tests = pass/fail/n.a.**, and the action
  taken (verified / fixed-forward / reverted-and-blocked).
- On any failure, capture the first compiler/test error (`file:line` + message).
- The final summary must state the end-state build and test status explicitly — never
  imply success without having run the final build.

## Guardrails

- Behavior-preserving by default; a "fix" that only silences the compiler by changing
  behavior is not acceptable — revert instead and flag for human review.
- Do not disable analyzers, downgrade `TreatWarningsAsErrors`, or comment out failing
  code to make the build pass.
- Never commit; the build gate verifies the working tree only and leaves committing to
  the user.
