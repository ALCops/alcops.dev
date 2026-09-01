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
    "CognitiveComplexityThreshold": 15,
    "CyclomaticComplexityThreshold": 8,
    "MaintainabilityIndexThreshold": 20
}
```

### Available properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Extends` | object | `null` | Loads one external `alcops.json` as the base configuration |
| `CognitiveComplexityThreshold` | integer | `15` | Maximum cognitive complexity before a diagnostic is reported |
| `CyclomaticComplexityThreshold` | integer | `8` | Maximum cyclomatic complexity before a diagnostic is reported |
| `MaintainabilityIndexThreshold` | integer | `20` | Minimum maintainability index before a diagnostic is reported |
| `LanguagesToTranslate` | string[] | `null` | Language codes to check for missing translations |
| `NamingPatterns` | object | `null` | Per-target naming pattern overrides |
| `UseSequentialGuidScope` | string | `null` | Set to `"AllGuidFields"` to require sequential GUIDs on all GUID fields |

Property names are case-insensitive. Comments and trailing commas are allowed.

### Extending a central configuration

Use `Extends.Source` to load a centrally maintained `alcops.json` as the base for the project configuration:

```json
{
    "Extends": {
        "Source": "https://example.com/company.alcops.json"
    },
    "SubscriberNamingPattern": "{Event Source}_{Event Name}[_{Element Name}]"
}
```

`Source` supports one anonymously accessible HTTP(S) URL or one absolute local file path. For example, a Windows file path must be escaped in JSON:

```json
{
    "Extends": {
        "Source": "C:\\ALCops\\company.alcops.json"
    }
}
```

The referenced configuration provides the base values, and settings specified in the local `alcops.json` take precedence. The merge follows these rules:

- Scalar values are replaced by the local value.
- Arrays are replaced as a whole rather than combined.
- Nested objects are merged property by property.
- A referenced configuration cannot declare its own `Extends` section; inheritance chains are not supported.

The external configuration is loaded once for each workspace path during the analyzer session. HTTP requests use a five-second timeout. If the source is unavailable, cannot be read, contains invalid JSON or incompatible setting values, ALCops ignores it and continues with the local configuration. This keeps local development and CI builds functional when a central configuration service is temporarily unavailable.

### NamingPatterns

Override the default naming validation patterns per target. Each target accepts `AllowPattern`, `DisallowPattern`, `AllowDescription`, and `DisallowDescription`.

```json
{
    "NamingPatterns": {
        "Variable": {
            "AllowPattern": "^[A-Z]",
            "AllowDescription": "should start with an uppercase letter"
        },
        "EnumValue": {
            "DisallowPattern": "^_",
            "DisallowDescription": "should not start with an underscore"
        }
    }
}
```

Valid targets: `Procedure`, `LocalProcedure`, `GlobalProcedure`, `EventSubscriber`, `EventDeclaration`, `Variable`, `Parameter`, `ReturnValue`, `Object`, `Field`, `Action`, `EnumValue`, `Control`.

`LocalProcedure`, `GlobalProcedure`, `EventSubscriber`, and `EventDeclaration` inherit from `Procedure` when no explicit override is configured.

### LanguagesToTranslate

Specify which language codes must have translations present in the `.xlf` files.

```json
{
    "LanguagesToTranslate": ["da-DK", "de-DE"]
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
