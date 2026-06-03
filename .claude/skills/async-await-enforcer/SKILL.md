---
name: async-await-enforcer
description: Detect and fix synchronous/blocking code and async/await anti-patterns in .NET — eliminate .Result/.Wait()/.GetAwaiter().GetResult(), async void, missing CancellationToken, and sync-over-async. Use when the user asks to review or fix async usage, deadlocks, blocking calls, threadpool starvation, or enforce async best practices.
---

# Async/Await Enforcer

Find synchronous implementations and async anti-patterns and recommend (or apply)
asynchronous, non-blocking alternatives.

## Procedure

1. **Detect project types** ([project-type-detection](../../../docs/standards/project-type-detection.md))
   — note entry-point shapes (controllers, function methods, `ExecuteAsync`,
   Durable orchestrators).
2. **Scan for violations** defined in
   [async-standard](../../../docs/standards/async-standard.md):
   `.Result`, `.Wait()`, `.GetAwaiter().GetResult()`, `async void`, blocking I/O on
   async paths, `Thread.Sleep` in async, missing `CancellationToken` propagation,
   missing `ConfigureAwait(false)` in library projects, and Durable-orchestrator
   non-determinism.
3. **Report** each as `file:line`, offending pattern, recommended fix, and severity
   (deadlock-risk = error).
4. **Apply** the mechanical fixes directly (edit the files); flag any change that alters a
   public signature or cascades through callers (async-all-the-way ripple) and confirm
   before applying those. Do not stop at a list of suggestions.

## Guardrails

- Do not introduce new business logic — only async-correctness changes.
- Preserve behavior; when making a method async ripples to callers, list the full
  call-chain impact before applying.
- **Build gate:** after applying changes, rebuild per
  [build-verification-standard](../../../docs/standards/build-verification-standard.md);
  never leave the build broken — fix-forward or revert before proceeding.
