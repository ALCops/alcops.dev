---
title: "Migrating from BusinessCentral.LinterCop"
linkTitle: "LinterCop Migration"
weight: 30
---

The [BusinessCentral.LinterCop](https://github.com/StefanMaron/BusinessCentral.LinterCop) project has been the foundation for community-driven AL code analysis and it's rules and diagnostics are now part of ALCops

- **[ApplicationCop](../analyzers/applicationcop/)** covers application conventions, labels, permissions, and UX standards.
- **[DocumentationCop](../analyzers/documentationcop/)** covers comments, XML documentation, and code documentation.
- **[FormattingCop](../analyzers/formattingcop/)** covers code style, formatting, and visual consistency.
- **[LinterCop](../analyzers/lintercop/)** covers code quality metrics, complexity, and modern AL patterns.
- **[PlatformCop](../analyzers/platformcop/)** covers platform-level correctness, runtime safety, and API constraints.
- **[TestAutomationCop](../analyzers/testautomationcop/)** covers test code structure and correctness.

If you are currently using the BusinessCentral.LinterCop, use the mapping below to find where each diagnostic moved.

## Diagnostic Mapping

| LinterCop | Title | Analyzer | ALCops |
|---|---|---|---|
| LC0001 | FlowFields should not be editable. | PlatformCop | [PC0001](../analyzers/platformcop/pc0001/) |
| LC0002 | Commit() needs a comment to justify its existence. | DocumentationCop | [DC0001](../analyzers/documentationcop/dc0001/) |
| LC0003 | Do not use an Object ID for properties or variable declarations. | LinterCop | [LC0003](../analyzers/lintercop/lc0003/) |
| LC0004 | DrillDownPageId and LookupPageId must be filled in table when table is used in list page. | ApplicationCop | [AC0001](../analyzers/applicationcop/ac0001/) |
| LC0005 | The casing of variable/method usage must align with the definition. | FormattingCop | [FC0002](../analyzers/formattingcop/fc0002/) |
| LC0006 | Fields with property AutoIncrement cannot be used in temporary table (TableType = Temporary). | PlatformCop | [PC0002](../analyzers/platformcop/pc0002/) |
| LC0007 | Every table needs to specify a value for the DataPerCompany property. Either true or false. | ApplicationCop | [AC0008](../analyzers/applicationcop/ac0008/) |
| LC0008 | Filter operators should not be used in SetRange. | PlatformCop | [PC0003](../analyzers/platformcop/pc0003/) |
| LC0009 | Show info message about code metrics for each function or trigger. | LinterCop | [LC0007](../analyzers/lintercop/lc0007/), [LC0009](../analyzers/lintercop/lc0009/) |
| LC0010 | Show warning about code metrics for each function or trigger. | LinterCop | [LC0008](../analyzers/lintercop/lc0008/), [LC0010](../analyzers/lintercop/lc0010/) |
| LC0011 | Every object needs to specify a value for the Access property. Either true or false. | PlatformCop | [PC0006](../analyzers/platformcop/pc0006/) |
| LC0012 | Using hardcoded IDs in functions like Codeunit.Run() is not allowed. | LinterCop | [LC0003](../analyzers/lintercop/lc0003/) |
| LC0013 | Any table with a single field in the PK of type code or text, should have set NotBlank on the PK field. | ApplicationCop | [AC0002](../analyzers/applicationcop/ac0002/) |
| LC0014 | The Caption of permissionset objects should not exceed the maximum length. | ApplicationCop | [AC0009](../analyzers/applicationcop/ac0009/) |
| LC0015 | All application objects should be covered by at least one permission set in the extension. | ApplicationCop | [AC0010](../analyzers/applicationcop/ac0010/) |
| LC0016 | Caption is missing. | ApplicationCop | [AC0011](../analyzers/applicationcop/ac0011/) |
| LC0017 | Writing to a FlowField is not common. Add a comment to explain this. | DocumentationCop | [DC0002](../analyzers/documentationcop/dc0002/) |
| LC0018 | Events in internal codeunits are not accessible to extensions and should therefore be avoided. | ApplicationCop | [AC0012](../analyzers/applicationcop/ac0012/) |
| LC0019 | If Data Classification is set on the Table, fields do not need the same classification. | LinterCop | [LC0019](../analyzers/lintercop/lc0019/) |
| LC0020 | If Application Area is set on the Page, controls do not need the same classification. | LinterCop | [LC0020](../analyzers/lintercop/lc0020/) |
| LC0021 | Confirm() must be implemented through the Confirm Management codeunit from the System Application. | ApplicationCop | [AC0004](../analyzers/applicationcop/ac0004/) |
| LC0022 | GlobalLanguage() must be implemented through the Translation Helper codeunit from the Base Application. | ApplicationCop | [AC0005](../analyzers/applicationcop/ac0005/) |
| LC0023 | Always provide fieldgroups DropDown and Brick on tables. | ApplicationCop | [AC0013](../analyzers/applicationcop/ac0013/) |
| LC0024 | Procedure or Trigger declaration should not end with semicolon. | FormattingCop | [FC0001](../analyzers/formattingcop/fc0001/) |
| LC0025 | Procedure must be either local, internal or define a documentation comment. | DocumentationCop | [DC0004](../analyzers/documentationcop/dc0004/) |
| LC0026 | ToolTip must end with a dot. | ApplicationCop | [AC0014](../analyzers/applicationcop/ac0014/) |
| LC0027 | Utilize the Page Management codeunit for launching page. | ApplicationCop | [AC0006](../analyzers/applicationcop/ac0006/) |
| LC0028 | Event subscriber arguments now use identifier syntax instead of string literals. | LinterCop | [LC0028](../analyzers/lintercop/lc0028/) |
| LC0029 | Use CompareDateTime method in Type Helper codeunit for DateTime variable comparisons. | ApplicationCop | — |
| LC0030 | Set Access property to Internal for Install/Upgrade codeunits. | ApplicationCop | [AC0007](../analyzers/applicationcop/ac0007/) |
| LC0031 | Use ReadIsolation instead of LockTable. | LinterCop | [LC0031](../analyzers/lintercop/lc0031/) |
| LC0032 | Clear(All) does not affect or change values for global variables in single instance codeunits. | PlatformCop | [PC0016](../analyzers/platformcop/pc0016/) |
| LC0033 | The specified runtime version in app.json is falling behind. | LinterCop | [LC0033](../analyzers/lintercop/lc0033/) |
| LC0034 | The property Extensible should be explicitly set for public objects. | PlatformCop | [PC0005](../analyzers/platformcop/pc0005/) |
| LC0035 | Explicitly set AllowInCustomizations for fields omitted on pages. | ApplicationCop | [AC0026](../analyzers/applicationcop/ac0026/) |
| LC0036 | ToolTip must start with the verb "Specifies". | ApplicationCop | [AC0015](../analyzers/applicationcop/ac0015/) |
| LC0037 | Do not use line breaks in ToolTip. | ApplicationCop | [AC0016](../analyzers/applicationcop/ac0016/) |
| LC0038 | Try to not exceed 200 characters (including spaces). | ApplicationCop | [AC0017](../analyzers/applicationcop/ac0017/) |
| LC0039 | The given argument has a different type from the one expected. | PlatformCop | [PC0017](../analyzers/platformcop/pc0017/) |
| LC0040 | Explicitly set the RunTrigger parameter on built-in methods. | LinterCop | [LC0040](../analyzers/lintercop/lc0040/) |
| LC0041 | Empty Captions should be Locked. | ApplicationCop | [AC0018](../analyzers/applicationcop/ac0018/) |
| LC0042 | AutoCalcFields should only be used for FlowFields or Blob fields. | PlatformCop | [PC0007](../analyzers/platformcop/pc0007/) |
| LC0043 | Use SecretText type to protect credentials and sensitive textual values from being revealed. | LinterCop | [LC0043](../analyzers/lintercop/lc0043/) |
| LC0044 | Tables coupled with TransferFields must have matching fields. | PlatformCop | [PC0020](../analyzers/platformcop/pc0020/), [PC0021](../analyzers/platformcop/pc0021/) |
| LC0045 | Zero (0) Enum value should be reserved for Empty Value. | ApplicationCop | [AC0019](../analyzers/applicationcop/ac0019/) |
| LC0046 | Label with suffix Tok must be locked. | ApplicationCop | [AC0020](../analyzers/applicationcop/ac0020/) |
| LC0047 | Locked Label must have a suffix Tok. | ApplicationCop | [AC0021](../analyzers/applicationcop/ac0021/) |
| LC0048 | Use Error with a ErrorInfo or Label variable to improve telemetry details. | LinterCop | [LC0048](../analyzers/lintercop/lc0048/) |
| LC0049 | SourceTable property not defined on Page. | PlatformCop | [PC0018](../analyzers/platformcop/pc0018/) |
| LC0050 | SetFilter with unsupported operator in filter expression. | PlatformCop | [PC0008](../analyzers/platformcop/pc0008/) |
| LC0051 | Do not assign a text to a target with smaller size. | PlatformCop | [PC0022](../analyzers/platformcop/pc0022/) |
| LC0052 | The internal procedure is declared but never used. | LinterCop | [LC0052](../analyzers/lintercop/lc0052/) |
| LC0053 | The internal procedure is only used in the object in which it is declared. Consider making the procedure local. | LinterCop | [LC0053](../analyzers/lintercop/lc0053/) |
| LC0054 | Interface name must start with the capital 'I' without any spaces following it. | LinterCop | [LC0054](../analyzers/lintercop/lc0054/) |
| LC0055 | The suffix Tok is meant to be used when the value of the label matches the name. | ApplicationCop | [AC0027](../analyzers/applicationcop/ac0027/) |
| LC0056 | Empty Enum values should not have a specified Caption property. | ApplicationCop | [AC0022](../analyzers/applicationcop/ac0022/) |
| LC0057 | Enum values must have non-empty a Caption to be selectable in the client. | ApplicationCop | [AC0023](../analyzers/applicationcop/ac0023/) |
| LC0058 | PageVariable.SetRecord(): You cannot use a temporary record for the Record parameter. | PlatformCop | [PC0036](../analyzers/platformcop/pc0036/) |
| LC0059 | Single quote escaping issue detected. | PlatformCop | [PC0019](../analyzers/platformcop/pc0019/) |
| LC0060 | The ApplicationArea property is not applicable to API pages. | PlatformCop | [PC0024](../analyzers/platformcop/pc0024/) |
| LC0061 | Pages of type API must have the ODataKeyFields property set to the SystemId field. | PlatformCop | [PC0025](../analyzers/platformcop/pc0025/) |
| LC0062 | Mandatory field is missing on API page. | PlatformCop | [PC0026](../analyzers/platformcop/pc0026/) |
| LC0063 | Consider naming field with a more descriptive name. | LinterCop | [LC0063](../analyzers/lintercop/lc0063/) |
| LC0064 | Missing ToolTip property on table field. | ApplicationCop | [AC0028](../analyzers/applicationcop/ac0028/) |
| LC0065 | Event subscriber var keyword mismatch. | PlatformCop | [PC0010](../analyzers/platformcop/pc0010/) |
| LC0066 | Duplicate ToolTip between page and table field. | ApplicationCop | [AC0029](../analyzers/applicationcop/ac0029/) |
| LC0067 | Set NotBlank property to false when 'No. Series' TableRelation exists. | ApplicationCop | [AC0003](../analyzers/applicationcop/ac0003/) |
| LC0068 | Informs the user that there are missing permission to access tabledata. | ApplicationCop | [AC0031](../analyzers/applicationcop/ac0031/) |
| LC0069 | Empty statements should be avoided or should have a comment explaining their use. | DocumentationCop | [DC0003](../analyzers/documentationcop/dc0003/) |
| LC0070 | Zero index access on 1-based List objects. | PlatformCop | [PC0004](../analyzers/platformcop/pc0004/) |
| LC0071 | Incorrect 'IsHandled' parameter assignment. | PlatformCop | [PC0023](../analyzers/platformcop/pc0023/) |
| LC0072 | The documentation comment must match the procedure syntax. | DocumentationCop | [DC0005](../analyzers/documentationcop/dc0005/) |
| LC0073 | Handled parameters in event signatures should be passed by var. | PlatformCop | [PC0011](../analyzers/platformcop/pc0011/) |
| LC0074 | Set values for FlowFilter fields using filtering methods. | PlatformCop | [PC0012](../analyzers/platformcop/pc0012/) |
| LC0075 | Incorrect number or type of arguments in .Get() method on Record object. | PlatformCop | [PC0013](../analyzers/platformcop/pc0013/) |
| LC0076 | The field with table relation should have at least the same length as the referenced field. | PlatformCop | [PC0028](../analyzers/platformcop/pc0028/) |
| LC0077 | Methods should always be called with parenthesis. | FormattingCop | [FC0003](../analyzers/formattingcop/fc0003/) |
| LC0078 | Temporary records should not trigger table triggers. | PlatformCop | [PC0027](../analyzers/platformcop/pc0027/) |
| LC0079 | Event publishers should not be public. | ApplicationCop | [AC0024](../analyzers/applicationcop/ac0024/) |
| LC0080 | Replace double quotes in JPath expressions with two single quotes. | PlatformCop | [PC0014](../analyzers/platformcop/pc0014/) |
| LC0081 | Use Rec.IsEmpty() for checking record existence. | LinterCop | [LC0081](../analyzers/lintercop/lc0081/) |
| LC0082 | Consider using a Query or Rec.Find('-') with Rec.Next(). | LinterCop | [LC0082](../analyzers/lintercop/lc0082/) |
| LC0083 | Use new Date/Time/DateTime methods for extracting parts. | LinterCop | [LC0083](../analyzers/lintercop/lc0083/) |
| LC0084 | Use return value for better error handling. | ApplicationCop | [AC0030](../analyzers/applicationcop/ac0030/) |
| LC0085 | Use the (CR)LFSeparator from the "Type Helper" codeunit. | ApplicationCop | [AC0025](../analyzers/applicationcop/ac0025/) |
| LC0086 | Use the new PageStyle datatype instead string literals. | LinterCop | [LC0086](../analyzers/lintercop/lc0086/) |
| LC0087 | Use IsNullGuid() to check for empty GUID values. | PlatformCop | [PC0015](../analyzers/platformcop/pc0015/) |
| LC0088 | Option types should be avoided, use enum if applicable. | LinterCop | [LC0088](../analyzers/lintercop/lc0088/) |
| LC0089 | Show Cognitive Complexity diagnostics for all methods. | LinterCop | [LC0089](../analyzers/lintercop/lc0089/) |
| LC0090 | Show Cognitive Complexity diagnostics for methods above threshold. | LinterCop | [LC0090](../analyzers/lintercop/lc0090/) |
| LC0091 | Translatable texts should be translated. | ApplicationCop | — |
| LC0092 | Names must match the allowed pattern and must not match the disallowed pattern. | LinterCop | [LC0092](../analyzers/lintercop/lc0092/) |
| LC0093 | Global procedure in test codeunit requires test attribute. | TestAutomationCop | [TA0001](../analyzers/testautomationcop/ta0001/) |
| LC0094 | A method invoked on a record must not contain same variable in parameter list as the one on which the call was made. | LinterCop | [LC0096](../analyzers/lintercop/lc0096/) |

Entries marked with **—** do not have a new diagnostic ID assigned (yet).

## Notable Changes

**Rules split into multiple diagnostics.** LC0009 and LC0010 each covered both cyclomatic complexity and maintainability index. In ALCops these are separate diagnostics (LC0007/LC0009 and LC0008/LC0010 respectively) so you can control each metric independently.

**Rules merged.** LC0003 and LC0012 both flagged the use of hardcoded object IDs. They are consolidated into [LC0003](../analyzers/lintercop/lc0003/).

**Rules split across analyzers.** LC0044 (TransferFields matching) is now covered by two PlatformCop diagnostics: [PC0020](../analyzers/platformcop/pc0020/) and [PC0021](../analyzers/platformcop/pc0021/).

## Updating Your Ruleset

If you maintain a custom `.ruleset.json` that references LinterCop rule IDs, update the IDs to match the new analyzer prefixes. For example:

```json
{
    "rules": [
        {
            "id": "PC0001",
            "action": "Warning",
            "justification": "Previously LC0001"
        },
        {
            "id": "AC0001",
            "action": "Warning",
            "justification": "Previously LC0004"
        }
    ]
}
```

Any pragma directives in your AL code (`#pragma warning disable LCxxxx`) need to be updated to the new diagnostic IDs as well.
