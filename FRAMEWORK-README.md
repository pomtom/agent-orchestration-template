# Claude Code Agentic Workflow Framework

A **reusable, repository-agnostic** engineering-automation framework for .NET solutions,
driven by **Claude Code** and **GitHub Copilot**. Drop it into any existing repository to
apply one consistent set of engineering standards — regardless of project type.

> **Scope:** This framework creates and maintains repository-level automation only —
> prompts, skills, instructions, workflows, CI, and MCP integrations. It does **not**
> create .NET projects, Web APIs, Function Apps, Web Apps, or sample/business code, and
> generates business code only when explicitly requested. (Creating a missing **test**
> project is the one permitted exception.)

## Quick Start

Once the framework files are in the **root of your .NET repo**, open Claude Code in that
repo and use one of these copy-paste prompts. Skills auto-invoke from natural language —
you describe the task, you don't name the file.

**Recommended first run** (read-only — detects project type, generates README, runs
architecture + compliance reports, changes no code):

```
Read .claude/instructions/engineering-standards.md, then run the repo-onboarding workflow on this solution.
```

**Review my code (step-by-step, build-gated):** runs each standard in order, keeps one
continuous context, and rebuilds after every step so the solution stays green:

```
Review my code using the code-review workflow. Run the standards step by step, keep context across steps, and make sure the project builds after each step.
```

**Other workflow entry points:**

```
Run the pre-pr-compliance workflow on my current changes.
Run the tech-debt-assessment workflow and give me a Now/Next/Later backlog.
```

**Trigger a single capability:**

| You want | Prompt |
|---|---|
| README | `Generate a comprehensive README for this solution.` |
| Tests | `Write unit tests for the OrdersService class.` |
| Async fixes | `Find and fix async/await violations in the Orders project.` |
| Naming | `Check naming conventions across the solution.` |
| Exceptions | `Review exception handling and add a centralized handler.` |
| Config | `Migrate this project to the Options pattern.` |
| Logging | `Enforce structured logging standards in this service.` |
| Packages | `Audit our NuGet packages and set up central package management.` |
| Correlation IDs | `Add X-Correlation-Id propagation to this Web API.` |
| Architecture | `Do an architecture review of this solution.` |

> **Review before commit.** Enforcer skills apply only mechanical, behavior-preserving
> fixes and flag breaking changes for approval — but they do **not** commit. Review every
> change (e.g. `git diff`) and commit yourself. Ask Claude to commit only when you
> explicitly want it; it will never auto-commit.

## Supported project types

ASP.NET Core Web API · Azure Functions · Durable Functions · ASP.NET MVC · Blazor ·
Worker Services · Console Applications. Every skill **auto-detects** the project type and
applies the same standards.

## What's inside

```
.claude/
├── agents/          architecture-reviewer, standards-auditor (read-only subagents)
├── instructions/    engineering-standards, project-type-detection (shared context)
├── skills/          10 capabilities (auto-invoked SKILL.md skills)
└── workflows/       code-review, repo-onboarding, pre-pr-compliance, tech-debt-assessment
.github/
├── copilot-instructions.md      Copilot follows the same standards
├── pull_request_template.md     checklist mapped to the standards
├── ISSUE_TEMPLATE/              bug / feature / tech-debt + config
└── workflows/standards-ci.yml   opt-in build/test + package & format checks
.mcp/
├── servers/         one card per MCP server (purpose, command, env, scopes)
└── configurations/  local / ci example profiles + .env.example
docs/
├── standards/       ← SINGLE SOURCE OF TRUTH (12 docs incl. detection + build gate)
├── architecture/    framework-overview, extending-the-framework
└── governance/      package-governance, security-policy, contribution-standards
.mcp.json            live config wiring all 8 MCP servers (env-var placeholders)
```

## Capabilities (skills)

| # | Skill | Does |
|---|---|---|
| 01 | `readme-generator` | Comprehensive README from the actual solution |
| 02 | `unit-test-generator` | Tests matching your framework; AAA; success/failure/edge |
| 03 | `async-await-enforcer` | Kills `.Result`/`.Wait()`/`async void`; flows tokens |
| 04 | `naming-convention-enforcer` | .NET naming across projects→events |
| 05 | `exception-handling-enforcer` | Centralized handling; ProblemDetails; correlation |
| 06 | `configuration-pattern-enforcer` | Options Pattern; validated; no raw `IConfiguration` |
| 07 | `logging-enforcer` | Structured `ILogger<T>`; App Insights; no PII |
| 08 | `package-governance` | CPM; vulnerable/deprecated/outdated/unused |
| 09 | `correlation-id-enforcer` | `X-Correlation-Id` read/generate/propagate |
| 10 | `architecture-reviewer` | Clean Arch/SOLID/CQRS/DI/security/perf report |

## Design principle: single source of truth

All rules live once in [`docs/standards/`](docs/standards/). Skills, subagents, and
`copilot-instructions.md` **reference** them — update a standard in one place and both
Claude Code and Copilot follow. See
[`docs/architecture/framework-overview.md`](docs/architecture/framework-overview.md).

## Install into an existing repo

1. Copy `.claude/`, `.github/`, `.mcp/`, `docs/`, `.mcp.json`, and (optionally) the
   `.gitignore` entries into the target repository root.
2. **MCP:** `cp .mcp/configurations/.env.example .mcp/configurations/.env`, set
   `WORKSPACE_ROOT` and any tokens you need. Credential-free servers (code-search,
   documentation-search, dependency-scanning) work immediately. Configure Azure/App
   Insights/security per the cards in `.mcp/servers/`.
3. **Copilot:** `.github/copilot-instructions.md` is picked up automatically in-editor.
4. **CI:** `standards-ci.yml` is opt-in (manual dispatch). Uncomment the `push`/
   `pull_request` triggers once the repo is compliant.
5. Open Claude Code in the repo — the 10 skills appear automatically. Run a workflow
   (e.g. **repo-onboarding**) to start.

## Usage

- **Review my code:** run the `code-review` workflow → standards enforced step-by-step in
  one continuous context, with a **build gate after every step** (solution stays green).
- **Onboard a repo:** run the `repo-onboarding` workflow → README + architecture +
  compliance reports.
- **Before a PR:** run `pre-pr-compliance` → enforcers + tests + package check.
- **Plan hardening:** run `tech-debt-assessment` → prioritized backlog.
- **One concern:** just ask (e.g. "enforce async standards on the Orders service") — the
  matching skill auto-invokes.

## Verify the install

- **Skills discovered:** every `.claude/skills/*/SKILL.md` has `name` + `description`
  frontmatter; they list in Claude Code.
- **MCP valid:** `.mcp.json` parses; `claude mcp list` shows servers; a credential-free
  server (code-search) starts.
- **No dangling links:** skills/agents/`copilot-instructions.md` link to existing
  `docs/standards/*.md`.
- **CI lint:** `standards-ci.yml` passes Actions schema validation.

## Extending

See [`docs/architecture/extending-the-framework.md`](docs/architecture/extending-the-framework.md)
to add a standard + skill, a subagent, or an MCP server.
