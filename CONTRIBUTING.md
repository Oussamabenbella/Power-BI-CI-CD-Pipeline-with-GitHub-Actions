# Contributing to Power BI CI/CD Pipeline

Thank you for your interest in contributing to this project! This document provides guidelines for contributing.

## How to Contribute

### Reporting Issues

If you encounter a bug or have a suggestion:

1. Check if the issue already exists in the [Issues](../../issues) section
2. If not, create a new issue with:
   - Clear title and description
   - Steps to reproduce (for bugs)
   - Expected vs actual behavior
   - Your environment details (OS, Power BI version, etc.)

### Submitting Changes

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes**
   - Follow the existing code style
   - Test your changes thoroughly
   - Update documentation if needed
4. **Commit your changes**
   ```bash
   git commit -m "Add: description of your changes"
   ```
5. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```
6. **Create a Pull Request**
   - Provide a clear description of the changes
   - Reference any related issues

## BPA Rules Contributions

When adding new BPA rules:

1. Add the rule to `BPA_Rule.json`
2. Include:
   - Unique ID
   - Descriptive name
   - Clear description
   - Appropriate severity level
   - Valid expression
3. Test the rule with sample Power BI projects
4. Document the rule in the README

## Workflow Contributions

When modifying workflows:

1. Test the workflow in your own repository first
2. Ensure it works on Windows runners
3. Add comments explaining complex steps
4. Update documentation to reflect changes

## Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Focus on the issue, not the person
- Help others learn and grow

## Questions?

Feel free to open an issue for questions or discussions!
