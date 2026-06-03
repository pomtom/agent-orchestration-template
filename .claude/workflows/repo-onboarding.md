# Workflow: Repo Onboarding

Bring a new (or unfamiliar) .NET repository up to a known baseline: understand it,
document it, and assess it — without touching business code.

## When to use

A repo was just cloned, or you're picking up an existing solution and need a fast,
accurate picture plus baseline documentation.

## Steps

1. **Detect project types.**
   Apply `docs/standards/project-type-detection.md`. Produce the detection summary
   (projects → types, TFMs). This header feeds every later step.

2. **Generate the README.**
   Invoke the **`readme-generator`** skill → comprehensive `README.md` per
   `docs/standards/readme-standard.md`. If a README exists, produce a proposed diff.

3. **Architecture review.**
   Invoke the **`architecture-reviewer`** skill (delegating to the
   `architecture-reviewer` subagent for large solutions) → actionable architecture
   report per `docs/standards/architecture-standard.md`.

4. **Standards audit.**
   Run the **`standards-auditor`** subagent → consolidated compliance report across
   async, naming, exceptions, configuration, logging, correlation IDs, and packages.

5. **Summarize.**
   Present: detection summary, README status, architecture scorecard, and the top
   blocking compliance issues, with a recommended remediation order. Do **not** apply
   fixes in this workflow — onboarding is read/document only.

## Output

- `README.md` (or proposed diff)
- Architecture review report
- Standards compliance report
- A short prioritized next-steps list
