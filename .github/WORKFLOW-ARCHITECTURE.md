# GitHub Workflow Architecture

This document describes the complete GitHub Actions workflow orchestration for .NET projects using the Claude Code Agentic Workflow Framework.

## Overview

The workflow architecture implements a **defense-in-depth** strategy across multiple automation layers:

```
┌─────────────────────────────────────────────────────────────┐
│                    Developer Experience                      │
├─────────────────────────────────────────────────────────────┤
│  Claude Code Skills  →  Local Development & Review          │
│  Copilot             →  In-Editor Assistance                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Integration                        │
├─────────────────────────────────────────────────────────────┤
│  PR Template         →  Standards Checklist                 │
│  Issue Templates     →  Structured Problem Reporting        │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Automated Workflows                       │
├─────────────────────────────────────────────────────────────┤
│  Code Review (PR)    →  Every Pull Request                  │
│  Security Scan       →  Push + Schedule                     │
│  Standards CI        →  Manual + Optional Auto              │
│  Dependency Update   →  Weekly Schedule                     │
└─────────────────────────────────────────────────────────────┘
```

## Workflow Layers

### Layer 1: Local Development (Claude Code)

**Location:** `.claude/`

**Purpose:** Primary development workflow — executed locally before commit.

**Components:**
- **Skills** (`skills/*/SKILL.md`) — 13 auto-invoked capabilities
- **Workflows** (`workflows/*.md`) — Orchestrated multi-step processes
- **Agents** (`agents/`) — Read-only deep reviewers (architecture, standards)
- **Instructions** (`instructions/`) — Shared context and detection logic

**Workflows:**
1. **`code-review.md`** — Step-by-step standards enforcement with build gates
2. **`repo-onboarding.md`** — README + architecture + compliance reports
3. **`pre-pr-compliance.md`** — Quick check before creating a PR
4. **`tech-debt-assessment.md`** — Prioritized remediation backlog

**Key Principle:** Review before commit. Skills never auto-commit — developers review and commit manually.

### Layer 2: GitHub Integration

**Location:** `.github/`

**Purpose:** Standardized PR/issue templates and workflow automation.

**Components:**
- `pull_request_template.md` — Standards checklist for every PR
- `ISSUE_TEMPLATE/` — Structured bug/feature/tech-debt templates
- `copilot-instructions.md` — Aligns Copilot with Claude Code standards
- `workflows/` — CI/CD automation

### Layer 3: Automated Workflows

**Location:** `.github/workflows/`

#### 3.1 Code Review (PR) — `code-review-pr.yml`

**Trigger:** Every pull request to `main` or `develop`

**Purpose:** Automated validation of engineering standards.

**Jobs:**
1. **Project Detection** — Identify project types (Web API, Functions, etc.)
2. **Async Check** — Detect `.Result`, `.Wait()`, `async void`
3. **Naming Check** — Verify .NET naming conventions
4. **Exception Check** — Validate exception handling patterns
5. **Configuration Check** — Ensure no hardcoded secrets/connections
6. **Package Security** — Scan for vulnerable packages
7. **Build Verification** — Full build + test suite
8. **Review Summary** — Consolidated report

**Gates:**
- ❌ **Blocks merge:** Hardcoded secrets, vulnerable packages, build failures
- ⚠️ **Warns:** Async violations, naming issues, empty catches

#### 3.2 Security Scan — `security-scan.yml`

**Trigger:** Push to main/develop, PRs, daily at midnight, manual

**Purpose:** Comprehensive security analysis.

**Jobs:**
1. **Secret Detection** — Find hardcoded API keys, tokens, passwords
2. **Code Analysis** — Security-focused static analysis
3. **Dependency Scan** — Vulnerable package detection
4. **Configuration Security** — Verify no secrets in config files
5. **CORS Security** — Check for permissive CORS policies
6. **Authentication Check** — Validate auth configuration
7. **Security Summary** — Consolidated security report

**Gates:**
- ❌ **Fails:** Hardcoded secrets, vulnerable packages, disabled JWT validation
- ⚠️ **Warns:** Permissive CORS, anonymous access patterns

#### 3.3 Standards CI — `standards-ci.yml`

**Trigger:** Manual dispatch, optional auto-trigger

**Purpose:** Drop-in-safe engineering standards enforcement.

**Jobs:**
1. **Build & Test** — Full solution build and test execution
2. **Standards Check** — Format, vulnerabilities, deprecations

**Special Features:**
- **Opt-in by design** — Only runs manually until enabled
- **Non-blocking** — `continue-on-error: true` by default
- **Progressive enablement** — Uncomment triggers once compliant

#### 3.4 Dependency Update — `dependency-update.yml`

**Trigger:** Weekly (Monday 9 AM), manual dispatch

**Purpose:** Proactive dependency management.

**Jobs:**
1. **Check Updates** — Find outdated packages
2. **Check Vulnerable** — Detect vulnerable dependencies
3. **Check Deprecated** — Identify deprecated packages
4. **Create Issue** — Auto-create security issues for vulnerabilities
5. **Upload Reports** — Artifact generation for review

**Automation:**
- Creates GitHub issues for vulnerable packages
- Uploads reports as artifacts
- Labels: `security`, `dependencies`, `tech-debt`

## Workflow Orchestration Patterns

### Pattern 1: Build Gate (Code Review)

Every standards check must pass a build gate:

```yaml
needs: [async-check, naming-check, exception-check, ...]
steps:
  - name: Build
    run: dotnet build --configuration Release
  - name: Test
    run: dotnet test --configuration Release
```

### Pattern 2: Fail Fast (Security)

Critical security issues block immediately:

```yaml
if grep -q "Password="; then
  echo "::error::Hardcoded secrets found"
  exit 1
fi
```

### Pattern 3: Progressive Disclosure (Standards CI)

Start non-blocking, enable enforcement when ready:

```yaml
# Initially: continues even on failure
continue-on-error: true

# After compliance: remove to enforce
# continue-on-error: true
```

### Pattern 4: Scheduled Maintenance (Dependencies)

Weekly automated checks with issue creation:

```yaml
schedule:
  - cron: '0 9 * * 1'  # Every Monday

- name: Create issue if vulnerabilities found
  if: steps.vulnerable.outputs.has_vulnerabilities == 'true'
```

## Integration Points

### Claude Code ↔ GitHub Workflows

**Local → Remote:**
1. Developer runs `pre-pr-compliance` workflow (Claude Code)
2. Commits changes after review
3. Creates PR → triggers `code-review-pr.yml`
4. GitHub validates same standards automatically

**Shared Standards:**
- Both reference `docs/standards/*.md` as single source of truth
- Copilot instructions mirror Claude Code standards
- PR template checklist matches enforcer capabilities

### Skills ↔ Workflows

**Skill Coverage:**

| Skill | Claude Code | GitHub Workflow | Layer |
|-------|-------------|-----------------|-------|
| Async | `async-await-enforcer` | `async-check` job | Both |
| Naming | `naming-convention-enforcer` | `naming-check` job | Both |
| Exceptions | `exception-handling-enforcer` | `exception-check` job | Both |
| Config | `configuration-pattern-enforcer` | `configuration-check` job | Both |
| Logging | `logging-enforcer` | (code-analysis) | Local |
| Correlation | `correlation-id-enforcer` | (code-analysis) | Local |
| Packages | `package-governance` | `package-security` job | Both |
| Tests | `unit-test-generator` | `build-verification` job | Both |
| Architecture | `architecture-reviewer` | (local only) | Local |
| README | `readme-generator` | (local only) | Local |

### MCP Integration

**Location:** `.mcp/`

MCP servers extend Claude Code capabilities:

- **Code Search** — Fast symbol lookup across solution
- **Documentation Search** — Microsoft Learn integration
- **Dependency Scanning** — Package vulnerability detection
- **Azure Integration** — App Insights, Key Vault, etc.

**Workflow Enhancement:**
MCP servers provide Claude Code with deeper context during local reviews, improving fix accuracy before GitHub workflows run.

## Deployment Strategy

### Phase 1: Install Framework

```bash
# Copy framework files to target repo
cp -r .claude .github .mcp docs .mcp.json target-repo/
```

### Phase 2: Configure MCP (Optional)

```bash
cd target-repo
cp .mcp/configurations/.env.example .mcp/configurations/.env
# Edit .env with workspace root and tokens
```

### Phase 3: Run Initial Assessment

```
# In Claude Code:
Read .claude/instructions/engineering-standards.md, then run the repo-onboarding workflow on this solution.
```

### Phase 4: Enable Workflows Progressively

1. **Week 1:** Manual `standards-ci.yml` runs
2. **Week 2:** Enable `code-review-pr.yml` (PR checks)
3. **Week 3:** Enable `security-scan.yml` (daily scans)
4. **Week 4:** Uncomment `standards-ci.yml` auto-triggers
5. **Ongoing:** `dependency-update.yml` runs weekly automatically

### Phase 5: Team Adoption

- Train team on Claude Code workflows
- Review PR template checklist
- Establish "review before commit" culture
- Monitor GitHub Actions summary reports

## Monitoring & Observability

### GitHub Actions

**Job Summaries:**
Every workflow writes to `$GITHUB_STEP_SUMMARY`:
- ✅ Passed checks
- ⚠️ Warnings
- ❌ Failures with remediation links

**Artifacts:**
- Test results (`.trx` files)
- Dependency reports
- Security scan outputs

### Issue Automation

**Auto-created issues:**
- Vulnerable packages → `security` label
- Outdated dependencies → `dependencies` label
- Standards violations → `tech-debt` label

## Extending the Architecture

### Adding a New Workflow

1. **Define the standard** in `docs/standards/new-standard.md`
2. **Create Claude Code skill** in `.claude/skills/new-skill/SKILL.md`
3. **Add GitHub workflow job** in `.github/workflows/code-review-pr.yml`
4. **Update PR template** with new checklist item
5. **Update this doc** with orchestration details

### Adding a New Job to Existing Workflow

```yaml
new-check:
  name: New Standard Check
  runs-on: ubuntu-latest
  needs: project-detection  # Dependency chain
  steps:
    - uses: actions/checkout@v4
    - name: Run check
      run: |
        # Check logic
        echo "::error::Issue found" && exit 1
```

### Adding a New Workflow File

```yaml
name: New Workflow
on:
  pull_request: {}
  workflow_dispatch: {}

permissions:
  contents: read

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - # Your jobs here
```

## Standards Reference

All workflows enforce standards documented in:

- `docs/standards/async-standard.md`
- `docs/standards/naming-standard.md`
- `docs/standards/exception-handling-standard.md`
- `docs/standards/configuration-standard.md`
- `docs/standards/logging-standard.md`
- `docs/standards/correlation-id-standard.md`
- `docs/standards/package-governance-standard.md`
- `docs/standards/testing-standard.md`
- `docs/standards/architecture-standard.md`
- `docs/standards/project-type-detection.md`
- `docs/standards/build-verification-standard.md`
- `docs/standards/readme-standard.md`

## Troubleshooting

### Workflow Fails on First Run

**Cause:** Repo not yet compliant with standards.

**Solution:**
1. Run `standards-ci.yml` manually to see failures
2. Use Claude Code `code-review` workflow to fix issues
3. Re-run workflow after fixes

### False Positives in Security Scan

**Cause:** Pattern matching can over-detect.

**Solution:**
1. Review the specific line flagged
2. If legitimate, add inline comment explaining
3. Adjust regex in workflow if pattern is too broad

### Build Gate Blocks PR

**Cause:** Changes broke the build or tests.

**Solution:**
1. Run `dotnet build` and `dotnet test` locally
2. Fix issues identified
3. Use Claude Code `build-verification-standard.md` guidance
4. Commit fixes and push

### Dependency Update Creates Too Many Issues

**Cause:** Backlog of outdated packages.

**Solution:**
1. Temporarily increase issue creation threshold
2. Batch update packages in phases
3. Re-enable normal threshold once current

## Success Metrics

Track these metrics to measure workflow effectiveness:

- **Pre-merge defect rate** — Issues caught by workflows before merge
- **Standards compliance** — % of standards checks passing
- **Vulnerability detection time** — Days from CVE to detection
- **Build success rate** — % of PRs passing build gate
- **Review cycle time** — Time from PR creation to merge

## References

- [Framework README](../../FRAMEWORK-README.md)
- [Standards Documentation](../../docs/standards/)
- [Extending the Framework](../../docs/architecture/extending-the-framework.md)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
