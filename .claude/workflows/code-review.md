# Workflow: Code Review (step-by-step, context-preserving, build-gated)

The primary orchestration for **"review my code against these standards and guardrails."**
It runs the standards **one step at a time, in order, without losing context**, and keeps
the solution **building successfully after every step**.

## Operating principles

- **Drive it from the main session.** Run each enforcer skill **inline** so findings,
  decisions, and edits accumulate in one continuous context. Do **not** fan every step
  out to fresh subagents — a new subagent starts cold and loses the review context.
  (Use the read-only `architecture-reviewer` / `standards-auditor` subagents only for the
  independent passes noted below, where a clean context is desirable.)
- **Maintain a Review Ledger** (see template) and update it after every step. It is the
  shared memory that carries context across steps.
- **Build gate after every step** per
  [build-verification-standard](../../docs/standards/build-verification-standard.md):
  no step starts on a red build.
- **Review by default, change on approval.** Apply only mechanical, behavior-preserving
  fixes automatically; list breaking changes for the user to approve. Never commit.

## Steps

### Step 0 — Scope & detect
- Determine scope: the user's named area, the current diff, or the whole solution.
- Run [project-type-detection](../../docs/standards/project-type-detection.md); record
  projects → types in the ledger.

### Step 1 — Baseline build (pre-flight gate)
- Restore + build (+ test if a suite exists) per the build-verification standard.
- **Red baseline → stop** and report; offer to fix the build first. **Green → record the
  known-good point** and continue.

### Steps 2–8 — Enforcers, one at a time (build after each)
Run in this order. For **each** step: apply fixes → **rebuild** → update the ledger
(build pass/fail, action taken) → only then move on. If a step breaks the build, fix-
forward or revert to the last known-good point and mark it *blocked*, then continue.

2. **Async** — `async-await-enforcer`
3. **Naming** — `naming-convention-enforcer` (apply non-breaking renames; list breaking)
4. **Exceptions** — `exception-handling-enforcer`
5. **Configuration** — `configuration-pattern-enforcer`
6. **Logging** — `logging-enforcer` (incl. `CorrelationId` + `InstanceId` scope)
7. **Correlation IDs** — `correlation-id-enforcer`
8. **Packages** — `package-governance` (rebuild after any version/CPM change)

### Step 9 — Tests
- `unit-test-generator` for new/changed public behavior; then **run the suite**. Treat
  new failures as a gate per the build standard.

### Step 10 — Architecture review (read-only)
- `architecture-reviewer` skill (delegate to the `architecture-reviewer` subagent for
  large solutions). No build needed — it does not modify code.

### Step 11 — Final verification & report
- Run a **full solution `dotnet build` and `dotnet test`** — the run succeeds only if both
  are green.
- Produce the consolidated review report from the ledger.

## Review Ledger (keep updated across steps)

```
Scope: <files/projects/diff>
Detected: <project → type, TFM>
Baseline: build=PASS  tests=PASS  (known-good @ start)

| Step | Standard        | Findings | Fixes applied | Breaking (await approval) | Build | Tests | Status   |
|------|-----------------|----------|---------------|---------------------------|-------|-------|----------|
| 2    | Async           | 3        | 3             | 0                         | PASS  | n.a.  | verified |
| 3    | Naming          | 5        | 4             | 1 (public DTO rename)     | PASS  | n.a.  | verified |
| ...  | ...             | ...      | ...           | ...                       | ...   | ...   | ...      |

Final: build=PASS  tests=PASS
```

## Output

- The completed Review Ledger.
- Findings grouped by standard with `file:line` evidence.
- Applied fixes vs. breaking changes awaiting approval.
- **Explicit final build + test status** (never implied).
- Any steps left *blocked* by a build failure, with the compiler error.
