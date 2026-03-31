# Step-by-Step Setup Guide

This guide walks you through setting up your Power BI CI/CD pipeline with GitHub Actions from scratch.

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 0: Create a GitHub Account](#step-0-create-a-github-account)
3. [Step 1: Create a New Repository](#step-1-create-a-new-repository)
4. [Step 2: Push a Power BI Project File](#step-2-push-a-power-bi-project-file)
5. [Step 3: Create GitHub Actions Workflows](#step-3-create-github-actions-workflows)
6. [Step 4: Run Your First Workflow](#step-4-run-your-first-workflow)
7. [Step 5: Customize BPA Rules](#step-5-customize-bpa-rules)
8. [Troubleshooting](#troubleshooting)

## Prerequisites

Before you begin, make sure you have:
- [ ] A GitHub account
- [ ] Git installed on your computer
- [ ] Power BI Desktop installed
- [ ] VS Code (recommended) or another code editor
- [ ] Basic understanding of Git commands

## Step 0: Create a GitHub Account

If you don't have a GitHub account yet:
1. Go to [github.com/signup](https://github.com/signup)
2. Follow the registration process
3. Verify your email address

## Step 1: Create a New Repository

### On GitHub:

1. Log in to your GitHub account
2. Click the **+** icon in the top right corner
3. Select **New repository**
4. Fill in the details:
   - **Repository name**: `powerbi-cicd-github` (or your preferred name)
   - **Description**: "Power BI CI/CD pipeline with GitHub Actions"
   - **Visibility**: Choose Public or Private
   - **Initialize**: Do NOT check "Add a README file" (we'll add it later)
5. Click **Create repository**

### On Your Computer:

1. Open VS Code
2. Open a new Terminal (Terminal → New Terminal)
3. Clone the repository:

```bash
git clone https://github.com/YOUR_USERNAME/powerbi-cicd-github.git "C:\Users\YOUR_NAME\powerbi-cicd-github"
```

4. Navigate to the folder:

```bash
cd C:\Users\YOUR_NAME\powerbi-cicd-github
```

## Step 2: Push a Power BI Project File

### Download Sample Data:

1. Download the Contoso sample data:
   - Go to [Contoso Data Generator releases](https://github.com/sql-bi/Contoso-Data-Generator-V2-Data/releases/tag/ready-to-use-data)
   - Download `pbix-10k.7z`
   - Extract the archive

### Convert to .pbip Format:

1. Open the `.pbix` file in Power BI Desktop
2. Go to **File** → **Save As**
3. Navigate to your cloned repository folder
4. **Important**: Change the file type to **Power BI Project Files (.pbip)**
5. Name it `Contoso 10k` and save

### Commit and Push:

1. In VS Code, go to the **Source Control** tab (left sidebar)
2. You should see all the new files
3. Click the **+** icon next to "Changes" to stage all files
4. Enter a commit message: `Initial commit with PBIP file`
5. Click **Commit**
6. Click **Sync Changes** to push to GitHub

## Step 3: Create GitHub Actions Workflows

You have two options:

### Option A: Use This Repository's Files

1. Copy the files from this repository:
   - `.github/workflows/Standard_BPA_Runner.yml`
   - `.github/workflows/Custom_BPA_Runner.yml`
   - `BPA_Rule.json`
   - `README.md`

2. Commit and push:

```bash
git add .
git commit -m "Add GitHub Actions workflows and BPA rules"
git push
```

### Option B: Create Manually on GitHub

1. Go to your repository on GitHub
2. Click the **Actions** tab
3. Click **Set up a workflow yourself**
4. Name it `Standard_BPA_Runner.yml`
5. Paste the content from `.github/workflows/Standard_BPA_Runner.yml`
6. Click **Commit changes**
7. Repeat for the Custom BPA Runner workflow

## Step 4: Run Your First Workflow

### Run Standard BPA Runner:

1. Go to the **Actions** tab in your GitHub repository
2. Select **Standard BPA Runner** from the left panel
3. Click **Run workflow** button (top right)
4. Select the branch (usually `main`)
5. Click the green **Run workflow** button

### Monitor the Execution:

1. Wait a few seconds for the job to appear
2. Click on the workflow run to see details
3. Click on the job name to see real-time logs
4. Watch as it:
   - Checks out your code
   - Downloads Tabular Editor
   - Runs BPA analysis

### View Results:

1. After completion, scroll down to **Artifacts**
2. Download **bpa-results**
3. Extract and open `BPA_output.txt`

## Step 5: Customize BPA Rules

### Understanding the Custom Ruleset:

The `BPA_Rule.json` file contains custom rules. Here's the structure:

```json
{
  "ID": "UNIQUE_RULE_ID",
  "Name": "Human-readable name",
  "Category": "Grouping category",
  "Description": "What this rule checks",
  "Severity": 2,
  "Scope": "Measure, CalculatedColumn, CalculatedTable",
  "Expression": "The validation logic",
  "CompatibilityLevel": 1200
}
```

### Add Your Own Rule:

1. Open `BPA_Rule.json`
2. Add a new rule to the array:

```json
[
  {
    "ID": "DAX_DIVISION_COLUMNS",
    ...existing rule...
  },
  {
    "ID": "YOUR_NEW_RULE",
    "Name": "Your rule name",
    "Category": "Your category",
    "Description": "What your rule checks",
    "Severity": 2,
    "Scope": "Measure",
    "Expression": "your validation expression",
    "CompatibilityLevel": 1200
  }
]
```

3. Commit and push your changes
4. The Custom BPA Runner will automatically run (if configured for push events)

### Test Your Custom Rule:

1. In Power BI Desktop, create a measure that violates your rule
2. For the default rule, create: `Division not following BP = [Margin] / [Total Cost]`
3. Save the project
4. Commit and push
5. Check the workflow run to see the violation detected

## Troubleshooting

### Error: "Process completed with exit code 1"

**Cause**: BPA found Severity 3 (Error) violations

**Solution**:
- Review the workflow logs to identify which rule failed
- Fix the violation in your Power BI project
- Or change the rule severity from 3 to 2 in your custom ruleset

### Workflow doesn't trigger automatically

**Check**:
- The workflow file has the correct `on:` triggers
- You're pushing to the correct branch (e.g., `main`)
- The workflow file is in `.github/workflows/` directory

### Path not found error

**Check**:
- Your Power BI project path in the workflow matches your actual folder structure
- The path should be: `"YOUR_PROJECT_NAME.SemanticModel/definition/database.tmdl"`

### Tabular Editor download fails

**Solution**:
- Check your internet connection
- Verify the download URL is still valid
- Try running the workflow again (temporary network issues)

## Next Steps

Once you have the basic pipeline working:

1. **Add more BPA rules** to enforce your team's standards
2. **Configure branch protection** to require BPA checks before merging
3. **Set up notifications** for failed workflows
4. **Explore advanced workflows** like automated deployment
5. **Integrate with other tools** in your DevOps pipeline

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Tabular Editor Documentation](https://docs.tabulareditor.com/)
- [Power BI Best Practices](https://learn.microsoft.com/en-us/power-bi/guidance/)

---

**Need help?** Open an issue in this repository or refer to the original article.
