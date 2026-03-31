# Quick Start Guide

Get your Power BI CI/CD pipeline running in 10 minutes!

## ⚡ Prerequisites

- [ ] GitHub account
- [ ] Power BI Desktop (latest version)
- [ ] Git installed
- [ ] VS Code (recommended)

## 🚀 5-Step Setup

### 1️⃣ Fork or Clone This Repository

**Option A: Fork on GitHub**
1. Click the "Fork" button at the top right
2. Clone your fork locally

**Option B: Clone directly**
```bash
git clone https://github.com/YOUR_USERNAME/powerbi-cicd-github.git
cd powerbi-cicd-github
```

### 2️⃣ Add Your Power BI Project

**Quick test with Contoso sample**:
1. Download: [Contoso 10k sample](https://github.com/sql-bi/Contoso-Data-Generator-V2-Data/releases/download/ready-to-use-data/pbix-10k.7z)
2. Extract and open in Power BI Desktop
3. **File → Save As** → Choose this repository folder
4. **Important**: Save as `.pbip` format (not `.pbix`)

**Or use your own project**:
1. Open your `.pbix` file in Power BI Desktop
2. **File → Save As** → Choose this repository folder
3. Save as `.pbip` format

### 3️⃣ Update Workflow Path

If your project name is different from "Contoso 10k":

1. Open `.github/workflows/Standard_BPA_Runner.yml`
2. Find line with `Contoso 10k.SemanticModel/definition/database.tmdl`
3. Replace with your project name: `YOUR_PROJECT.SemanticModel/definition/database.tmdl`
4. Repeat for `Custom_BPA_Runner.yml`

### 4️⃣ Commit and Push

```bash
git add .
git commit -m "Add Power BI project and configure workflows"
git push origin main
```

### 5️⃣ Run Your First Workflow

1. Go to your repository on GitHub
2. Click **Actions** tab
3. Select **Standard BPA Runner**
4. Click **Run workflow** → **Run workflow**
5. Wait ~30 seconds
6. Click on the running workflow to see live logs

## ✅ Verify It Works

After the workflow completes:

1. Check the workflow status (should be ✅ green or ⚠️ yellow)
2. Scroll down to **Artifacts**
3. Download **bpa-results**
4. Open `BPA_output.txt` to see the analysis

## 🎯 What's Next?

### Test the Custom Ruleset

1. In Power BI Desktop, add a measure:
   ```dax
   Test Division = [Margin] / [Total Cost]
   ```
2. Save the project
3. Commit and push
4. The **Custom BPA Runner** will automatically trigger
5. Check the results - it should flag the division operator

### Customize BPA Rules

Edit `BPA_Rule.json` to add your own rules:

```json
[
  {
    "ID": "YOUR_RULE_ID",
    "Name": "Your rule name",
    "Category": "Your category",
    "Description": "What this checks",
    "Severity": 2,
    "Scope": "Measure",
    "Expression": "your validation logic",
    "CompatibilityLevel": 1200
  }
]
```

### Configure Auto-Triggers

The **Custom BPA Runner** already runs on:
- ✅ Every push to `main`
- ✅ Every pull request to `main`
- ✅ Manual trigger

To change triggers, edit `.github/workflows/Custom_BPA_Runner.yml`:

```yaml
on:
  push:
    branches:
      - main
      - develop  # Add more branches
  pull_request:
    branches:
      - main
  schedule:
    - cron: '0 9 * * 1'  # Weekly on Mondays at 9 AM
```

## 🔧 Common Adjustments

### Change Severity Levels

In `BPA_Rule.json`, adjust severity:
- `1` = Information (won't fail workflow)
- `2` = Warning (won't fail workflow)
- `3` = Error (will fail workflow)

### Add More Rules

See `docs/BPA_RULES_REFERENCE.md` for examples and syntax.

### Use Different Rulesets

In workflow files, change the `-A` parameter:

```yaml
# Microsoft official rules
-A "https://raw.githubusercontent.com/microsoft/Analysis-Services/master/BestPracticeRules/BPARules.json"

# Your custom rules
-A "./BPA_Rule.json"

# Multiple rulesets (create combined file)
-A "./BPA_Combined.json"
```

## 📚 Learn More

- **Full Setup Guide**: See `SETUP_GUIDE.md`
- **BPA Rules Reference**: See `docs/BPA_RULES_REFERENCE.md`
- **Troubleshooting**: See `docs/TROUBLESHOOTING.md`
- **Advanced Workflows**: See `docs/ADVANCED_WORKFLOWS.md`

## 💡 Tips

1. **Start Simple**: Use default rules first, then customize
2. **Test Locally**: Download Tabular Editor and test rules before committing
3. **Incremental Changes**: Add rules gradually to avoid overwhelming violations
4. **Document Rules**: Add clear descriptions so team understands why rules exist
5. **Review Regularly**: Check BPA results in code reviews

## 🆘 Need Help?

- Check `docs/TROUBLESHOOTING.md` for common issues
- Open an issue in this repository
- Review the [original article](https://fabricatedinsights.substack.com/p/building-your-power-bi-cicd-pipeline-7f9)

## 🎉 You're Done!

Your Power BI CI/CD pipeline is now running. Every time you push changes, BPA will automatically check your model for best practice violations.

Happy coding! 🚀
