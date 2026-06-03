# Pull Request

## Summary

<!-- What does this change do and why? Link issues with #id. -->

## Type of change

- [ ] Feature
- [ ] Bug fix
- [ ] Refactor / tech debt
- [ ] Documentation
- [ ] Repository automation (framework: skills/prompts/workflows/CI/MCP)

## Engineering standards checklist

> Standards are defined in [`docs/standards/`](../docs/standards/). Run the
> **pre-pr-compliance** workflow to verify.

- [ ] **Project type considered** — change suits the detected project type(s)
- [ ] **Async** — no `.Result`/`.Wait()`/`async void`; `CancellationToken` flows
- [ ] **Naming** — follows .NET conventions; no unflagged breaking renames
- [ ] **Exceptions** — centralized handling; no swallowed exceptions; `throw;` not `throw ex;`
- [ ] **Configuration** — Options Pattern + validation; no raw `IConfiguration` in business code; **no secrets committed**
- [ ] **Logging** — structured templates, correct levels, correlation scope, no PII
- [ ] **Correlation IDs** — entry points read/generate and propagate `X-Correlation-Id`
- [ ] **Tests** — added/updated; success, failure, and edge cases; suite passes
- [ ] **Packages** — centrally managed; no vulnerable/deprecated additions
- [ ] **Architecture** — respects layering/SOLID; no new circular references
- [ ] **README/docs** — updated if behavior, config, or deployment changed

## Verification

<!-- How was this validated? Commands run, tests, manual steps. -->

## Notes for reviewers

<!-- Breaking changes, follow-ups, anything flagged for approval. -->
