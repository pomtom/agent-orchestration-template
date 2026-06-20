# Engineering Standards (shared context)

This framework enforces one set of engineering standards across **any** .NET solution
(Web API, Azure/Durable Functions, MVC, Blazor, Worker, Console). The **canonical**
definitions live under `docs/standards/` and are the single source of truth shared by
Claude Code skills/agents and GitHub Copilot.

**Always run [project type detection](../../docs/standards/project-type-detection.md)
first** — every other standard adapts to the detected project type.

## The standards

| Area | Canonical doc | Skill |
|---|---|---|
| Project type detection | `docs/standards/project-type-detection.md` | (foundation for all) |
| Build verification (build gate) | `docs/standards/build-verification-standard.md` | (gate for all edits) |
| README generation | `docs/standards/readme-standard.md` | `readme-generator` |
| Unit testing | `docs/standards/testing-standard.md` | `unit-test-generator` |
| Async/await | `docs/standards/async-standard.md` | `async-await-enforcer` |
| Naming | `docs/standards/naming-standard.md` | `naming-convention-enforcer` |
| Exception handling | `docs/standards/exception-handling-standard.md` | `exception-handling-enforcer` |
| Configuration | `docs/standards/configuration-standard.md` | `configuration-pattern-enforcer` |
| Logging | `docs/standards/logging-standard.md` | `logging-enforcer` |
| Package governance | `docs/standards/package-governance-standard.md` | `package-governance` |
| Correlation IDs | `docs/standards/correlation-id-standard.md` | `correlation-id-enforcer` |
| CQRS / Mediator | `docs/standards/cqrs-standard.md` | `cqrs-pattern-enforcer` |
| Repository & Unit of Work | `docs/standards/repository-pattern-standard.md` | `repository-pattern-enforcer` |
| Dependency Injection | `docs/standards/dependency-injection-standard.md` | `dependency-injection-enforcer` |
| Architecture | `docs/standards/architecture-standard.md` | `architecture-reviewer` |

> **Standards split — assess vs. apply.** The `architecture-standard` is a **read-only
> assessment** of layering, SOLID, CQRS, repository, and DI. The pattern enforcers (`cqrs`,
> `repository`, `dependency-injection`) are the **apply/fix** companions for those same
> patterns. They must stay mutually consistent — the architecture standard reports, the
> pattern standards correct.

## Operating rules

- This framework creates and maintains **repository-level automation only** — prompts,
  skills, instructions, workflows, CI, and MCP config. It does **not** create .NET
  projects, Web APIs, Function Apps, Web Apps, or sample/business code, and does not
  generate business code unless explicitly requested. (Creating a missing **test**
  project is the only permitted exception, per the testing standard.)
- **Enforcer skills apply fixes by default — they are action-first, not advisory.** They
  edit the files directly for mechanical, behavior-preserving changes (recording
  `file:line` evidence in the report). Only **breaking** changes (public API / serialized
  contract renames, major version bumps, behavior changes) are paused for approval before
  applying. Ending with a summary of what *could* change, without changing it, is
  incomplete.
- Never introduce secrets into source; configuration secrets belong in User Secrets /
  Key Vault / App Settings.
- **Build gate.** Any step that modifies code must rebuild afterward per
  [build-verification-standard](../../docs/standards/build-verification-standard.md).
  Never leave the build broken — fix-forward or revert before the next step; no step
  starts on a red build. End multi-step runs with a full `dotnet build` + `dotnet test`
  and report the result explicitly.
- **Preserve context in multi-step reviews.** Run a step-by-step review (the
  `code-review` workflow) **inline in the main session** so findings and edits
  accumulate in one context; use read-only subagents only for independent passes where a
  clean context is wanted. Maintain the Review Ledger across steps.
