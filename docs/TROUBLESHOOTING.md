# Troubleshooting Guide

Common issues and their solutions when working with Power BI CI/CD pipelines.

## GitHub Actions Issues

### ❌ Error: "Process completed with exit code 1"

**Symptom**: Workflow fails with this generic error message

**Cause**: BPA found one or more Severity 3 (Error) violations

**Solutions**:
1. Review the workflow logs before the error to see which BPA rule failed
2. Fix the violation in your Power BI model
3. Alternatively, modify the rule severity in your custom ruleset from 3 to 2
4. For testing purposes, you can temporarily use only Severity 2 rules

**Example**:
```json
{
  "Severity": 2  // Changed from 3 to 2 to allow workflow to continue
}
```

### ❌ Workflow doesn't trigger automatically

**Symptom**: Push or PR doesn't trigger the workflow

**Checklist**:
- [ ] Workflow file is in `.github/workflows/` directory
- [ ] File has `.yml` or `.yaml` extension
- [ ] The `on:` section includes the correct triggers
- [ ] You're pushing to the branch specified in the workflow
- [ ] The workflow file has valid YAML syntax

**Example trigger configuration**:
```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

### ❌ Error: "TabularEditor.exe not found"

**Symptom**: Workflow fails when trying to run Tabular Editor

**Cause**: Download or extraction failed

**Solutions**:
1. Check the "Download Tabular Editor" step logs
2. Verify the download URL is accessible
3. Ensure the extraction step completed successfully
4. Try re-running the workflow (might be a temporary network issue)

### ❌ Error: "Path not found" for database.tmdl

**Symptom**: Cannot find the semantic model file

**Cause**: Incorrect path in the workflow

**Solution**: Update the path in your workflow file to match your project structure

**Check your actual structure**:
```
Your Project Name.SemanticModel/
└── definition/
    └── database.tmdl
```

**Update workflow**:
```yaml
./TabularEditor.exe "Your Project Name.SemanticModel/definition/database.tmdl" -A ...
```

## Power BI Issues

### ❌ Cannot save as .pbip format

**Symptom**: Save As dialog doesn't show .pbip option

**Cause**: Using an older version of Power BI Desktop

**Solution**: 
- Update to the latest version of Power BI Desktop
- .pbip format requires Power BI Desktop version from 2023 or later

### ❌ Project files not showing in repository

**Symptom**: After saving as .pbip, files don't appear in Git

**Cause**: Files might be gitignored or not saved correctly

**Solutions**:
1. Check `.gitignore` file - ensure it's not excluding `.pbip` folders
2. Verify the project was saved in the correct location
3. Refresh VS Code's Source Control view
4. Check if files exist in File Explorer

## BPA Rule Issues

### ❌ Custom rule not working

**Symptom**: BPA doesn't detect violations you expect

**Checklist**:
- [ ] `BPA_Rule.json` has valid JSON syntax
- [ ] Rule expression is correct
- [ ] Scope matches the object type you're testing
- [ ] CompatibilityLevel is appropriate
- [ ] Workflow is using the custom ruleset file

**Validate JSON**:
```bash
# Use an online JSON validator or:
python -m json.tool BPA_Rule.json
```

### ❌ Too many BPA violations

**Symptom**: Workflow fails with hundreds of violations

**Solutions**:
1. Start with a subset of rules (lower severity)
2. Fix violations incrementally
3. Use Severity 2 (Warning) instead of 3 (Error) during initial setup
4. Create a baseline and improve over time

## Git Issues

### ❌ Large file size warnings

**Symptom**: Git warns about large files when committing .pbip

**Cause**: Power BI projects can be large, especially with embedded data

**Solutions**:
1. Use Git LFS (Large File Storage) for .pbip files
2. Ensure `.pbix` files are in `.gitignore` (use .pbip instead)
3. Consider using DirectQuery instead of Import mode

### ❌ Merge conflicts in .pbip files

**Symptom**: Git shows conflicts in semantic model files

**Cause**: Multiple people editing the same Power BI project

**Solutions**:
1. Use feature branches and coordinate changes
2. Merge frequently to avoid large conflicts
3. For complex conflicts, use Power BI Desktop to resolve
4. Consider splitting large models into smaller ones

## Performance Issues

### ❌ Workflow takes too long

**Symptom**: Workflow runs for several minutes

**Causes & Solutions**:
1. **Large model**: BPA analysis time increases with model size
   - Consider running BPA only on changed files
   - Use caching for Tabular Editor download
2. **Too many rules**: Each rule adds processing time
   - Prioritize critical rules
   - Run different rule sets in separate workflows

**Optimization example**:
```yaml
- name: Cache Tabular Editor
  uses: actions/cache@v3
  with:
    path: TabularEditor.exe
    key: tabular-editor-${{ runner.os }}
```

## Debugging Tips

### Enable verbose logging

Add this to your workflow for more detailed output:

```yaml
- name: Run BPA with Custom Ruleset
  run: |
    ./TabularEditor.exe "..." -A "..." -V | Tee-Object -FilePath "./BPA_output.txt"
  shell: pwsh
```

### Test locally

Before pushing to GitHub:

1. Download Tabular Editor locally
2. Run BPA from command line:
```powershell
.\TabularEditor.exe "path\to\database.tmdl" -A "path\to\BPA_Rule.json"
```

### Check workflow syntax

Use GitHub's workflow validator:
1. Go to Actions tab
2. Click "New workflow"
3. Paste your YAML
4. GitHub will highlight syntax errors

## Still Having Issues?

1. Check the [GitHub Actions documentation](https://docs.github.com/en/actions)
2. Review [Tabular Editor documentation](https://docs.tabulareditor.com/)
3. Open an issue in this repository with:
   - Detailed description
   - Workflow logs
   - Your environment details
   - Steps to reproduce

## Useful Commands

```bash
# Validate YAML syntax
yamllint .github/workflows/*.yml

# Check Git status
git status

# View workflow logs locally (if using act)
act -l

# Test BPA rule JSON
python -m json.tool BPA_Rule.json
```
