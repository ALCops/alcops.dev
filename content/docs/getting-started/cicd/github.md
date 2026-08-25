---
title: "GitHub"
linkTitle: "GitHub"
weight: 10
type: docs
---

There are two main approaches to running ALCops in GitHub: using AL-Go for GitHub (the recommended path) or building your own GitHub Actions workflow.

## AL-Go for GitHub

AL-Go for GitHub has built-in support for custom code analyzers through its settings file. ALCops integrates via an initialization hook script that downloads the analyzer DLLs before compilation.

### 1. Add the settings

Add the `customCodeCops` property to your AL-Go settings file (`.AL-Go/settings.json` or the project-level settings):

```json
{
  "customCodeCops": [
    ".alcops/ALCops.ApplicationCop.dll",
    ".alcops/ALCops.DocumentationCop.dll",
    ".alcops/ALCops.FormattingCop.dll",
    ".alcops/ALCops.LinterCop.dll",
    ".alcops/ALCops.PlatformCop.dll",
    ".alcops/ALCops.TestAutomationCop.dll",
    ".alcops/ALCops.Common.dll"
  ]
}
```

AL-Go passes these paths to the compiler automatically. For a full list of AL-Go settings, see the [AL-Go advanced settings reference](https://github.com/microsoft/AL-Go/blob/main/Scenarios/settings.md#advanced-settings).

### 2. Create the initialization script

Which hook script is relevant for ALCops initialization depends on whether [workspace compilation](https://github.com/microsoft/AL-Go/releases/tag/v9.0) is enabled in your `.AL-Go/settings.json`:

```json
"workspaceCompilation": {
  "enabled": true
}
```

| Workspace compilation | Hook script |
|---|---|
| Disabled (default) | `PipelineInitialize.ps1` |
| Enabled | `PreCompileApp.ps1` |

`PipelineInitialize.ps1` runs only when workspace compilation is disabled. When workspace compilation is enabled, AL-Go skips this hook, so `PreCompileApp.ps1` initializes ALCops instead.

Both hook scripts can safely be included in reusable AL-Go templates. `PreCompileApp.ps1` may also be invoked by the classic compilation path, where `CustomAnalyzers` is not part of the compilation parameters. The script only applies the workspace-specific analyzer path workaround when that parameter is available.

{{< tabpane persist=false >}}
{{< tab header="PipelineInitialize (default)" lang="powershell" >}}
# .AL-Go/PipelineInitialize.ps1
Param([Hashtable] $parameters)

# Skip when not running in GitHub Actions (e.g. localDevEnv.ps1).
# AL-Go can invoke PipelineInitialize.ps1 in local development scenarios
# where CI environment variables like GITHUB_WORKSPACE are not available.
$githubActions = $env:GITHUB_ACTIONS
if ([string]::IsNullOrWhiteSpace($githubActions) -or $githubActions.Trim().ToLowerInvariant() -eq "false") {
    Write-Host "Not running in GitHub Actions. Skipping ALCops analyzer install."
    return
}

$ErrorActionPreference = "Stop"

$outputPath = Join-Path $env:GITHUB_WORKSPACE ".alcops"

Write-Host "Installing ALCops analyzers..."
Write-Host "  Output path: $outputPath"
Write-Host "  Detect using: $env:artifact"

npx --yes @alcops/core download `
    --output $outputPath `
    --detect-using $env:artifact `
    --detect-from bc-artifact

if ($LASTEXITCODE -ne 0) {
    throw "ALCops download failed with exit code $LASTEXITCODE"
}

Write-Host "ALCops analyzers installed successfully."
{{< /tab >}}
{{< tab header="PreCompileApp (workspace compilation)" lang="powershell" >}}
# .AL-Go/PreCompileApp.ps1
Param(
    [ValidateSet('app', 'testApp', 'bcptApp')]
    [string] $appType,
    [ref] $compilationParams
)

$ErrorActionPreference = "Stop"

# Skip when not running in GitHub Actions (e.g. localDevEnv.ps1 / local builds)
$githubActions = $env:GITHUB_ACTIONS
if ([string]::IsNullOrWhiteSpace($githubActions) -or $githubActions.Trim().ToLowerInvariant() -eq "false") {
    Write-Host "Not running in GitHub Actions. Skipping ALCops analyzer install."
    return
}

$outputPath = Join-Path $env:GITHUB_WORKSPACE ".alcops"

# PreCompileApp runs once per app group (apps + testApps + bcptApps). Skip the
# download if analyzers are already on disk so they are not downloaded again.
$alreadyInstalled = (Test-Path $outputPath) -and
    @(Get-ChildItem -Path $outputPath -Filter '*.dll' -ErrorAction SilentlyContinue).Count -gt 0

if ($alreadyInstalled) {
    Write-Host "ALCops analyzers already present in $outputPath. Skipping download (appType=$appType)."
}
else {
    Write-Host "Installing ALCops analyzers (appType=$appType)..."
    Write-Host "  Output path: $outputPath"
    Write-Host "  Detect using: $env:artifact"

    npx --yes '@alcops/core' download `
        --output $outputPath `
        --detect-using $env:artifact `
        --detect-from bc-artifact

    if ($LASTEXITCODE -ne 0) {
        throw "ALCops download failed with exit code $LASTEXITCODE"
    }

    Write-Host "ALCops analyzers installed successfully."
}

# https://github.com/microsoft/AL-Go/issues/2235
# Workaround: some AL-Go versions pass relative custom analyzer paths during
# workspace compilation. Resolve them against the workspace so alc.exe can
# find every DLL regardless of which project it is compiling.
# PreCompileApp can also run in the classic compilation path, where the
# compilation parameters do not contain CustomAnalyzers.
if (
    $compilationParams -and
    $compilationParams.Value -and
    $compilationParams.Value.ContainsKey('CustomAnalyzers') -and
    $compilationParams.Value['CustomAnalyzers']
) {
    $resolved = @()
    foreach ($cop in $compilationParams.Value['CustomAnalyzers']) {
        if ([System.IO.Path]::IsPathRooted($cop)) {
            $resolved += $cop
            continue
        }
        $abs = Join-Path $env:GITHUB_WORKSPACE $cop
        if (Test-Path $abs) {
            $resolved += (Resolve-Path -LiteralPath $abs).Path
        }
        else {
            Write-Host "::Warning::Custom analyzer not found at expected path: $abs"
            $resolved += $abs
        }
    }
    $compilationParams.Value['CustomAnalyzers'] = $resolved
    Write-Host "Resolved CustomAnalyzers paths to absolute:"
    $resolved | ForEach-Object { Write-Host "  $_" }
}
{{< /tab >}}
{{< /tabpane >}}

### How it works

Both scripts use [`@alcops/core`](https://www.npmjs.com/package/@alcops/core) via `npx` to auto-detect the target framework from the BC artifact URL (`$env:artifact`, set by AL-Go), download the matching ALCops analyzer package from NuGet, and extract the DLLs to `.alcops/` in the workspace. AL-Go then picks up the DLLs via the `customCodeCops` setting and passes them to the compiler.

The GitHub Actions guard at the top of each script prevents failures in local development scenarios (e.g., `localDevEnv.ps1`), where `GITHUB_WORKSPACE` and other CI variables are not available.

**PipelineInitialize.ps1** runs once at pipeline start, before any compilation begins. The script downloads the analyzers once and they are available for all subsequent compilation steps.

**PreCompileApp.ps1** runs once per app type (`app`, `testApp`, `bcptApp`) right before compilation of that group. Because it runs multiple times, the script includes a caching guard that skips the download if `.alcops/` already contains DLLs from a previous pass.

### Pinning a version

By default the script downloads the latest stable release. To pin to a specific ALCops version, add `--version`:

```powershell
npx --yes @alcops/core download `
    --output $outputPath `
    --detect-using $env:artifact `
    --detect-from bc-artifact `
    --version "1.0.0"
```

### Ruleset in AL-Go

To configure rule severity in AL-Go builds, place a `.ruleset.json` file in your project root. AL-Go picks it up automatically. See [Configuration](../../configuration/) for the file format.

## Custom GitHub Actions

If you do not use AL-Go, you can call the `@alcops/core` CLI directly in a workflow step:

```yaml
- name: Download ALCops Analyzers
  run: |
    result=$(npx --yes @alcops/core download --detect-using latest --output ./analyzers)
    echo "dir=$(echo "$result" | jq -r '.outputDir')" >> "$GITHUB_OUTPUT"
  id: alcops

- name: Compile with ALCops
  run: |
    alc /project:. \
      /analyzer:${{ steps.alcops.outputs.dir }}/ALCops.LinterCop.dll \
      /analyzer:${{ steps.alcops.outputs.dir }}/ALCops.ApplicationCop.dll \
      /analyzer:${{ steps.alcops.outputs.dir }}/ALCops.Common.dll
```

For the full CLI reference including all detection sources and options, see [CLI](../cli/).