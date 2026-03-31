# Power BI CI/CD Pipeline with GitHub Actions

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/Oussamabenbella/Power-BI-CI-CD-Pipeline-with-GitHub-Actions/Custom_BPA_Runner.yml?label=Custom%20BPA&style=flat-square)
![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/Oussamabenbella/Power-BI-CI-CD-Pipeline-with-GitHub-Actions/Standard_BPA_Runner.yml?label=Standard%20BPA&style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)

This project demonstrates how to build a CI/CD pipeline for Power BI using GitHub Actions and the Best Practice Analyzer (BPA).

Based on the article: [Building your Power BI CI/CD pipeline with GitHub Actions (Part 1)](https://fabricatedinsights.substack.com/p/building-your-power-bi-cicd-pipeline-7f9)

## 🆕 Latest Updates

- ✅ **PR BPA Comments**: Automatic BPA analysis on Pull Requests with detailed comments
- ✅ **Weekly Audit**: Automated weekly comprehensive BPA audit
- ✅ **Performance**: Caching added to all workflows for faster execution

## 📋 Prerequisites

- GitHub account
- Power BI Desktop
- A Power BI project file (.pbip format)
- Git installed on your machine
- VS Code (recommended)

## 🚀 Quick Start

### Step 1: Clone this repository

```bash
git clone https://github.com/YOUR_USERNAME/powerbi-cicd-github.git
cd powerbi-cicd-github
```

### Step 2: Add your Power BI project

1. Download the [Contoso sample data](https://github.com/sql-bi/Contoso-Data-Generator-V2-Data/releases/download/ready-to-use-data/pbix-10k.7z)
2. Open the `.pbix` file in Power BI Desktop
3. Save it as a `.pbip` file in the root of this repository
4. Commit and push your changes

### Step 3: Run the workflows

#### Option A: Standard BPA Runner (Manual trigger only)
- Go to the **Actions** tab in your GitHub repository
- Select **Standard BPA Runner** workflow
- Click **Run workflow**
- This uses the official Microsoft BPA ruleset

#### Option B: Custom BPA Runner (Auto-triggered)
- This workflow runs automatically on:
  - Every push to `main` branch
  - Every pull request targeting `main` branch
  - Can also be triggered manually
- Uses the custom ruleset defined in `BPA_Rule.json`

## 📁 Project Structure

```
powerbi-cicd-github/
├── .github/
│   └── workflows/
│       ├── Standard_BPA_Runner.yml    # Workflow with default BPA rules
│       └── Custom_BPA_Runner.yml      # Workflow with custom BPA rules
├── BPA_Rule.json                      # Custom BPA ruleset
├── Contoso 10k.SemanticModel/         # Your Power BI project (to be added)
│   └── definition/
│       └── database.tmdl
└── README.md
```

## 🔧 Workflows Explained

### Standard BPA Runner

This workflow:
1. Downloads and installs Tabular Editor 2.x (with caching)
2. Runs BPA using Microsoft's official ruleset
3. Saves results to `BPA_output.txt`
4. Uploads results as an artifact

**Trigger:** Manual only (`workflow_dispatch`)

### Custom BPA Runner

This workflow:
1. Downloads and installs Tabular Editor 2.x (with caching)
2. Runs BPA using the custom ruleset (`BPA_Rule.json`)
3. Saves results to `BPA_output.txt`
4. Uploads results as an artifact

**Triggers:**
- Manual (`workflow_dispatch`)
- Push to `main` branch
- Pull requests targeting `main` branch

### 🆕 PR BPA with Comments

Automatically analyzes Pull Requests and posts comments:
- Runs only when Power BI files are modified
- Posts detailed summary as PR comment
- Shows error and warning counts
- Includes full report in artifacts

**Trigger:** Automatic on Pull Requests to `main`

### 🆕 Weekly BPA Audit

Comprehensive weekly audit:
- Runs every Monday at 9 AM UTC
- Uses Microsoft's complete ruleset
- Categorizes violations by type
- Keeps 90-day history

**Triggers:**
- Automatic: Every Monday at 9 AM UTC
- Manual: Via Actions tab

## 📝 Custom BPA Rules

The `BPA_Rule.json` file contains a custom rule that checks for:
- **DAX Division**: Ensures developers use `DIVIDE()` function instead of `/` operator

You can modify this file to add your own rules. Each rule has:
- **ID**: Unique identifier
- **Name**: Descriptive name
- **Category**: Grouping category
- **Description**: What the rule checks
- **Severity**: 1 (Info), 2 (Warning), or 3 (Error)
- **Scope**: Which objects to check (Measure, CalculatedColumn, etc.)
- **Expression**: The logic to evaluate
- **CompatibilityLevel**: Minimum compatibility level

## 🎯 Understanding BPA Severity Levels

- **Severity 1 (Information)**: Informational messages
- **Severity 2 (Warning)**: Best practice violations (workflow continues)
- **Severity 3 (Error)**: Critical issues (workflow fails with exit code 1)

## 🔍 Viewing Results

After a workflow runs:
1. Go to the **Actions** tab
2. Click on the workflow run
3. Download the **bpa-results** artifact
4. Open `BPA_output.txt` to see the analysis

## ⚙️ Customization

### Change the Power BI project path

If your project structure is different, update the path in the workflow files:

```yaml
./TabularEditor.exe "YOUR_PROJECT_NAME.SemanticModel/definition/database.tmdl" -A ...
```

### Add more BPA rules

Edit `BPA_Rule.json` and add additional rules to the array:

```json
[
  {
    "ID": "YOUR_RULE_ID",
    "Name": "Your rule name",
    ...
  },
  {
    "ID": "ANOTHER_RULE_ID",
    "Name": "Another rule",
    ...
  }
]
```

### Change workflow triggers

Modify the `on:` section in the workflow files:

```yaml
on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Mondays
```

## 📚 Resources

- [Tabular Editor Documentation](https://docs.tabulareditor.com/)
- [Tabular Editor CLI Options](https://docs.tabulareditor.com/te2/Command-line-Options.html)
- [Microsoft BPA Rules](https://github.com/microsoft/Analysis-Services/tree/master/BestPracticeRules)
- [Contoso Sample Data](https://github.com/sql-bi/Contoso-Data-Generator-V2-Data)
- [Original Article](https://fabricatedinsights.substack.com/p/building-your-power-bi-cicd-pipeline-7f9)

## 🤝 Contributing

Feel free to submit issues or pull requests to improve this project!

## 📄 License

This project is open source and available under the MIT License.
