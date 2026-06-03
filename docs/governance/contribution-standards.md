# Governance: Contribution Standards

How to contribute to a repository that uses this framework, and to the framework itself.

## Before you start

- Read [`docs/architecture/framework-overview.md`](../architecture/framework-overview.md)
  and the relevant [`docs/standards/`](../standards/) docs.
- Confirm the change is in scope: this framework maintains **repository-level
  automation only** — no new .NET projects or business code unless explicitly requested
  (test projects excepted).

## Workflow

1. **Branch** from `main` with a descriptive name (`feature/...`, `fix/...`,
   `chore/...`).
2. **Make the change.** For code changes in a host repo, follow all standards.
3. **Run the `pre-pr-compliance` workflow** (or the individual enforcer skills) and fix
   findings.
4. **Tests** added/updated and passing ([testing-standard](../standards/testing-standard.md)).
5. **Open a PR** using the template; complete the standards checklist; link the issue.
6. CI (`standards-ci.yml`) and review must pass before merge.

## Commit & PR conventions

- Clear, imperative commit subjects; explain *why* in the body when non-obvious.
- Keep PRs focused and reviewable; flag breaking changes explicitly.
- Update docs/README when behavior, config, or deployment changes.

## Contributing to the framework itself

- Follow [`extending-the-framework`](../architecture/extending-the-framework.md).
- Keep `docs/standards/` the single source of truth; update all indexes
  (instructions table, framework overview, PR checklist) when adding a capability.
- Validate cross-references (no dangling links) and JSON/YAML validity before opening
  the PR — see the verification steps in `FRAMEWORK-README.md`.

## Definition of done

- [ ] Standards satisfied / violations flagged
- [ ] Tests pass
- [ ] Docs + indexes updated
- [ ] No secrets committed
- [ ] PR checklist complete
