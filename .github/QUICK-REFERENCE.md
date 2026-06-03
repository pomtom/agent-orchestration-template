# GitHub Workflows Quick Reference

Quick reference for all GitHub Actions workflows in the Claude Code Framework.

## Workflow Summary

| Workflow | Trigger | Purpose | Duration |
|----------|---------|---------|----------|
| `standards-ci.yml` | Manual (opt-in) | Core standards enforcement | ~5 min |
| `code-review-pr.yml` | Every PR | Automated PR validation | ~8 min |
| `security-scan.yml` | Push/PR/Daily | Security scanning | ~6 min |
| `dependency-update.yml` | Weekly Mon 9AM | Dependency management | ~3 min |
| `cicd-pipeline.yml` | Push main/Tags | Full CI/CD deployment | ~15 min |

## Common Commands

### Trigger Workflows Manually

```bash
# Standards CI
gh workflow run standards-ci.yml

# Security scan
gh workflow run security-scan.yml

# Dependency update check
gh workflow run dependency-update.yml

# CI/CD to specific environment
gh workflow run cicd-pipeline.yml -f environment=staging
```

### View Workflow Status

```bash
# List all runs
gh run list

# View specific workflow runs
gh run list --workflow=code-review-pr.yml

# View run details
gh run view <run-id>

# View run logs
gh run view <run-id> --log

# Watch run in progress
gh run watch <run-id>
```

### Check Workflow Syntax

```bash
# Validate YAML
yamllint .github/workflows/*.yml

# Check with GitHub API
gh workflow list
```

## Workflow Jobs Quick Reference

### standards-ci.yml

```yaml
✅ build-and-test
   ├─ Setup .NET (8.x, 9.x)
   ├─ Restore packages
   ├─ Build (Release)
   └─ Test

✅ standards-check (continue-on-error: true)
   ├─ Format check (dotnet format --verify-no-changes)
   ├─ Vulnerable packages (BLOCKING if enabled)
   ├─ Deprecated packages (report only)
   └─ Outdated packages (report only)
```

### code-review-pr.yml

```yaml
✅ project-detection
   └─ Detect .NET project types

✅ async-check (needs: project-detection)
   ├─ Check for .Result usage
   ├─ Check for .Wait() usage
   └─ Check for async void

✅ naming-check (needs: project-detection)
   └─ Verify naming conventions

✅ exception-check (needs: project-detection)
   ├─ Check for "throw ex;" anti-pattern
   └─ Check for empty catch blocks

✅ configuration-check (needs: project-detection)
   └─ Check for hardcoded connection strings

✅ package-security (needs: project-detection)
   ├─ Vulnerable packages scan
   └─ Deprecated packages check

✅ build-verification (needs: all checks)
   ├─ Restore → Build → Test
   └─ Upload test results

✅ review-summary (needs: build-verification)
   └─ Generate summary
```

### security-scan.yml

```yaml
✅ secret-scan
   ├─ Check for API keys
   ├─ Check for connection strings
   └─ Check for JWT secrets

✅ code-analysis
   ├─ Security analyzers
   └─ SQL injection checks

✅ dependency-scan
   └─ Vulnerable packages (BLOCKING)

✅ configuration-security
   ├─ Check appsettings.json
   └─ Verify no secrets files

✅ cors-security
   └─ Check for AllowAnyOrigin()

✅ authentication-check
   ├─ Review anonymous access
   └─ JWT configuration validation

✅ summary (needs: all)
   └─ Security report
```

### dependency-update.yml

```yaml
✅ check-updates
   ├─ Check outdated packages
   ├─ Check vulnerable packages
   ├─ Check deprecated packages
   ├─ Create issue if vulnerabilities found
   └─ Upload reports as artifacts
```

### cicd-pipeline.yml

```yaml
✅ build
   ├─ Determine version
   ├─ Build solution
   ├─ Run tests + coverage
   └─ Publish artifacts

✅ code-quality
   ├─ Code analysis
   └─ Format check

✅ security-gate
   ├─ Vulnerability scan (BLOCKING)
   └─ Secrets check (BLOCKING)

✅ deploy-staging (needs: all gates, if: main branch)
   ├─ Deploy to staging environment
   └─ Run smoke tests

✅ deploy-production (needs: deploy-staging, if: version tag)
   ├─ Deploy to production environment
   ├─ Run smoke tests
   └─ Create GitHub release

✅ notify (needs: all deploys)
   └─ Deployment summary
```

## Exit Codes

| Code | Meaning | Example |
|------|---------|---------|
| 0 | Success | All checks passed |
| 1 | Failure | Vulnerable packages, hardcoded secrets |
| Continue | Non-blocking | Deprecated packages (warning only) |

## Status Checks for Branch Protection

Required status checks for `main` branch:

```
✅ code-review-pr / project-detection
✅ code-review-pr / build-verification
✅ security-scan / secret-scan
✅ security-scan / dependency-scan
✅ security-scan / configuration-security
```

Optional (recommended):
```
✅ code-review-pr / async-check
✅ code-review-pr / naming-check
✅ code-review-pr / exception-check
✅ standards-ci / standards-check
```

## Workflow Outputs

### Artifacts Generated

| Workflow | Artifact | Content |
|----------|----------|---------|
| `code-review-pr.yml` | `test-results` | `.trx` test result files |
| `dependency-update.yml` | `dependency-reports` | Outdated/vulnerable/deprecated package lists |
| `cicd-pipeline.yml` | `publish` | Published application binaries |
| `cicd-pipeline.yml` | `coverage` | Code coverage reports |

**Download artifacts:**
```bash
gh run download <run-id>
gh run download <run-id> --name test-results
```

### Issues Created

| Workflow | Trigger | Labels |
|----------|---------|--------|
| `dependency-update.yml` | Vulnerable packages found | `security`, `dependencies`, `tech-debt` |

**Query:**
```bash
gh issue list --label security
gh issue list --label dependencies,tech-debt
```

## Environment Configuration

### Secrets Required

**For CI/CD Pipeline:**
```
# Required
AZURE_CREDENTIALS          # Azure service principal (if deploying to Azure)
AZURE_WEBAPP_NAME          # Target web app name
AZURE_FUNCTIONAPP_NAME     # Target function app name

# Optional
SLACK_WEBHOOK_URL          # Slack notifications
TEAMS_WEBHOOK_URL          # Teams notifications
NUGET_API_KEY              # Package publishing
```

**Set secrets:**
```bash
gh secret set AZURE_CREDENTIALS < azure-creds.json
gh secret set SLACK_WEBHOOK_URL --body "https://hooks.slack.com/..."
```

### Environment Protection Rules

**Staging:**
```
Required reviewers: 0
Wait timer: 0 minutes
Deployment branches: main
```

**Production:**
```
Required reviewers: 2+ (team leads)
Wait timer: 10 minutes
Deployment branches: main, tags matching v*.*.*
```

## Troubleshooting Quick Fixes

### Workflow Syntax Error

```bash
# Validate locally
yamllint .github/workflows/my-workflow.yml

# Check with GitHub
gh workflow view my-workflow.yml
```

### Job Failure

```bash
# Get logs
gh run view <run-id> --log

# Re-run failed jobs only
gh run rerun <run-id> --failed

# Re-run all jobs
gh run rerun <run-id>
```

### Blocked by Status Check

```bash
# Check which checks are failing
gh pr checks

# View specific check logs
gh pr checks --watch

# Override (admin only, emergency)
gh pr merge --admin --merge <pr-number>
```

### Slow Workflow

```yaml
# Add caching to restore step
- name: Cache NuGet
  uses: actions/cache@v4
  with:
    path: ~/.nuget/packages
    key: ${{ runner.os }}-nuget-${{ hashFiles('**/packages.lock.json') }}

# Parallelize jobs (remove unnecessary 'needs:' dependencies)
```

### False Positive

```bash
# Option 1: Fix in code with comment
// Security scan exception: Development storage only
var conn = "UseDevelopmentStorage=true";

# Option 2: Adjust workflow regex
if grep "secret" | grep -v "// Security scan exception"; then
  exit 1
fi
```

## Integration with Local Development

### Pre-Push Checklist

```bash
# 1. Run Claude Code pre-PR compliance
Claude Code: "Run pre-pr-compliance workflow"

# 2. Verify build locally
dotnet build --configuration Release
dotnet test --configuration Release

# 3. Check for secrets
git diff | grep -i "password\|secret\|key"

# 4. Check for vulnerable packages
dotnet list package --vulnerable

# 5. Push
git push origin feature/my-feature
```

### After Push

```bash
# Monitor PR checks
gh pr checks --watch

# View workflow run
gh run list --workflow=code-review-pr.yml --limit 1
gh run view <run-id>
```

## Workflow Modification Templates

### Add New Check to PR Workflow

```yaml
# In .github/workflows/code-review-pr.yml
my-new-check:
  name: My New Standard Check
  runs-on: ubuntu-latest
  needs: project-detection
  steps:
    - uses: actions/checkout@v4
    - name: Run my check
      run: |
        if grep -r "bad-pattern" --include="*.cs"; then
          echo "::error::Bad pattern found"
          exit 1
        fi
```

### Add Notification

```yaml
# At end of workflow
- name: Notify on failure
  if: failure()
  run: |
    curl -X POST ${{ secrets.SLACK_WEBHOOK_URL }} \
      -H 'Content-Type: application/json' \
      -d '{"text":"Workflow failed: ${{ github.event.pull_request.html_url }}"}'
```

### Add Matrix Build

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    dotnet: [6.0.x, 8.0.x, 9.0.x]
runs-on: ${{ matrix.os }}
steps:
  - uses: actions/setup-dotnet@v4
    with:
      dotnet-version: ${{ matrix.dotnet }}
```

## Performance Benchmarks

Expected workflow execution times:

| Workflow | Small Repo | Medium Repo | Large Repo |
|----------|-----------|-------------|-----------|
| `code-review-pr.yml` | 3-5 min | 5-8 min | 8-12 min |
| `security-scan.yml` | 2-4 min | 4-6 min | 6-10 min |
| `standards-ci.yml` | 3-5 min | 5-10 min | 10-15 min |
| `dependency-update.yml` | 1-2 min | 2-3 min | 3-5 min |
| `cicd-pipeline.yml` | 8-12 min | 12-18 min | 18-30 min |

*Small: <10 projects, Medium: 10-30 projects, Large: 30+ projects*

## Resources

- **[Workflow Architecture](./WORKFLOW-ARCHITECTURE.md)** — Technical deep dive
- **[Orchestration Guide](./ORCHESTRATION-GUIDE.md)** — Step-by-step operations
- **[README](.github/README.md)** — Overview and setup
- **[Standards](../docs/standards/)** — Engineering standards reference

## Quick Links

```bash
# View all workflows
gh workflow list

# View workflow runs
gh run list

# View PR checks
gh pr checks

# View issues
gh issue list

# View secrets
gh secret list
```
