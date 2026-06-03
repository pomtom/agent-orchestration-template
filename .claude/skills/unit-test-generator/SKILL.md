---
name: unit-test-generator
description: Generate comprehensive unit tests for .NET code following the existing test framework and conventions — Arrange-Act-Assert, mocked dependencies, meaningful names, and success/failure/edge-case coverage. Use when the user asks to write, generate, add, or improve unit tests, increase coverage, or test a class/handler/function/service.
---

# Unit Test Generator

Generate meaningful unit tests that maximize useful coverage and match the codebase's
existing conventions.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   to know what you are testing (controller, handler, function, orchestrator, worker).
2. **Detect the test stack.** Inspect existing test projects for framework (xUnit/
   NUnit/MSTest), mocking library, and assertion style; **match them**. If none exist,
   use the defaults in
   [testing-standard](../../../docs/standards/testing-standard.md) and create a
   `<Project>.Tests` project (test project only — permitted).
3. **For each target type**, generate tests per
   [testing-standard](../../../docs/standards/testing-standard.md):
   - Arrange-Act-Assert with clear separation.
   - `Method_Scenario_ExpectedOutcome` naming.
   - Mock all external dependencies; abstract the clock; no real I/O.
   - Cover success, failure, and edge cases (null/empty, boundaries, cancellation,
     exception paths).
   - Apply the project-type specifics (functions/orchestrators/workers/Blazor).
4. **Place tests** mirroring the source folder structure in the matching test project.

## Guardrails

- Do not weaken production code to make it testable without flagging the change.
- Reference only packages available to (or explicitly added to) the test project.
- Generated tests must compile against the detected framework.
- **Build gate:** after adding tests, build the solution **and run the suite** per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  do not leave the build or tests red — fix-forward or revert before proceeding.
