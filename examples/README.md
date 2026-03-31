# Examples

This folder contains example BPA rulesets and configurations for different use cases.

## Files

### BPA_Strict.json

A stricter ruleset with higher severity levels, suitable for:
- Production environments
- Teams with mature Power BI practices
- Projects requiring high quality standards

**Key differences from default**:
- Division operator violations are Severity 3 (Error)
- Missing measure descriptions are Severity 3 (Error)
- Additional performance and formatting checks

**Usage**:
```yaml
./TabularEditor.exe "model.tmdl" -A "./examples/BPA_Strict.json"
```

## Creating Your Own Rulesets

1. Copy one of the example files
2. Modify the rules to match your requirements
3. Test with your Power BI projects
4. Update your workflow to use the new ruleset

## Rule Categories

Common categories to organize your rules:

- **DAX Expressions**: DAX syntax and best practices
- **Performance**: Query and refresh optimization
- **Documentation**: Descriptions and metadata
- **Formatting**: Display formats and naming
- **Data Model**: Relationships and structure
- **Security**: RLS and data protection

## Severity Guidelines

Choose appropriate severity levels:

- **Severity 1 (Info)**: Nice-to-have improvements
- **Severity 2 (Warning)**: Should fix but not blocking
- **Severity 3 (Error)**: Must fix before deployment

## Tips

1. Start with Severity 2 for new rules
2. Increase to Severity 3 once team is compliant
3. Use different rulesets for different environments
4. Document why each rule exists
5. Review and update rules regularly
