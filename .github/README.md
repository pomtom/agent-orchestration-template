# GitHub Integration for Claude Code Framework

This folder contains GitHub-specific integrations for the Claude Code Agentic Workflow Framework, providing seamless orchestration between local development and CI/CD automation.

## Overview

The `.github` folder implements a **three-layer defense** strategy:

1. **Templates** — Standardized PR/issue workflows
2. **Instructions** — Copilot alignment with Claude Code standards
3. **Workflows** — Automated validation and deployment

```
.github/
├── ISSUE_TEMPLATE/              Issue templates for structured reporting
│   ├── bug_report.md           Bug report template
│   ├── feature_request.md      Feature request template
│   ├── tech_debt.md            Technical debt template
│   └── config.yml              Template configuration
├── workflows/                   GitHub Actions workflows
│   ├── standards-ci.yml        Core standards enforcement
│   ├── code-review-pr.yml      Automated PR validation
│   ├── security-scan.yml       Security scanning
│   ├── dependency-update.yml   Automated dependency checks
│   └── cicd-pipeline.yml       Complete CI/CD deployment
├── copilot-instructions.md      Copilot engineering standards
├── pull_request_template.md     PR checklist
├── WORKFLOW-ARCHITECTURE.md     Complete architecture documentation
└── ORCHESTRATION-GUIDE.md       Operational guide
```

## Quick Start

### 1. Enable Workflows Progressively

**Week 1: Manual Testing**
```bash
# Test workflows manually first
gh workflow run standards-ci.yml
gh workflow run security-scan.yml
```

**Week 2: Enable PR Checks**
```yaml
# Already enabled by default in code-review-pr.yml
on:
  pull_request:
    branches: [main, develop]
```

**Week 3: Enable CI**
```yaml
# In standards-ci.yml, uncomment:
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

**Week 4: Production**
- Remove `continue-on-error: true` from standards-ci.yml
- Configure cicd-pipeline.yml with deployment targets
- Set up GitHub environments (staging, production)

### 2. Configure Branch Protection

```bash
# Settings → Branches → Add rule for 'main'
✅ Require a pull request before merging
✅ Require status checks to pass before merging
   ✅ code-review-pr / build-verification
   ✅ security-scan / dependency-scan
✅ Require conversation resolution before merging
✅ Do not allow bypassing the above settings
```

### 3. Set Up Environments

```bash
# Settings → Environments → New environment
Name: staging
✅ Required reviewers: [team-leads]
✅ Wait timer: 0 minutes

Name: production
✅ Required reviewers: [team-leads, tech-leads]
✅ Wait timer: 10 minutes
```

## Workflows

### Core Workflows (Always Enabled)

#### `standards-ci.yml`
**Purpose:** Core engineering standards enforcement.
**Trigger:** Manual dispatch (opt-in by default)
**Jobs:**
- Build & Test
- Format check
- Package vulnerability scan
- Deprecated package report
- Outdated package report

**Enable auto-trigger:**
```yaml
# Uncomment in standards-ci.yml:
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
```

#### `code-review-pr.yml`
**Purpose:** Automated PR validation against all standards.
**Trigger:** Every pull request
**Jobs:**
1. Project type detection
2. Async/await validation
3. Naming conventions
4. Exception handling
5. Configuration patterns
6. Package security
7. Build verification
8. Review summary

**Customization:**
```yaml
# Add additional branches
on:
  pull_request:
    branches: [main, develop, release/*]
```

### Security Workflows

#### `security-scan.yml`
**Purpose:** Comprehensive security analysis.
**Trigger:** Push, PR, daily midnight, manual
**Jobs:**
- Secret detection (API keys, tokens, passwords)
- Code security analysis
- Dependency vulnerability scan
- Configuration security check
- CORS policy validation
- Authentication security

**Schedule adjustment:**
```yaml
# Change frequency in security-scan.yml
schedule:
  - cron: '0 0 * * 0'  # Weekly instead of daily
```

### Maintenance Workflows

#### `dependency-update.yml`
**Purpose:** Proactive dependency management.
**Trigger:** Weekly (Monday 9 AM), manual
**Jobs:**
- Check outdated packages
- Detect vulnerabilities
- Identify deprecated packages
- Auto-create GitHub issues
- Generate reports

**Automation:**
- Creates issues with labels: `security`, `dependencies`, `tech-debt`
- Uploads reports as artifacts
- Fails fast on critical vulnerabilities

### Deployment Workflows

#### `cicd-pipeline.yml`
**Purpose:** Complete CI/CD for staging and production.
**Trigger:** Push to main, version tags, manual dispatch
**Jobs:**
1. Build & Package
2. Code Quality Analysis
3. Security Gate
4. Deploy to Staging
5. Deploy to Production
6. Notification

**Configuration required:**
```yaml
# Update deployment commands for your platform:
- name: Deploy to staging
  run: |
    # Azure Web App
    az webapp deploy --name myapp-staging ...
    
    # Azure Functions
    func azure functionapp publish myapp-staging ...
    
    # Docker/Kubernetes
    docker build -t myapp:${{ needs.build.outputs.version }} .
    kubectl apply -f k8s/staging/
```

## Integration with Claude Code

### Local → Remote Workflow

```
┌─────────────────────────────────────────────┐
│ Developer (Local)                           │
│                                             │
│ 1. Write code with Copilot                 │
│    (follows copilot-instructions.md)       │
│                                             │
│ 2. Claude Code: "Run pre-pr-compliance"    │
│    ✅ All standards pass                    │
│                                             │
│ 3. git commit && git push                  │
└─────────────────┬───────────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────────┐
│ GitHub (Remote)                             │
│                                             │
│ 1. PR created with template                │
│                                             │
│ 2. code-review-pr.yml triggers             │
│    ✅ All automated checks pass             │
│                                             │
│ 3. security-scan.yml validates             │
│    ✅ No vulnerabilities                    │
│                                             │
│ 4. Human review + merge                    │
│                                             │
│ 5. cicd-pipeline.yml deploys               │
│    ✅ Staging → Production                  │
└─────────────────────────────────────────────┘
```

### Standards Alignment

Both Claude Code and GitHub workflows enforce the **same standards** from `docs/standards/`:

| Standard | Claude Code Skill | GitHub Workflow Job |
|----------|-------------------|---------------------|
| Async/await | `async-await-enforcer` | `async-check` |
| Naming | `naming-convention-enforcer` | `naming-check` |
| Exceptions | `exception-handling-enforcer` | `exception-check` |
| Configuration | `configuration-pattern-enforcer` | `configuration-check` |
| Packages | `package-governance` | `package-security` |
| CQRS / Mediator | `cqrs-pattern-enforcer` | (local only) |
| Repository & Unit of Work | `repository-pattern-enforcer` | (local only) |
| Dependency injection | `dependency-injection-enforcer` | (local only) |
| Architecture | `architecture-reviewer` | (local only) |
| Testing | `unit-test-generator` | `build-verification` |

## Templates

### Pull Request Template

**File:** `pull_request_template.md`

**Purpose:** Ensures every PR addresses engineering standards.

**Checklist maps to:**
- Project type detection
- Standards from `docs/standards/`
- Build verification requirements

**Usage:** Automatically populated when creating PR.

### Issue Templates

**Bug Report** — `ISSUE_TEMPLATE/bug_report.md`
- Environment details
- Reproduction steps
- Expected vs actual behavior
- Project type context

**Feature Request** — `ISSUE_TEMPLATE/feature_request.md`
- User story format
- Acceptance criteria
- Standards considerations

**Tech Debt** — `ISSUE_TEMPLATE/tech_debt.md`
- Current state
- Desired state
- Affected standards
- Effort estimation

## Copilot Instructions

**File:** `copilot-instructions.md`

**Purpose:** Align GitHub Copilot with Claude Code framework standards.

**Key Features:**
- References same `docs/standards/` as Claude Code
- Project type detection before suggestions
- Async-first, validated, observable patterns
- No secrets, proper configuration patterns

**Automatic Loading:** GitHub Copilot reads this file automatically in the repository.

## Monitoring & Observability

### GitHub Actions Dashboard

**View workflow runs:**
```bash
gh run list
gh run view <run-id>
gh run view <run-id> --log
```

**Monitor specific workflow:**
```bash
gh run list --workflow=code-review-pr.yml
gh run list --workflow=security-scan.yml
```

### Automated Issue Creation

**dependency-update.yml creates issues for:**
- Vulnerable packages (label: `security`)
- Deprecated packages (label: `dependencies`)
- Outdated packages (label: `tech-debt`)

**View:**
```bash
gh issue list --label security
gh issue list --label dependencies
```

### Workflow Summaries

Every workflow writes to `$GITHUB_STEP_SUMMARY`:
- ✅ Passed checks
- ⚠️ Warnings
- ❌ Failures with links to standards docs

**View in GitHub:** Actions → Select run → Summary tab

## Troubleshooting

### Workflow Fails on First Enable

**Cause:** Repository not yet compliant.

**Solution:**
1. Run Claude Code `code-review` workflow locally first
2. Fix all standards violations
3. Enable GitHub workflows after compliance

### False Positives

**Example:** Security scan flags valid pattern.

**Solution:**
```yaml
# Add exclusion pattern in workflow
if grep -r "secret" | grep -v "// Allowed: explanation"; then
  echo "::error::Secret detected"
fi
```

**Or in code:**
```cs
// Security scan exception: Development storage only
var connectionString = "UseDevelopmentStorage=true";
```

### Workflow Performance

**Problem:** Workflows taking too long.

**Optimize:**
```yaml
# Add NuGet package caching
- name: Cache packages
  uses: actions/cache@v4
  with:
    path: ~/.nuget/packages
    key: ${{ runner.os }}-nuget-${{ hashFiles('**/packages.lock.json') }}

# Parallelize independent jobs (remove unnecessary 'needs')
```

### Deployment Failures

**Check:**
1. Environment configuration (Settings → Environments)
2. Deployment secrets (Settings → Secrets)
3. Workflow logs (`gh run view <run-id> --log`)
4. Smoke test endpoints

## Customization

### Add Custom Check to PR Workflow

```yaml
# In code-review-pr.yml
custom-check:
  name: Custom Standard
  runs-on: ubuntu-latest
  needs: project-detection
  steps:
    - uses: actions/checkout@v4
    - name: Run custom check
      run: |
        # Your check logic
        echo "Running custom check..."
```

### Add New Workflow

```yaml
# .github/workflows/my-workflow.yml
name: My Workflow
on:
  workflow_dispatch: {}

permissions:
  contents: read

jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - # Your steps
```

### Modify Deployment Targets

```yaml
# In cicd-pipeline.yml
environments:
  - staging
  - qa          # Add QA environment
  - production
```

## Best Practices

### Do's

✅ Test workflows manually first (`workflow_dispatch`)
✅ Enable progressively (manual → PR → auto)
✅ Monitor workflow summaries regularly
✅ Keep standards docs synchronized
✅ Use branch protection rules
✅ Set up environment approvals
✅ Review all workflow outputs

### Don'ts

❌ Don't enable all workflows immediately
❌ Don't bypass failing checks
❌ Don't commit secrets to workflows
❌ Don't ignore warnings (they become errors)
❌ Don't modify standards without updating docs
❌ Don't disable security scans
❌ Don't skip local checks before pushing

## References

- **[Workflow Architecture](./WORKFLOW-ARCHITECTURE.md)** — Complete technical architecture
- **[Orchestration Guide](./ORCHESTRATION-GUIDE.md)** — Operational procedures
- **[Framework README](../FRAMEWORK-README.md)** — Framework overview
- **[Standards Documentation](../docs/standards/)** — Engineering standards
- **[GitHub Actions Docs](https://docs.github.com/en/actions)** — Official documentation

## Support

**For issues:**
1. Check [ORCHESTRATION-GUIDE.md](./ORCHESTRATION-GUIDE.md) troubleshooting section
2. Review workflow logs: `gh run view <run-id> --log`
3. Run locally with Claude Code first
4. Check [WORKFLOW-ARCHITECTURE.md](./WORKFLOW-ARCHITECTURE.md) for integration details

**For framework updates:**
- All standards live in `docs/standards/` (single source of truth)
- Update workflows to match standard changes
- Test locally before deploying workflow changes
