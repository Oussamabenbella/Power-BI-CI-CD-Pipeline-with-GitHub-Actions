# Advanced Workflows

This guide covers advanced GitHub Actions workflows for Power BI CI/CD pipelines.

## Table of Contents

1. [Automated Deployment](#automated-deployment)
2. [Multi-Environment Pipelines](#multi-environment-pipelines)
3. [Scheduled BPA Checks](#scheduled-bpa-checks)
4. [Pull Request Validation](#pull-request-validation)
5. [Notifications and Reporting](#notifications-and-reporting)

## Automated Deployment

### Deploy to Power BI Service on Merge

```yaml
name: Deploy to Power BI Service

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: windows-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Install Power BI PowerShell Module
        run: |
          Install-Module -Name MicrosoftPowerBIMgmt -Force -Scope CurrentUser
        shell: pwsh
      
      - name: Authenticate to Power BI
        run: |
          $password = ConvertTo-SecureString "${{ secrets.PBI_PASSWORD }}" -AsPlainText -Force
          $credential = New-Object System.Management.Automation.PSCredential("${{ secrets.PBI_USERNAME }}", $password)
          Connect-PowerBIServiceAccount -Credential $credential
        shell: pwsh
      
      - name: Deploy Report
        run: |
          # Your deployment script here
          New-PowerBIReport -Path "path/to/report.pbix" -WorkspaceId "${{ secrets.WORKSPACE_ID }}"
        shell: pwsh
```

## Multi-Environment Pipelines

### Dev → Test → Prod Pipeline

```yaml
name: Multi-Environment Deployment

on:
  push:
    branches:
      - develop
      - main

jobs:
  validate:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run BPA
        run: |
          # BPA validation steps
  
  deploy-dev:
    needs: validate
    if: github.ref == 'refs/heads/develop'
    runs-on: windows-latest
    environment: development
    steps:
      - name: Deploy to Dev
        run: |
          # Deploy to dev workspace
  
  deploy-prod:
    needs: validate
    if: github.ref == 'refs/heads/main'
    runs-on: windows-latest
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          # Deploy to prod workspace
```

## Scheduled BPA Checks

### Daily BPA Audit

```yaml
name: Daily BPA Audit

on:
  schedule:
    - cron: '0 9 * * 1-5'  # 9 AM weekdays
  workflow_dispatch:

jobs:
  audit:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download Tabular Editor
        run: |
          $url = "https://github.com/TabularEditor/TabularEditor/releases/latest/download/TabularEditor.Portable.zip"
          Invoke-WebRequest -Uri $url -OutFile TabularEditor.zip
          Expand-Archive -Path TabularEditor.zip -DestinationPath . -Force
        shell: pwsh
      
      - name: Run Comprehensive BPA
        run: |
          ./TabularEditor.exe "*.SemanticModel/definition/database.tmdl" -A "https://raw.githubusercontent.com/microsoft/Analysis-Services/master/BestPracticeRules/BPARules.json" `
          | Tee-Object -FilePath "./audit_$(Get-Date -Format 'yyyy-MM-dd').txt"
        shell: pwsh
      
      - name: Upload Audit Results
        uses: actions/upload-artifact@v4
        with:
          name: daily-audit-${{ github.run_number }}
          path: audit_*.txt
          retention-days: 90
```

## Pull Request Validation

### Comprehensive PR Checks

```yaml
name: PR Validation

on:
  pull_request:
    branches:
      - main
      - develop

jobs:
  validate-changes:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Get Changed Files
        id: changed-files
        run: |
          $files = git diff --name-only origin/${{ github.base_ref }}...HEAD
          echo "files=$files" >> $env:GITHUB_OUTPUT
        shell: pwsh
      
      - name: Download Tabular Editor
        run: |
          $url = "https://github.com/TabularEditor/TabularEditor/releases/latest/download/TabularEditor.Portable.zip"
          Invoke-WebRequest -Uri $url -OutFile TabularEditor.zip
          Expand-Archive -Path TabularEditor.zip -DestinationPath . -Force
        shell: pwsh
      
      - name: Run BPA on Changed Models
        run: |
          ./TabularEditor.exe "*.SemanticModel/definition/database.tmdl" -A "./BPA_Rule.json" `
          | Tee-Object -FilePath "./pr_validation.txt"
        shell: pwsh
      
      - name: Comment PR with Results
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const results = fs.readFileSync('./pr_validation.txt', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## BPA Validation Results\n\`\`\`\n${results}\n\`\`\``
            });
      
      - name: Fail if Errors Found
        run: |
          $content = Get-Content "./pr_validation.txt"
          if ($content -match "Error") {
            Write-Error "BPA found errors. Please fix before merging."
            exit 1
          }
        shell: pwsh
```

## Notifications and Reporting

### Slack Notifications

```yaml
name: BPA with Slack Notification

on:
  push:
    branches:
      - main

jobs:
  bpa-check:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run BPA
        id: bpa
        continue-on-error: true
        run: |
          # BPA steps
      
      - name: Send Slack Notification
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "BPA Check ${{ steps.bpa.outcome }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*BPA Check Result:* ${{ steps.bpa.outcome }}\n*Repository:* ${{ github.repository }}\n*Branch:* ${{ github.ref_name }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Email Reports

```yaml
name: Weekly BPA Report

on:
  schedule:
    - cron: '0 8 * * 1'  # Monday 8 AM

jobs:
  weekly-report:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run BPA
        run: |
          # BPA steps
      
      - name: Generate HTML Report
        run: |
          $results = Get-Content "./BPA_output.txt"
          $html = @"
          <html>
          <head><title>Weekly BPA Report</title></head>
          <body>
            <h1>Power BI BPA Report</h1>
            <pre>$results</pre>
          </body>
          </html>
          "@
          $html | Out-File -FilePath "./report.html"
        shell: pwsh
      
      - name: Send Email
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: smtp.gmail.com
          server_port: 465
          username: ${{ secrets.EMAIL_USERNAME }}
          password: ${{ secrets.EMAIL_PASSWORD }}
          subject: Weekly Power BI BPA Report
          to: team@example.com
          from: github-actions@example.com
          html_body: file://report.html
          attachments: BPA_output.txt
```

## Matrix Builds

### Test Multiple Rulesets

```yaml
name: Multi-Ruleset Validation

on:
  workflow_dispatch:

jobs:
  validate:
    runs-on: windows-latest
    strategy:
      matrix:
        ruleset:
          - name: "Standard"
            file: "https://raw.githubusercontent.com/microsoft/Analysis-Services/master/BestPracticeRules/BPARules.json"
          - name: "Custom"
            file: "./BPA_Rule.json"
          - name: "Strict"
            file: "./BPA_Strict.json"
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download Tabular Editor
        run: |
          # Download steps
      
      - name: Run BPA - ${{ matrix.ruleset.name }}
        run: |
          ./TabularEditor.exe "*.SemanticModel/definition/database.tmdl" -A "${{ matrix.ruleset.file }}" `
          | Tee-Object -FilePath "./BPA_${{ matrix.ruleset.name }}.txt"
        shell: pwsh
      
      - name: Upload Results
        uses: actions/upload-artifact@v4
        with:
          name: bpa-results-${{ matrix.ruleset.name }}
          path: BPA_${{ matrix.ruleset.name }}.txt
```

## Caching for Performance

### Cache Tabular Editor

```yaml
name: BPA with Caching

on:
  push:
    branches:
      - main

jobs:
  bpa-check:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Cache Tabular Editor
        id: cache-te
        uses: actions/cache@v3
        with:
          path: |
            TabularEditor.exe
            *.dll
          key: tabular-editor-${{ runner.os }}-v2
      
      - name: Download Tabular Editor
        if: steps.cache-te.outputs.cache-hit != 'true'
        run: |
          $url = "https://github.com/TabularEditor/TabularEditor/releases/latest/download/TabularEditor.Portable.zip"
          Invoke-WebRequest -Uri $url -OutFile TabularEditor.zip
          Expand-Archive -Path TabularEditor.zip -DestinationPath . -Force
        shell: pwsh
      
      - name: Run BPA
        run: |
          ./TabularEditor.exe "*.SemanticModel/definition/database.tmdl" -A "./BPA_Rule.json"
        shell: pwsh
```

## Conditional Workflows

### Run Only on Model Changes

```yaml
name: Smart BPA Runner

on:
  pull_request:
    paths:
      - '**.tmdl'
      - '**.bim'
      - '**.SemanticModel/**'

jobs:
  bpa-check:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run BPA
        run: |
          # Only runs when model files change
```

## Reusable Workflows

### Create Reusable BPA Workflow

**`.github/workflows/reusable-bpa.yml`**:
```yaml
name: Reusable BPA Workflow

on:
  workflow_call:
    inputs:
      ruleset:
        required: true
        type: string
      model-path:
        required: true
        type: string

jobs:
  run-bpa:
    runs-on: windows-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download Tabular Editor
        run: |
          # Download steps
      
      - name: Run BPA
        run: |
          ./TabularEditor.exe "${{ inputs.model-path }}" -A "${{ inputs.ruleset }}"
```

**Use in another workflow**:
```yaml
name: Main Workflow

on:
  push:
    branches:
      - main

jobs:
  call-bpa:
    uses: ./.github/workflows/reusable-bpa.yml
    with:
      ruleset: "./BPA_Rule.json"
      model-path: "Contoso 10k.SemanticModel/definition/database.tmdl"
```

## Best Practices

1. **Use Secrets**: Never hardcode credentials
2. **Cache Dependencies**: Speed up workflows with caching
3. **Fail Fast**: Stop on critical errors
4. **Artifact Retention**: Keep important results
5. **Notifications**: Alert team of failures
6. **Documentation**: Comment complex workflows
7. **Testing**: Test workflows in feature branches first

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Marketplace Actions](https://github.com/marketplace?type=actions)
