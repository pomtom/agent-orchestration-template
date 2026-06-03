# Workflow: Code Review (step-by-step, context-preserving, build-gated)

The primary orchestration for **"review my code against these standards and guardrails."**
It runs the standards **one step at a time, in order, without losing context**, and keeps
the solution **building successfully after every step**.

## ⚠️ Execution contract — read first

**This workflow MODIFIES code. It is NOT a report-only review.** For every step you MUST:

1. **Apply the fixes directly** to the files using the editor (Edit/Write) — do not just
   describe them.
2. **Run the build/tests yourself** with the `dotnet` CLI via the shell — do not assume.
3. Only after applying and building do you move to the next step.

> A run that ends with *"here's what should change"* **without having actually changed the
> files and rebuilt** is an **incomplete run**. Summaries describe what you **did**, not
> what someone *could* do. Keep going until the standards are applied and the final build
> + tests are green (or a step is genuinely blocked — then say so explicitly).

**The only things you pause and ask about before applying** are **breaking changes**:
renames of public API / serialized contract members, major NuGet version bumps, or any
change that alters observable behavior. List those, get a yes/no, then continue. Apply
**everything else** (mechanical, behavior-preserving fixes) without asking.

**Never `git commit` or `git push`** — leave changes staged in the working tree for the
user to review and commit.

## Operating principles

- **Drive it from the main session.** Run each enforcer skill **inline** so findings,
  decisions, and edits accumulate in one continuous context. Do **not** fan every step
  out to fresh subagents — a new subagent starts cold and loses the review context.
  (Use the read-only `architecture-reviewer` subagent only for Step 10, where a clean
  context is desirable and no edits are made.)
- **Maintain a Review Ledger** (see template) and update it after every step.
- **Build gate after every step** per
  [build-verification-standard](../../docs/standards/build-verification-standard.md):
  no step starts on a red build.

## Steps

### Step 0 — Scope & detect
- Determine scope: the user's named area, the current diff, or the whole solution.
- Run [project-type-detection](../../docs/standards/project-type-detection.md); record
  projects → types in the ledger.

### Step 1 — Baseline build (pre-flight gate)
- **Run** `dotnet restore` + `dotnet build` (+ `dotnet test` if a suite exists).
- **Red baseline → stop** and report the failure; offer to fix the build first.
  **Green → record the known-good point** and continue.

### Steps 2–8 — Enforcers, one at a time (APPLY, then build)
Run in this order. For **each** step:
**(a) find violations → (b) APPLY the mechanical fixes with Edit/Write → (c) run
`dotnet build` → (d) update the ledger → only then move on.** For breaking changes,
pause and ask (see Execution contract). If a step breaks the build, fix-forward or revert
to the last known-good point, mark it *blocked* with the compiler error, and continue.

2. **Async** — `async-await-enforcer`: replace `.Result`/`.Wait()`/`async void`, thread `CancellationToken`.
3. **Naming** — `naming-convention-enforcer`: apply non-breaking renames; list breaking ones for approval.
4. **Exceptions** — `exception-handling-enforcer`: add/centralize the handler; fix `throw ex;`, empty catches.
5. **Configuration** — `configuration-pattern-enforcer`: introduce Options classes + validation; remove raw `IConfiguration`.
6. **Logging** — `logging-enforcer`: structured templates, levels, `CorrelationId` + `InstanceId` scope.
7. **Correlation IDs** — `correlation-id-enforcer`: add middleware/handlers + propagation.
8. **Packages** — `package-governance`: centralize versions, remove unused; rebuild after changes.

### Step 9 — Tests
- **Generate and write** tests with `unit-test-generator` for new/changed public behavior,
  then **run** `dotnet test`. Treat new failures as a gate per the build standard.

### Step 10 — Architecture review (read-only)
- `architecture-reviewer` skill (delegate to the `architecture-reviewer` subagent for
  large solutions). This step **reports only** — it does not modify code.

### Step 11 — Final verification & report
- **Run** a full solution `dotnet build` **and** `dotnet test`. The run succeeds only if
  both are green.
- Produce the consolidated report from the ledger, describing **what was actually applied**.

## Review Ledger (keep updated across steps)

```
Scope: <files/projects/diff>
Detected: <project → type, TFM>
Baseline: build=PASS  tests=PASS  (known-good @ start)

| Step | Standard | Found | Fixes APPLIED | Breaking (awaiting approval) | Build | Tests | Status   |
|------|----------|-------|---------------|------------------------------|-------|-------|----------|
| 2    | Async    | 3     | 3 applied     | 0                            | PASS  | n.a.  | done     |
| 3    | Naming   | 5     | 4 applied     | 1 (public DTO rename)        | PASS  | n.a.  | partial  |
| ...  | ...      | ...   | ...           | ...                          | ...   | ...   | ...      |

Final: build=PASS  tests=PASS
```

## Output

- The completed Review Ledger showing **fixes that were applied** (not just proposed).
- Remaining breaking changes awaiting the user's approval.
- **Explicit final build + test status** (never implied).
- Any steps left *blocked* by a build failure, with the compiler error.
