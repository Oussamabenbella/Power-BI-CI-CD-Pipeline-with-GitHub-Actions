# Power BI CI/CD Pipeline

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/Oussamabenbella/Power-BI-CI-CD-Pipeline-with-GitHub-Actions/Custom_BPA_Runner.yml?label=BPA)
![License](https://img.shields.io/badge/license-MIT-blue)

Automated quality checks for Power BI projects using GitHub Actions and Best Practice Analyzer.

## Prerequisites

- GitHub account
- Power BI Desktop
- A Power BI project file (.pbip format)
- Git installed on your machine
- VS Code (recommended)

## Quick Start

1. Clone and add your Power BI project (.pbip format)
2. Push to `main` - Custom BPA runs automatically
3. Check results in Actions tab

### Manual Analysis

Go to Actions > Standard BPA Runner > Run workflow for full Microsoft ruleset analysis.

## Workflows

**Custom BPA Runner**
- Triggers: Push to main, PR to main, manual
- Uses: Custom rules (BPA_Rule.json)

**Standard BPA Runner**  
- Triggers: Manual only
- Uses: Microsoft official ruleset

## Customization

Edit `BPA_Rule.json` to add your own rules. Update project path in workflow files if needed.

## Resources

- [Tabular Editor Docs](https://docs.tabulareditor.com/)
- [Microsoft BPA Rules](https://github.com/microsoft/Analysis-Services/tree/master/BestPracticeRules)

## License

MIT
