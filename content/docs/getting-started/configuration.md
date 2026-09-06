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

Property names are case-insensitive. Comments and trailing commas are allowed. An empty, whitespace-only, comment-only or JSON-null local file uses defaults without CM0001. A declared inherited configuration must still contain a JSON object.

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

`Source` supports one anonymously accessible HTTP(S) URL or one absolute local file path. HTTP(S) URLs containing embedded credentials, such as `https://user:pass@example.com/alcops.json`, are rejected before a network request is made. The username and password are omitted from the resulting diagnostic. Committing an `alcops.json` that references an external source means trusting that source to supply analyzer settings.

For example, a Windows file path must be escaped in JSON:

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

Successfully loaded configurations are cached per workspace path during the analyzer session. Each compilation uses a consistent configuration snapshot. HTTP requests use a five-second timeout and accept at most **1 MiB (1,048,576 bytes)** of response content. The size limit also applies to chunked responses and responses without a `Content-Length` header.

If a declared `Extends` source cannot be resolved, **the entire configuration falls back to the built-in defaults**. Neither the inherited settings nor the local overrides are applied. This includes unreachable sources, HTTP errors, timeouts, oversized HTTP responses, unreadable files, malformed JSON, incompatible setting values, invalid `Extends.Source` declarations, and inheritance chains. A [CM0001 warning](/docs/analyzers/common/cm0001/) identifies the failing source and reason, so the fallback is visible in VS Code and command-line builds.

For example, with a local `CyclomaticComplexityThreshold` of `41` and an unavailable base configuration, the effective threshold is the built-in default `8`. Keeping `41` would apply only part of the intended configuration.

Unknown top-level setting names are handled separately: recognized settings still apply, with one `CM0001` warning per unknown name in either configuration. An invalid value in the base configuration is still an error even when a local override would replace it.

Failed HTTP requests are retried when a later compilation requests settings. This covers network errors, timeouts, HTTP error statuses and oversized response bodies. The failing compilation keeps its defaults and CM0001 diagnostic; a later successful compilation uses the recovered configuration without restarting the language server. There is no timer or background refresh.

The first analysis using an uncached HTTP source can wait for up to the five-second request timeout. Cancelling that analysis also cancels the request. Cancellation does not produce CM0001 or cache a failed result.

Successfully loaded settings and deterministic configuration errors, such as malformed JSON or an invalid source declaration, remain cached for the analyzer session. After changing these, restart the analyzer process; in VS Code, use **Developer: Reload Window**. Command-line builds reload settings when a new compiler process starts.

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
