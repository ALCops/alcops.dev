---
title: "Configuration"
linkTitle: "Configuration"
weight: 30
type: docs
---

ALCops supports several mechanisms to configure which rules are active and how they behave. These mechanisms work across all environments: VS Code, command line, and CI/CD pipelines.

## alcops.json

The `alcops.json` file provides analyzer-specific configuration. Place it in the root of your AL project alongside `app.json`.

```json
{
    "cognitiveComplexityThreshold": 15,
    "cyclomaticComplexityThreshold": 8,
    "maintainabilityIndexThreshold": 20
}
```

### Available properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `cognitiveComplexityThreshold` | integer | `15` | Maximum cognitive complexity before a diagnostic is reported |
| `cyclomaticComplexityThreshold` | integer | `8` | Maximum cyclomatic complexity before a diagnostic is reported |
| `maintainabilityIndexThreshold` | integer | `20` | Minimum maintainability index before a diagnostic is reported |
| `languagesToTranslate` | string[] | `null` | Language codes to check for missing translations |
| `namingPatterns` | object | `null` | Per-target naming pattern overrides |
| `useSequentialGuidScope` | string | `null` | Set to `"AllGuidFields"` to require sequential GUIDs on all GUID fields |

Property names are case-insensitive. Comments and trailing commas are allowed.

### namingPatterns

Override the default naming validation patterns per target. Each target accepts `allowPattern`, `disallowPattern`, `allowDescription`, and `disallowDescription`.

```json
{
    "namingPatterns": {
        "Variable": {
            "allowPattern": "^[A-Z]",
            "allowDescription": "should start with an uppercase letter"
        },
        "EnumValue": {
            "disallowPattern": "^_",
            "disallowDescription": "should not start with an underscore"
        }
    }
}
```

Valid targets: `Procedure`, `LocalProcedure`, `GlobalProcedure`, `EventSubscriber`, `EventDeclaration`, `Variable`, `Parameter`, `ReturnValue`, `Object`, `Field`, `Action`, `EnumValue`, `Control`.

`LocalProcedure`, `GlobalProcedure`, `EventSubscriber`, and `EventDeclaration` inherit from `Procedure` when no explicit override is configured.

### languagesToTranslate

Specify which language codes must have translations present in the `.xlf` files.

```json
{
    "languagesToTranslate": ["da-DK", "de-DE"]
}
```

{{% alert title="Note" %}}
The `alcops.json` file is read when the analyzer loads. In VS Code, changes to this file require reloading the window (`Ctrl+Shift+P` → **Developer: Reload Window**) to take effect.
{{% /alert %}}

## Ruleset Files (.ruleset.json)

Ruleset files are the standard Microsoft mechanism for configuring diagnostic severity. Create a `.ruleset.json` file and reference it in your `app.json` or VS Code settings.

```json
{
    "name": "My Project Ruleset",
    "rules": [
        {
            "id": "AC0001",
            "action": "Warning"
        },
        {
            "id": "LC0007",
            "action": "None"
        },
        {
            "id": "PC0006",
            "action": "Error"
        }
    ]
}
```

Supported actions:

| Action | Effect |
|--------|--------|
| `Error` | Treat the diagnostic as a build error. |
| `Warning` | Treat the diagnostic as a warning (default for most rules). |
| `Info` | Treat the diagnostic as an informational message. |
| `Hidden` | Hide the diagnostic but keep it active (visible in code actions). |
| `None` | Disable the diagnostic entirely. |

For full details, see the [Microsoft documentation on ruleset files](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-rule-set-syntax-for-code-analysis-tools).

## Pragma Directives

Use `#pragma warning` directives to suppress specific diagnostics inline:

```al
#pragma warning disable AC0001
table 50100 MyTable
{
    // AC0001 is suppressed for this block
}
#pragma warning restore AC0001
```

This is useful for targeted suppression where a ruleset-level change would be too broad.