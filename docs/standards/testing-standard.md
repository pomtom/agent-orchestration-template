# Standard: Unit Testing

> Canonical testing rules. Consumed by the `unit-test-generator` skill.

## Framework selection

- **Detect, don't impose.** Inspect existing test projects for the framework in use
  (xUnit, NUnit, MSTest) and the mocking library (Moq, NSubstitute, FakeItEasy) and
  the assertion style (FluentAssertions vs. built-in). Match what exists.
- If **no** test project exists, default to **xUnit + NSubstitute + FluentAssertions**
  and create a sibling test project named `<Project>.Tests` (test project only — this
  is permitted; it is not business code).

## Structure & naming

- One test class per unit under test: `<TypeUnderTest>Tests`.
- Test method name: `Method_Scenario_ExpectedOutcome`
  (e.g., `Handle_WhenOrderMissing_ThrowsNotFoundException`).
- Follow **Arrange–Act–Assert**, with the three sections visually separated.
- Prefer one logical assertion per test; group related assertions with FluentAssertions.

## Coverage expectations

- Cover **success**, **failure**, and **edge** cases for each public member.
- Edge cases: null/empty inputs, boundary values, cancellation, concurrency where
  relevant, and exception paths.
- Aim to maximize meaningful coverage — exercise branches, not just lines. Do not write
  vacuous tests purely to inflate the number.

## Dependencies & isolation

- Mock **all** external dependencies (I/O, HTTP, DB, message brokers, time, Azure SDKs).
- Inject abstractions; never hit the network or filesystem in a unit test.
- Use a fake/abstracted clock instead of `DateTime.Now`/`DateTimeOffset.UtcNow`.
- For async code, await results and assert on completion; verify no blocking. See
  [async-standard](./async-standard.md).

## Project-type specifics (after [detection](./project-type-detection.md))

- **Web API/MVC:** test controllers/handlers with mocked services; model validation;
  problem-details mapping per [exception-handling-standard](./exception-handling-standard.md).
- **Azure / Durable Functions:** test the function method directly with a mocked
  context, trigger bindings, and `ILogger`; for orchestrators, mock
  `TaskOrchestrationContext`/`IDurableOrchestrationContext` and assert the activity
  call sequence.
- **Worker Services:** test `ExecuteAsync` with a `CancellationToken`; assert graceful
  shutdown.
- **Blazor:** component tests via bUnit when present.

## Output rules

- Place tests in the matching test project, mirroring the source folder structure.
- Do not modify production code to make it testable without flagging the change.
- Each generated test file must compile against the detected framework and reference
  only packages already available or explicitly added to the test project.
