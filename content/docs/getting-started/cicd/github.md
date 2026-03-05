---
title: "GitHub"
linkTitle: "GitHub"
weight: 10
type: docs
---

There are two main approaches to running ALCops in GitHub: using AL-Go for GitHub (the recommended path) or building your own GitHub Actions workflow.

## AL-Go for GitHub

AL-Go for GitHub has built-in support for custom code analyzers through its settings file.

Add the `customCodeCops` property to your AL-Go `settings.json` (typically `.github/AL-Go-Settings.json` or the project-level settings file):

```json
{
    "enableCodeCops": true,
    "customCodeCops": [
        "ALCops.ApplicationCop.dll",
        "ALCops.DocumentationCop.dll",
        "ALCops.FormattingCop.dll",
        "ALCops.LinterCop.dll",
        "ALCops.PlatformCop.dll",
        "ALCops.TestAutomationCop.dll",
        "ALCops.Common.dll"
    ]
}
```

AL-Go downloads the DLLs at build time and passes them to the compiler automatically.

{{% alert title="Tip" %}}
Pair `customCodeCops` with `"vsixFile": "latest"` to ensure the compiler version matches the latest AL Language extension. For prerelease testing, use `"vsixFile": "preview"` and the corresponding prerelease analyzer DLLs.
{{% /alert %}}

### AL-Go Helper

ALCops provides a dedicated [AL-Go helper](https://github.com/ALCops/AL-Go) that automates analyzer download and setup using AL-Go's pipeline hook mechanism. This is the simplest way to keep your analyzers up to date.

1. Create a file called `PipelineInitialize.ps1` in your `.AL-Go` folder.
2. Add the following content:

```PowerShell
Param([Hashtable] $parameters)

# Configuration
$scriptUrl = "https://raw.githubusercontent.com/ALCops/AL-Go/v1.0.0/scripts/Install-ALCops.ps1"

# Download and run the installer script
$scriptPath = Join-Path ([System.IO.Path]::GetTempPath()) "Install-ALCops.ps1"
Invoke-WebRequest -Uri $scriptUrl -OutFile $scriptPath -UseBasicParsing

& $scriptPath
```

The installer runs during AL-Go's **PipelineInitialize** stage, before compilation starts. It downloads the analyzer package from NuGet and stages the DLLs so AL-Go picks them up as custom code cops automatically.

#### Configuration Options

Pass parameters to `Install-ALCops.ps1` to control the version and target framework:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `$packageVersion` | `""` | Version channel: `""` (latest stable), `"alpha"`, `"beta"`, or an exact version like `"1.2.3"`. |
| `$targetFramework` | `net8.0` | Leave blank for current AL Language versions. Set to `netstandard2.1` for AL Language versions below v16.0. |

Example with a specific version:

```PowerShell
& $scriptPath -packageVersion "1.2.3"
```

### Ruleset in AL-Go

To configure rule severity in AL-Go builds, place a `.ruleset.json` file in your project root. AL-Go picks it up automatically. See [Configuration](../../configuration/) for the file format.