# Workflow Orchestration Guide

Complete guide for implementing and operating the Claude Code Agentic Workflow Framework with GitHub Actions.

## Quick Start

### For New Repositories

```bash
# 1. Copy framework to your repo
cp -r .claude .github .mcp docs .mcp.json /path/to/your/repo/

# 2. Initialize MCP (optional but recommended)
cd /path/to/your/repo
cp .mcp/configurations/.env.example .mcp/configurations/.env
# Edit .env with your workspace root

# 3. Open in Claude Code and run initial assessment
# Prompt: "Read .claude/instructions/engineering-standards.md, then run the repo-onboarding workflow on this solution."

# 4. Enable GitHub workflows gradually
# Start with manual dispatch, then enable PR triggers
```

### For Existing Repositories

```bash
# 1. Assess current state
# Prompt: "Run the standards-auditor on this solution and give me a compliance report."

# 2. Run code review workflow
# Prompt: "Review my code using the code-review workflow. Run step by step with build gates."

# 3. After fixing issues, enable GitHub workflows
git commit -am "Enable Claude Code workflow framework"
git push
```

## Workflow Decision Tree

```
User Action
    │
    ├─ Local Development
    │  │
    │  ├─ New Feature/Fix
    │  │  └─→ [Write code] → [Run pre-pr-compliance] → [Commit] → [Create PR]
    │  │
    │  ├─ Major Refactor
    │  │  └─→ [Run code-review workflow] → [Fix step-by-step] → [Commit] → [Create PR]
    │  │
    │  └─ New Repo
    │     └─→ [Run repo-onboarding] → [Review reports] → [Fix issues] → [Enable workflows]
    │
    ├─ Pull Request Created
    │  │
    │  └─→ [code-review-pr.yml] → [security-scan.yml]
    │      │
    │      ├─ Pass → [Merge]
    │      └─ Fail → [Fix locally] → [Push] → [Re-run]
    │
    ├─ Scheduled Maintenance
    │  │
    │  ├─ Monday 9 AM
    │  │  └─→ [dependency-update.yml] → [Creates issues if needed]
    │  │
    │  └─ Daily Midnight
    │     └─→ [security-scan.yml] → [Report/issue if needed]
    │
    └─ On-Demand
       │
       └─→ [standards-ci.yml] → [Manual trigger] → [Full compliance check]
```

## Workflow Execution Patterns

### Pattern 1: Feature Development

**Scenario:** Developer implements a new feature.

**Steps:**

1. **Local Development**
   ```
   # Write feature code
   # Run Copilot suggestions (follows .github/copilot-instructions.md)
   ```

2. **Pre-Commit Check**
   ```
   Claude Code prompt:
   "Run pre-pr-compliance on my changes"
   
   Output:
   - Async violations: 0
   - Naming issues: 0
   - Build: PASS
   - Tests: PASS
   ```

3. **Commit & PR**
   ```bash
   git commit -am "Add feature X"
   git push origin feature/x
   # Create PR using template
   ```

4. **Automated Validation**
   - `code-review-pr.yml` runs automatically
   - All checks pass ✅
   - PR ready for human review

### Pattern 2: Standards Remediation

**Scenario:** Existing codebase needs standards enforcement.

**Steps:**

1. **Initial Assessment**
   ```
   Claude Code prompt:
   "Run repo-onboarding workflow"
   
   Output:
   - README generated
   - Architecture report
   - Standards compliance: 60%
   - Top issues: async violations, package vulnerabilities
   ```

2. **Step-by-Step Remediation**
   ```
   Claude Code prompt:
   "Run code-review workflow step-by-step with build gates"
   
   Process:
   Step 1: Baseline build → PASS
   Step 2: Fix async → 15 fixes → Build → PASS
   Step 3: Fix naming → 8 fixes → Build → PASS
   ...
   Step 11: Final verification → BUILD PASS, TESTS PASS
   ```

3. **Enable Workflows**
   ```bash
   # Uncomment triggers in standards-ci.yml
   vim .github/workflows/standards-ci.yml
   # Change: continue-on-error: true → false
   git commit -am "Enable standards enforcement"
   ```

### Pattern 3: Security Response

**Scenario:** Vulnerability discovered in dependency.

**Steps:**

1. **Automated Detection**
   ```
   # dependency-update.yml runs Monday 9 AM
   # Detects: Newtonsoft.Json 12.0.1 (CVE-2024-XXXX)
   # Creates issue automatically with label: security
   ```

2. **Local Fix**
   ```
   Claude Code prompt:
   "Check package vulnerabilities and update to safe versions"
   
   Output:
   - Updated Newtonsoft.Json → 13.0.3
   - Tested: all tests pass
   - CPM updated
   ```

3. **Verify**
   ```bash
   git commit -am "Security: Update Newtonsoft.Json to 13.0.3"
   git push
   # security-scan.yml validates no vulnerabilities remain
   ```

### Pattern 4: Continuous Compliance

**Scenario:** Maintain standards across team over time.

**Setup:**

1. **GitHub Branch Protection**
   ```
   Settings → Branches → main
   ✅ Require status checks to pass
   ✅ code-review-pr / build-verification
   ✅ security-scan / dependency-scan
   ```

2. **Team Training**
   ```
   - Claude Code workflows guide
   - PR template checklist review
   - Standards documentation
   ```

3. **Monitoring**
   ```
   # Weekly review:
   - Check dependency-update issues
   - Review security-scan summaries
   - Track standards compliance trend
   ```

## Workflow Configuration

### Trigger Customization

**Code Review PR** — Adjust branches:
```yaml
on:
  pull_request:
    branches: [main, develop, release/*]  # Add release branches
```

**Security Scan** — Change schedule:
```yaml
schedule:
  - cron: '0 0 * * *'  # Daily at midnight
  # Change to: '0 0 * * 0'  # Weekly on Sunday
```

**Dependency Update** — Adjust frequency:
```yaml
schedule:
  - cron: '0 9 * * 1'  # Monday 9 AM
  # Change to: '0 9 1 * *'  # First day of month
```

### Job Customization

**Add .NET Version:**
```yaml
- name: Setup .NET
  uses: actions/setup-dotnet@v4
  with:
    dotnet-version: |
      6.0.x
      7.0.x
      8.0.x
      9.0.x
```

**Skip Tests Temporarily:**
```yaml
- name: Test
  if: false  # Disable temporarily
  run: dotnet test
```

**Add Custom Check:**
```yaml
custom-check:
  name: Custom Standard
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Run custom check
      run: |
        # Your custom logic
```

### Permission Management

**Minimal Permissions:**
```yaml
permissions:
  contents: read
```

**Write Permissions (for auto-fix PRs):**
```yaml
permissions:
  contents: write
  pull-requests: write
```

**Security Scanning:**
```yaml
permissions:
  contents: read
  security-events: write
```

## Integration with Claude Code

### Workflow Invocation from Claude Code

**Skills automatically reference GitHub workflows:**

```markdown
<!-- In .claude/skills/*/SKILL.md -->

After local fixes, remind the user:
"Push changes to trigger GitHub code-review-pr workflow for validation."
```

**Workflows reference skills:**

```yaml
# In .github/workflows/code-review-pr.yml
- name: Async check
  run: |
    # Same checks as async-await-enforcer skill
    grep -r "\.Result" --include="*.cs"
```

### Status Feedback Loop

```
┌─────────────────────────────────────────────┐
│  Claude Code (Local)                        │
│  ├─ Run pre-pr-compliance                   │
│  └─ Output: All checks pass                 │
└─────────────────┬───────────────────────────┘
                  │ Commit & Push
                  ↓
┌─────────────────────────────────────────────┐
│  GitHub Actions (Remote)                    │
│  ├─ code-review-pr.yml                      │
│  ├─ security-scan.yml                       │
│  └─ Output: All checks pass ✅              │
└─────────────────┬───────────────────────────┘
                  │ Status → PR
                  ↓
┌─────────────────────────────────────────────┐
│  Developer Reviews                          │
│  ├─ Local checks: PASS                      │
│  ├─ Remote checks: PASS                     │
│  └─ Decision: Merge                         │
└─────────────────────────────────────────────┘
```

## Best Practices

### Do's

✅ **Run local checks first** — Fix issues before pushing
✅ **Review PR template** — Check all boxes before requesting review
✅ **Monitor workflow summaries** — Check GitHub Actions tab regularly
✅ **Address security issues immediately** — Vulnerable packages have priority
✅ **Keep standards docs updated** — Single source of truth
✅ **Use manual dispatch first** — Test workflows before enabling auto-triggers
✅ **Commit Claude Code changes manually** — Review every change before commit

### Don'ts

❌ **Don't skip local checks** — Don't rely only on GitHub workflows
❌ **Don't ignore warnings** — Warnings become errors in production
❌ **Don't commit secrets** — Use User Secrets and Key Vault
❌ **Don't disable security checks** — Even temporarily
❌ **Don't merge with failing workflows** — Fix or revert
❌ **Don't let Claude Code auto-commit** — Always review first
❌ **Don't bypass PR template** — It maps to enforced standards

## Troubleshooting

### Workflow Runs But No Results

**Check:**
```bash
# Verify workflow file syntax
yamllint .github/workflows/*.yml

# Check workflow runs
gh run list --workflow=code-review-pr.yml

# View specific run
gh run view <run-id>
```

### Standards Check Fails on Valid Code

**Scenario:** Regex pattern over-matches.

**Fix:**
```yaml
# In workflow, refine pattern
if grep -r "\.Result" --include="*.cs" | grep -v "TestResult"; then
  # Exclude test files from check
fi
```

### Merge Blocked by Workflow

**Option 1: Fix the issue**
```bash
# Run locally
Claude Code: "Run pre-pr-compliance and fix issues"
git commit -am "Fix standards violations"
git push
```

**Option 2: Override (emergency only)**
```bash
# Requires admin permissions
gh pr merge --admin --merge
```

### Workflow Too Slow

**Optimize:**
```yaml
# Add caching
- name: Cache NuGet
  uses: actions/cache@v4
  with:
    path: ~/.nuget/packages
    key: ${{ runner.os }}-nuget-${{ hashFiles('**/packages.lock.json') }}

# Parallelize jobs (remove `needs` dependencies where possible)
```

### False Positive Security Alerts

**Document exceptions:**
```cs
// Allowed: Development storage connection
// Security scan exception: UseDevelopmentStorage=true
var connectionString = "UseDevelopmentStorage=true";
```

**Update workflow pattern:**
```yaml
if grep "password" | grep -v "UseDevelopmentStorage"; then
  # Exclude known safe patterns
fi
```

## Advanced Scenarios

### Multi-Project Solution

**Project detection:**
```yaml
- name: Detect project types
  run: |
    for proj in $(find . -name "*.csproj"); do
      echo "Analyzing $proj"
      # Type detection logic
    done
```

**Selective building:**
```yaml
- name: Build specific projects
  run: |
    dotnet build src/ProjectA/*.csproj
    dotnet build src/ProjectB/*.csproj
```

### Monorepo with Multiple Solutions

**Path filtering:**
```yaml
on:
  pull_request:
    paths:
      - 'services/auth/**'
      - 'services/api/**'
```

**Matrix strategy:**
```yaml
strategy:
  matrix:
    solution:
      - services/auth
      - services/api
steps:
  - run: dotnet build ${{ matrix.solution }}
```

### Integration with External Systems

**Notify Slack on failure:**
```yaml
- name: Notify Slack
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "Standards CI failed: ${{ github.event.pull_request.html_url }}"
      }
```

**Update external dashboard:**
```yaml
- name: Report metrics
  run: |
    curl -X POST https://metrics.example.com/api/standards \
      -d "{\"repo\": \"${{ github.repository }}\", \"status\": \"pass\"}"
```

## Success Metrics

Track these metrics to measure effectiveness:

### Leading Indicators
- **Pre-commit fix rate** — Issues caught by Claude Code locally
- **First-run pass rate** — PRs passing GitHub workflows on first attempt
- **Review cycle time** — Time from PR creation to merge

### Lagging Indicators
- **Production defect rate** — Issues escaping to production
- **Security incident rate** — Vulnerabilities reaching production
- **Technical debt trend** — Standards compliance over time

### Operational Metrics
- **Workflow execution time** — Average time per workflow run
- **False positive rate** — Invalid alerts per 100 runs
- **Team adoption rate** — % of PRs using framework

## References

- [Workflow Architecture](./WORKFLOW-ARCHITECTURE.md)
- [Framework README](../FRAMEWORK-README.md)
- [Standards Documentation](../docs/standards/)
- [GitHub Actions Best Practices](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

## Support

For issues or questions:

1. **Check documentation** — Review standards in `docs/standards/`
2. **Review workflow runs** — Check GitHub Actions tab for details
3. **Run locally first** — Use Claude Code workflows to debug
4. **Check framework README** — Quick-start and verify sections
