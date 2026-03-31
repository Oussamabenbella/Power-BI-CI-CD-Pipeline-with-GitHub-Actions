# BPA Rules Reference Guide

Complete guide to understanding and creating Best Practice Analyzer (BPA) rules for Power BI.

## Table of Contents

1. [Rule Structure](#rule-structure)
2. [Severity Levels](#severity-levels)
3. [Scope Options](#scope-options)
4. [Expression Language](#expression-language)
5. [Common Rule Examples](#common-rule-examples)
6. [Official Microsoft Rules](#official-microsoft-rules)

## Rule Structure

Each BPA rule is a JSON object with the following properties:

```json
{
  "ID": "UNIQUE_IDENTIFIER",
  "Name": "Human-readable rule name",
  "Category": "Grouping category",
  "Description": "Detailed explanation of what this rule checks",
  "Severity": 1,
  "Scope": "Measure, CalculatedColumn",
  "Expression": "Validation logic using TOM expressions",
  "CompatibilityLevel": 1200,
  "FixExpression": "Optional: automatic fix logic"
}
```

### Property Details

| Property | Required | Type | Description |
|----------|----------|------|-------------|
| ID | Yes | String | Unique identifier (use UPPERCASE_WITH_UNDERSCORES) |
| Name | Yes | String | Display name for the rule |
| Category | Yes | String | Logical grouping (e.g., "DAX Expressions", "Performance") |
| Description | Yes | String | Detailed explanation of the rule and why it matters |
| Severity | Yes | Integer | 1 (Info), 2 (Warning), or 3 (Error) |
| Scope | Yes | String | Comma-separated list of object types to check |
| Expression | Yes | String | C# LINQ expression that returns true for violations |
| CompatibilityLevel | Yes | Integer | Minimum model compatibility level (usually 1200) |
| FixExpression | No | String | C# expression to automatically fix the violation |

## Severity Levels

### Severity 1: Information ℹ️

- **Purpose**: Informational messages or suggestions
- **Behavior**: Logged but doesn't affect workflow status
- **Use for**: Style preferences, optional optimizations

**Example**: Suggesting to hide key columns

### Severity 2: Warning ⚠️

- **Purpose**: Best practice violations that should be addressed
- **Behavior**: Logged, workflow continues
- **Use for**: Performance issues, maintainability concerns

**Example**: Using division operator instead of DIVIDE()

### Severity 3: Error ❌

- **Purpose**: Critical issues that must be fixed
- **Behavior**: Workflow fails (exit code 1)
- **Use for**: Breaking changes, serious performance problems

**Example**: Missing descriptions on measures

## Scope Options

The `Scope` property determines which objects the rule applies to:

| Scope Value | Description |
|-------------|-------------|
| `Measure` | DAX measures |
| `CalculatedColumn` | Calculated columns |
| `CalculatedTable` | Calculated tables |
| `Table` | All tables (including calculated) |
| `Column` | All columns (including calculated) |
| `Relationship` | Model relationships |
| `Hierarchy` | Hierarchies |
| `Level` | Hierarchy levels |
| `Partition` | Table partitions |
| `DataSource` | Data sources |
| `Model` | The entire model |
| `Culture` | Translations |
| `Perspective` | Perspectives |

**Multiple scopes**: Separate with commas
```json
"Scope": "Measure, CalculatedColumn, CalculatedTable"
```

## Expression Language

BPA expressions use C# LINQ syntax with access to Tabular Object Model (TOM) properties.

### Common Properties

```csharp
// For Measures, CalculatedColumns, CalculatedTables
Expression          // The DAX expression
Description         // Object description
IsHidden            // Visibility status
DisplayFolder       // Display folder path
FormatString        // Format string

// For Tables
Columns             // Collection of columns
Measures            // Collection of measures
Partitions          // Collection of partitions

// For Columns
DataType            // Column data type
SourceColumn        // Source column name
```

### Common Methods

```csharp
// String operations
.Contains("text")
.StartsWith("text")
.EndsWith("text")
.Length

// DAX tokenization
Tokenize()          // Parse DAX into tokens
.Any()              // Check if any token matches
.Type               // Token type (e.g., DIV, FUNCTION)
.Next               // Next token
.Previous           // Previous token

// Collections
.Count()            // Number of items
.Any(condition)     // True if any item matches
.All(condition)     // True if all items match
```

### Expression Examples

**Check if expression contains specific text**:
```csharp
Expression.Contains("CALCULATE")
```

**Check if description is empty**:
```csharp
string.IsNullOrWhiteSpace(Description)
```

**Check for division operator**:
```csharp
Tokenize().Any(Type = DIV)
```

**Check for specific DAX function**:
```csharp
Tokenize().Any(Type = FUNCTION and Value = "COUNTROWS")
```

**Check column data type**:
```csharp
DataType = "String" and IsHidden = false
```

## Common Rule Examples

### 1. Avoid Division Operator

```json
{
  "ID": "DAX_DIVISION",
  "Name": "Avoid division (use DIVIDE function instead)",
  "Category": "DAX Expressions",
  "Description": "Use DIVIDE() instead of / to avoid division by zero errors",
  "Severity": 2,
  "Scope": "Measure, CalculatedColumn, CalculatedTable",
  "Expression": "Tokenize().Any(\n    Type = DIV and\n    Next.Type <> INTEGER_LITERAL and\n    Next.Type <> REAL_LITERAL\n)",
  "CompatibilityLevel": 1200
}
```

### 2. Measures Must Have Descriptions

```json
{
  "ID": "MEASURE_DESCRIPTION",
  "Name": "Measures should have descriptions",
  "Category": "Documentation",
  "Description": "All measures should have a description for documentation",
  "Severity": 2,
  "Scope": "Measure",
  "Expression": "string.IsNullOrWhiteSpace(Description)",
  "CompatibilityLevel": 1200
}
```

### 3. Hide Key Columns

```json
{
  "ID": "HIDE_KEY_COLUMNS",
  "Name": "Hide key columns",
  "Category": "Model Design",
  "Description": "Columns with 'ID' or 'Key' in the name should be hidden",
  "Severity": 1,
  "Scope": "Column",
  "Expression": "(Name.Contains(\"ID\") or Name.Contains(\"Key\")) and IsHidden = false",
  "CompatibilityLevel": 1200
}
```

### 4. Avoid SELECT * in Partitions

```json
{
  "ID": "AVOID_SELECT_STAR",
  "Name": "Avoid SELECT * in partitions",
  "Category": "Performance",
  "Description": "Explicitly list columns instead of using SELECT *",
  "Severity": 2,
  "Scope": "Partition",
  "Expression": "Query.Contains(\"SELECT *\")",
  "CompatibilityLevel": 1200
}
```

### 5. Use VAR for Repeated Expressions

```json
{
  "ID": "USE_VAR_FOR_REPEATED",
  "Name": "Use VAR for repeated expressions",
  "Category": "Performance",
  "Description": "Repeated sub-expressions should be stored in variables",
  "Severity": 1,
  "Scope": "Measure",
  "Expression": "Expression.Length > 100 and not Expression.Contains(\"VAR\")",
  "CompatibilityLevel": 1200
}
```

### 6. Avoid Floating Point Data Types

```json
{
  "ID": "AVOID_FLOAT",
  "Name": "Avoid floating point data types",
  "Category": "Data Types",
  "Description": "Use Decimal or Integer instead of Double for better precision",
  "Severity": 2,
  "Scope": "Column",
  "Expression": "DataType = \"Double\"",
  "CompatibilityLevel": 1200
}
```

## Official Microsoft Rules

Microsoft maintains an official BPA ruleset at:
```
https://raw.githubusercontent.com/microsoft/Analysis-Services/master/BestPracticeRules/BPARules.json
```

### Categories in Official Ruleset:

1. **DAX Expressions**: DAX best practices
2. **Performance**: Query and refresh performance
3. **Formatting**: Display formatting standards
4. **Maintenance**: Model maintainability
5. **Naming Conventions**: Object naming standards
6. **Error Prevention**: Avoiding common errors

### Using Official Rules

**In your workflow**:
```yaml
./TabularEditor.exe "..." -A "https://raw.githubusercontent.com/microsoft/Analysis-Services/master/BestPracticeRules/BPARules.json"
```

**Combine with custom rules**:
1. Download the official rules
2. Merge with your custom rules
3. Reference the combined file

## Creating Your Own Rules

### Step 1: Identify the Issue

What problem are you trying to prevent?
- Performance issue?
- Maintainability concern?
- Documentation requirement?
- Naming convention?

### Step 2: Define the Scope

Which objects should be checked?
- Measures only?
- All DAX objects?
- Specific tables?

### Step 3: Write the Expression

Start simple and test:
```json
{
  "Expression": "Name.Contains(\"test\")"
}
```

Then refine:
```json
{
  "Expression": "Name.Contains(\"test\") and IsHidden = false"
}
```

### Step 4: Test the Rule

1. Create a test Power BI project with violations
2. Run BPA locally
3. Verify the rule catches the issue
4. Verify it doesn't create false positives

### Step 5: Document the Rule

Write a clear description explaining:
- What the rule checks
- Why it matters
- How to fix violations

## Advanced Techniques

### Using FixExpression

Automatically fix violations:

```json
{
  "ID": "SET_FORMAT_STRING",
  "Name": "Measures should have format strings",
  "Expression": "string.IsNullOrWhiteSpace(FormatString)",
  "FixExpression": "FormatString = \"#,0\""
}
```

### Conditional Logic

```csharp
// Check multiple conditions
(Condition1 or Condition2) and not Condition3

// Check nested properties
Table.Name.StartsWith("Dim") and IsHidden = false
```

### Regular Expressions

```csharp
// Use System.Text.RegularExpressions
System.Text.RegularExpressions.Regex.IsMatch(Name, "^[A-Z]")
```

## Testing Your Rules

### Local Testing

```powershell
# Run BPA locally
.\TabularEditor.exe "model.bim" -A "your_rules.json" -V

# Save output to file
.\TabularEditor.exe "model.bim" -A "your_rules.json" > output.txt
```

### Validation Checklist

- [ ] Rule has unique ID
- [ ] Expression syntax is valid
- [ ] Rule catches intended violations
- [ ] No false positives
- [ ] Severity level is appropriate
- [ ] Description is clear and helpful
- [ ] Tested on multiple models

## Resources

- [Tabular Editor BPA Documentation](https://docs.tabulareditor.com/te2/Best-Practice-Analyzer.html)
- [Microsoft Official Rules](https://github.com/microsoft/Analysis-Services/tree/master/BestPracticeRules)
- [Tabular Object Model Reference](https://learn.microsoft.com/en-us/dotnet/api/microsoft.analysisservices.tabular)
- [DAX Best Practices](https://www.sqlbi.com/articles/best-practices-using-summarize-and-addcolumns/)

## Contributing Rules

Have a useful rule? Contribute it:
1. Test thoroughly
2. Document clearly
3. Submit a pull request
4. Share with the community!
