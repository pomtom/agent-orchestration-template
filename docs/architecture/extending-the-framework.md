# Extending the Framework

How to add or modify capabilities while keeping Claude Code and Copilot in lockstep.

## Golden rule

**Rules live in `docs/standards/`; everything else references them.** Never encode a
standard inside a skill, subagent, or `copilot-instructions.md` — point to the canonical
doc so both toolchains stay consistent.

## Add a new standard + skill

1. **Write the standard.** Create `docs/standards/<area>-standard.md`. Start with
   project-type detection, then Principles, Anti-patterns (table), and Enforcement
   output. Cross-link related standards.
2. **Create the skill.** Add `.claude/skills/<name>/SKILL.md` with YAML frontmatter:
   - `name`: kebab-case, matches the folder.
   - `description`: written so Claude auto-invokes it — include the trigger verbs and the
     project types it applies to.
   Body: a short procedure that (1) runs detection, (2) references the new standard,
   (3) reports `file:line` + fixes, (4) states guardrails.
3. **Wire Copilot.** Add a bullet to `.github/copilot-instructions.md` linking the new
   standard.
4. **Update indexes:**
   - `.claude/instructions/engineering-standards.md` table
   - `docs/architecture/framework-overview.md` capability map + diagram
   - `pull_request_template.md` checklist (if it gates PRs)
5. **Add to workflows** if it belongs in onboarding / pre-PR / tech-debt.
6. **CI (optional).** Add a check to `standards-ci.yml` if it can be machine-verified.

## Add a subagent

Create `.claude/agents/<name>.md` with frontmatter (`name`, `description`, `tools`,
`model`). Keep agents **read-only** for review tasks (`tools: Read, Grep, Glob, Bash`).
Use a subagent when a task is large enough to deserve its own context window (deep
reviews, full-repo sweeps).

## Add an MCP server

1. Add the server to `.mcp.json` with `${ENV_VAR}` placeholders (never real secrets).
2. Add a card under `.mcp/servers/<name>.md` (purpose, command, env, scopes, setup).
3. Add any new env vars to `.mcp/configurations/.env.example`.
4. Update the relevant example profile (`local.example.json` / `ci.example.json`).

## Conventions

- Keep skill descriptions specific and trigger-rich; vague descriptions don't auto-invoke.
- Prefer behavior-preserving, mechanical fixes by default; flag breaking changes.
- Don't generate business code or new projects (test projects excepted).
- Validate after changes: see the verification steps in `FRAMEWORK-README.md`.
