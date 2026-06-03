---
name: unit-test-generator
description: Generate comprehensive, production-ready unit tests for .NET code using xUnit + Moq + Microsoft.NET.Test.Sdk. Covers EVERY public function with full positive, negative, and edge-case scenarios; mocks all dependencies; asserts the project's exception hierarchy; and produces complete, runnable test files ready for immediate integration. Use when the user asks to write, generate, add, or improve unit tests, increase coverage, or test a class/handler/function/service/controller.
---

# Unit Test Generator (xUnit + Moq)

Generate **complete, runnable, production-ready** unit tests that cover **every single
public function** in the target code, exercising both **positive (happy-path)** and
**negative (failure)** behavior, plus edge cases — while honoring the project's
architectural patterns, exception hierarchy, and existing testing conventions.

> Output contract: every generated test file must compile and pass as-is — **no
> `TODO`s, no placeholder bodies, no pseudo-code, no `throw new NotImplementedException()`**.
> If you cannot determine a behavior, write the test against the observable contract and
> note the assumption in a code comment, but keep the test runnable.

---

## Tech stack (pinned)

Generate against this stack unless the repo already uses a different one (see Step 2):

| Concern | Package |
|---|---|
| Test framework | **xUnit** (`xunit`) |
| Mocking | **Moq** |
| Test SDK / host | **Microsoft.NET.Test.Sdk** |
| Test runner (required to run) | `xunit.runner.visualstudio` |
| Coverage collector | `coverlet.collector` |

Assertions use xUnit's built-in `Assert` (and `Assert.ThrowsAsync`) — no extra assertion
library is required, keeping the dependency surface to exactly what's listed.

---

## Procedure

### Step 1 — Detect project types
Run [project-type-detection](../../../docs/standards/project-type-detection.md). Note for
each target whether it is a controller, MediatR handler, service, repository, Azure /
Durable Function, worker, or Blazor component — the test shape differs (see
[Per-type recipes](#per-type-recipes)).

### Step 2 — Detect or establish the test stack
- **Match what exists.** If the solution already has test projects, detect the framework
  (xUnit/NUnit/MSTest), mocking library, and assertion style, and **follow them** for
  consistency — do not mix frameworks in one project.
- **Otherwise default to the pinned stack** (xUnit + Moq + Microsoft.NET.Test.Sdk) and
  create a sibling test project `<Project>.Tests` (a test project is permitted; it is not
  business code). Use the [csproj template](#test-project-csproj-template) below.

### Step 3 — Enumerate the surface to cover
For each type under test, list **every** member to be tested:
- All `public` (and `internal` when `[InternalsVisibleTo]` is present) methods, including
  overloads, constructors with logic/guards, public properties with non-trivial getters/
  setters, indexers, and operators.
- For each method, enumerate every **branch/path**: each `if`/`switch`/`?:`, guard clause,
  loop boundary, `catch` block, and early return is a distinct scenario to cover.
- Skip trivial auto-properties and compiler-generated members.

### Step 4 — Build the scenario matrix (positive + negative + edge)
For **every** function, generate tests across all applicable categories:

**Positive (happy path)**
- Valid, typical inputs → expected return value / state change.
- Representative valid variations via `[Theory]` + `[InlineData]`/`[MemberData]`.
- Correct interactions with dependencies (verified with Moq `Verify`).

**Negative (failure / defensive)**
- `null` arguments → `ArgumentNullException` (one test per nullable parameter).
- Empty/whitespace strings, empty collections → `ArgumentException`/domain rule.
- Out-of-range / invalid values → `ArgumentOutOfRangeException` or domain exception.
- Dependency throws → assert the method propagates or wraps it per the
  **exception hierarchy** ([exception-handling-standard](../../../docs/standards/exception-handling-standard.md)):
  use `Assert.Throws<TSpecificException>` / `Assert.ThrowsAsync<TSpecificException>` and
  assert message/properties (e.g. correlation id, status mapping) where defined. Never
  assert on the base `Exception` when a specific domain type is expected.
- Not-found / conflict / unauthorized domain conditions → the specific domain exception.
- Validation failures → the project's validation exception/result type.

**Edge / boundary**
- Boundary values (min, max, zero, off-by-one, first/last element).
- Empty vs. single vs. many for collections.
- Duplicate/idempotency, ordering, and concurrency where relevant.
- Cancellation: pass an already-cancelled `CancellationToken` → assert
  `OperationCanceledException`/`TaskCanceledException` for async methods that accept one.
- Time-dependent logic exercised through an abstracted clock (never `DateTime.Now`).
- Culture/precision (decimals, rounding) where applicable.

**State & interaction**
- Verify side effects (repository `SaveChangesAsync`, publisher `Publish`, logger calls)
  with `mock.Verify(..., Times.Once())` and assert **no unexpected** calls with
  `mock.VerifyNoOtherCalls()` where strictness adds value.

### Step 5 — Generate complete test files
- One test class per unit under test: `<TypeUnderTest>Tests`.
- Mirror the source folder structure inside the test project; namespace
  `<SourceNamespace>.Tests` (or the repo's existing convention).
- Include all `using`s, the namespace, the class, fixtures, and fully-written tests.
- Apply the [conventions](#conventions) and [Moq patterns](#moq-patterns) below.

### Step 6 — Build gate (mandatory)
After writing tests, **build the solution and run the suite** per
[build-verification-standard](../../../docs/standards/build-verification-standard.md):
`dotnet build` then `dotnet test`. Generated tests must compile and pass. Fix-forward or
revert before finishing — never leave the build or tests red.

---

## Conventions

- **AAA:** every test has clearly separated `// Arrange`, `// Act`, `// Assert` sections.
- **Naming:** `Method_Scenario_ExpectedOutcome`
  (e.g. `HandleAsync_WhenOrderNotFound_ThrowsOrderNotFoundException`,
  `Calculate_WithZeroQuantity_ReturnsZero`).
- **One logical assertion per test** (a single behavior); multiple `Assert` lines are fine
  when they verify one outcome (e.g. status + body).
- **Parameterize** with `[Theory]` + `[InlineData]` for simple cases and `[MemberData]`/
  a `TheoryData<>` for complex/typed cases. Don't loop inside one `[Fact]`.
- **Isolation:** no real I/O, network, DB, filesystem, or `Thread.Sleep`. Mock every
  external dependency. Inject an abstracted clock (e.g. `TimeProvider`/`IClock`) instead
  of `DateTime.Now`/`DateTimeOffset.UtcNow`.
- **Async:** `await` the system under test; assert failures with `await Assert.ThrowsAsync<T>`.
  Never block with `.Result`/`.Wait()` ([async-standard](../../../docs/standards/async-standard.md)).
- **Determinism:** no random/time/order dependence; seed any randomness; fix culture if
  the code is culture-sensitive.
- **Test data builders / `ObjectMother`:** use small builder helpers or factory methods for
  complex entities to keep Arrange readable and DRY.
- **Fixtures:** use `IClassFixture<T>` for shared expensive setup and `ICollectionFixture<T>`
  for cross-class sharing; otherwise construct fresh state per test (xUnit creates a new
  class instance per test — leverage the constructor for common Arrange).
- **No shared mutable state** across tests; each test stands alone and can run in parallel.

---

## Moq patterns

```csharp
// Strict mocks catch unexpected calls; loose mocks are fine when you only assert some calls.
var repo = new Mock<IOrderRepository>(MockBehavior.Strict);

// Returns / async returns
repo.Setup(r => r.GetByIdAsync(orderId, It.IsAny<CancellationToken>()))
    .ReturnsAsync(order);

// Throwing to drive the negative path
repo.Setup(r => r.GetByIdAsync(missingId, It.IsAny<CancellationToken>()))
    .ReturnsAsync((Order?)null); // or .ThrowsAsync(new OrderNotFoundException(missingId));

// Argument matching: It.IsAny / It.Is for precise expectations
publisher.Setup(p => p.PublishAsync(It.Is<OrderCancelledEvent>(e => e.OrderId == orderId),
                                    It.IsAny<CancellationToken>()))
         .Returns(Task.CompletedTask);

// Callback to capture arguments for assertion
Order? saved = null;
repo.Setup(r => r.SaveAsync(It.IsAny<Order>(), It.IsAny<CancellationToken>()))
    .Callback<Order, CancellationToken>((o, _) => saved = o)
    .Returns(Task.CompletedTask);

// Verify interactions (and absence of others)
repo.Verify(r => r.SaveAsync(It.IsAny<Order>(), It.IsAny<CancellationToken>()), Times.Once);
repo.VerifyNoOtherCalls();

// Verifying ILogger<T> was called at a level (logging-standard)
logger.Verify(l => l.Log(
    LogLevel.Error,
    It.IsAny<EventId>(),
    It.IsAny<It.IsAnyType>(),
    It.IsAny<Exception>(),
    (Func<It.IsAnyType, Exception?, string>)It.IsAny<object>()), Times.Once);
```

---

## Test project csproj template

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework> <!-- match the project under test -->
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.*" />
    <PackageReference Include="xunit" Version="2.*" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.*">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
    <PackageReference Include="Moq" Version="4.*" />
    <PackageReference Include="coverlet.collector" Version="6.*">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\<ProjectUnderTest>\<ProjectUnderTest>.csproj" />
  </ItemGroup>
</Project>
```
If the repo uses Central Package Management
([package-governance-standard](../../../docs/standards/package-governance-standard.md)),
omit the `Version` attributes and add `<PackageVersion>` entries to
`Directory.Packages.props` instead.

---

## Per-type recipes

- **Service / domain class:** construct with mocked collaborators; cover each method's
  positive, negative (incl. domain exceptions), and edge scenarios; verify side effects.
- **MediatR / CQRS handler:** `new THandler(mocks...)`; test `Handle`/`HandleAsync` for the
  success result, each validation/guard failure, not-found/conflict exceptions, and that it
  calls the repository/publisher exactly as expected. Pipeline behaviors tested separately.
- **ASP.NET Core controller:** invoke the action with mocked services; assert the
  `IActionResult` type and status (`OkObjectResult`, `NotFoundResult`, etc.), the returned
  model, model-state/validation handling, and `ProblemDetails` on error paths.
- **Repository:** prefer testing logic via abstractions/mocks; for EF Core use the
  in-memory or SQLite-in-memory provider **only** for repository-level tests, asserting
  query/translation behavior — keep it out of pure unit tests of higher layers.
- **Azure Function (HTTP/Queue/Timer/ServiceBus):** call the function method directly with
  a mocked trigger input, mocked bindings, mocked `ILogger`/`FunctionContext`; assert the
  response/output binding and the success + failure + cancellation paths.
- **Durable orchestrator:** mock `TaskOrchestrationContext`/`IDurableOrchestrationContext`;
  assert the **sequence** of `CallActivityAsync` calls, fan-out/fan-in, retry options, and
  error handling; keep it deterministic (no real time/IO).
- **Durable activity:** test like a normal method with mocked dependencies.
- **Worker / `BackgroundService`:** test `ExecuteAsync` with a `CancellationToken`; assert
  it processes work and shuts down gracefully when cancelled.
- **Blazor component:** use bUnit if already present; assert rendered output and event
  callbacks. (Add bUnit only if the repo already uses it.)

---

## Worked example (service, xUnit + Moq)

```csharp
using Moq;
using Xunit;

namespace Contoso.Orders.Application.Tests.Orders;

public sealed class CancelOrderHandlerTests
{
    private readonly Mock<IOrderRepository> _repository = new(MockBehavior.Strict);
    private readonly Mock<IEventPublisher> _publisher = new(MockBehavior.Strict);
    private readonly CancelOrderHandler _sut;

    public CancelOrderHandlerTests()
        => _sut = new CancelOrderHandler(_repository.Object, _publisher.Object);

    // ---------- Positive ----------
    [Fact]
    public async Task HandleAsync_WhenOrderIsCancellable_CancelsAndPublishesEvent()
    {
        // Arrange
        var order = new Order(OrderId: 1, status: OrderStatus.Pending);
        _repository.Setup(r => r.GetByIdAsync(1, It.IsAny<CancellationToken>()))
                   .ReturnsAsync(order);
        _repository.Setup(r => r.SaveAsync(order, It.IsAny<CancellationToken>()))
                   .Returns(Task.CompletedTask);
        _publisher.Setup(p => p.PublishAsync(It.Is<OrderCancelledEvent>(e => e.OrderId == 1),
                                             It.IsAny<CancellationToken>()))
                  .Returns(Task.CompletedTask);

        // Act
        await _sut.HandleAsync(new CancelOrderCommand(1), CancellationToken.None);

        // Assert
        Assert.Equal(OrderStatus.Cancelled, order.Status);
        _repository.Verify(r => r.SaveAsync(order, It.IsAny<CancellationToken>()), Times.Once);
        _publisher.Verify(p => p.PublishAsync(It.IsAny<OrderCancelledEvent>(),
                                              It.IsAny<CancellationToken>()), Times.Once);
        _repository.VerifyNoOtherCalls();
        _publisher.VerifyNoOtherCalls();
    }

    // ---------- Negative ----------
    [Fact]
    public async Task HandleAsync_WhenOrderNotFound_ThrowsOrderNotFoundException()
    {
        // Arrange
        _repository.Setup(r => r.GetByIdAsync(99, It.IsAny<CancellationToken>()))
                   .ReturnsAsync((Order?)null);

        // Act + Assert
        var ex = await Assert.ThrowsAsync<OrderNotFoundException>(
            () => _sut.HandleAsync(new CancelOrderCommand(99), CancellationToken.None));
        Assert.Equal(99, ex.OrderId);
        _repository.Verify(r => r.SaveAsync(It.IsAny<Order>(), It.IsAny<CancellationToken>()), Times.Never);
    }

    [Fact]
    public async Task HandleAsync_WhenCommandIsNull_ThrowsArgumentNullException()
        => await Assert.ThrowsAsync<ArgumentNullException>(
            () => _sut.HandleAsync(null!, CancellationToken.None));

    [Theory]
    [InlineData(OrderStatus.Shipped)]
    [InlineData(OrderStatus.Cancelled)]
    public async Task HandleAsync_WhenOrderNotCancellable_ThrowsInvalidOrderStateException(OrderStatus status)
    {
        // Arrange
        var order = new Order(OrderId: 1, status: status);
        _repository.Setup(r => r.GetByIdAsync(1, It.IsAny<CancellationToken>()))
                   .ReturnsAsync(order);

        // Act + Assert
        await Assert.ThrowsAsync<InvalidOrderStateException>(
            () => _sut.HandleAsync(new CancelOrderCommand(1), CancellationToken.None));
    }

    // ---------- Edge ----------
    [Fact]
    public async Task HandleAsync_WhenCancelled_ThrowsOperationCanceledException()
    {
        using var cts = new CancellationTokenSource();
        cts.Cancel();
        _repository.Setup(r => r.GetByIdAsync(1, cts.Token))
                   .ThrowsAsync(new OperationCanceledException());

        await Assert.ThrowsAsync<OperationCanceledException>(
            () => _sut.HandleAsync(new CancelOrderCommand(1), cts.Token));
    }
}
```
The example is illustrative — replace types, members, and exception names with the
**actual** code under test and its real exception hierarchy.

---

## Guardrails

- **Cover every public function and every branch** — do not stop at the happy path.
- **Always include negative tests**: null args, invalid input, and the specific domain
  exceptions from the project's hierarchy (not the base `Exception`).
- Match the existing test framework if one is present; otherwise use the pinned stack.
- Reference only packages available to the test project (add them to the csproj / CPM).
- Do not weaken production code to make it testable without flagging the change; prefer
  testing through existing seams (interfaces, virtual members, DI).
- **No placeholders or skipped tests** — every emitted test is complete and runnable.
- **Build gate:** after adding tests, run `dotnet build` and `dotnet test` per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  do not leave the build or tests red — fix-forward or revert before finishing.
