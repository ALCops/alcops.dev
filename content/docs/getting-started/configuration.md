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

### Where alcops.json is loaded from

The analyzer looks for `alcops.json` starting in the **app folder** — the directory that contains `app.json` — and then walks **up the parent directories** until it finds one. The first `alcops.json` it encounters is used; files at different levels are **not merged**. A file closer to the app fully shadows any file higher up.

This makes it easy to share one configuration across every app in a repository or multi-root workspace: place a single `alcops.json` at the workspace root, and let individual apps override it by adding their own.

```text
workspace/
├── alcops.json        ← shared by every app below
├── App1/
│   ├── app.json
│   └── alcops.json    ← overrides the shared file for App1 (nearest wins)
└── App2/
    └── app.json       ← uses workspace/alcops.json
```

The search stops at the filesystem root, or earlier at any directory it cannot access.

Because the nearest file wins outright, this mechanism *replaces* configuration rather than combining it. To share a common base **and** keep local overrides on top of it — for example a company-wide standard applied across repositories and machines — use [`Extends`](#extending-a-central-configuration) instead.

| Need | Mechanism | Behavior |
|------|-----------|----------|
| Share one config across apps in the **same repository or workspace** | Parent-directory traversal (a root `alcops.json`) | Nearest file wins; no merging |
| Share a **company-wide** base across repositories or machines, with local overrides | [`Extends.Source`](#extending-a-central-configuration) (recommended) | Base and local are merged; local wins |

### Extending a central configuration

`Extends` is the recommended way to apply one configuration across every app in your company. Unlike a shared file found by [directory traversal](#where-alcopsjson-is-loaded-from) — which replaces the local file entirely — `Extends` layers a centrally maintained base *underneath* the local `alcops.json`, so each project can still override individual settings.

Use `Extends.Source` to load the central `alcops.json` as the base for the project configuration:

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

#### Loading behavior and limits

**Fetching**

- HTTP(S) requests use a **five-second timeout** and accept at most **1 MiB (1,048,576 bytes)** of response content. The size limit also applies to chunked responses and responses without a `Content-Length` header.
- The first analysis using an uncached HTTP source — and each retry after a failure cooldown — can wait up to that five-second timeout. Cancelling the analysis also cancels the request; cancellation never produces CM0001, caches a failed result, or starts a cooldown.

**Failure is all-or-nothing**

- If a declared `Extends` source cannot be resolved, **the entire configuration falls back to the built-in defaults** — neither the inherited settings nor the local overrides are applied. For example, with a local `CyclomaticComplexityThreshold` of `41` and an unavailable base, the effective threshold is the default `8`; keeping `41` would apply only part of the intended configuration.
- This covers unreachable sources, HTTP errors, timeouts, oversized responses, unreadable files, malformed JSON, incompatible setting values, invalid `Extends.Source` declarations, and inheritance chains.
- A [CM0001 warning](/docs/analyzers/common/cm0001/) identifies the failing source and reason, so the fallback is visible in VS Code and command-line builds.
- Unknown top-level setting names are handled separately: recognized settings still apply, with one CM0001 per unknown name in either configuration. An invalid value in the base configuration is an error even when a local override would replace it.

**Caching and reloads**

- Successfully loaded configurations are cached per workspace path for the analyzer session, and each compilation uses a consistent snapshot.
- Failed HTTP requests are cached per workspace path for **30 seconds after the failed request completes** (network errors, timeouts, HTTP error statuses, oversized bodies). During the cooldown, compilations reuse defaults and CM0001 without fetching again, so offline editing does not re-request on every analysis pass; cache hits do not extend the cooldown. After it expires, the first compilation that needs settings makes one shared retry — a success is cached for the session, another failure starts a new 30-second cooldown. Existing compilations keep their original settings and diagnostic snapshot even after recovery; there is no timer or background refresh.
- Deterministic errors, such as malformed JSON or an invalid source declaration, are also cached for the session. After changing these settings, restart the analyzer process; in VS Code, use **Developer: Reload Window**. Command-line builds reload when a new compiler process starts.

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
