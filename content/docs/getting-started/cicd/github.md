---
title: "GitHub"
linkTitle: "GitHub"
weight: 10
type: docs
---

There are two main approaches to running ALCops in GitHub: using AL-Go for GitHub (the recommended path) or building your own GitHub Actions workflow.

## AL-Go for GitHub

AL-Go for GitHub has built-in support for custom code analyzers through its settings file. ALCops integrates via the `PreCompileApp` hook, which AL-Go invokes once per app type (`app`, `testApp`, `bcptApp`) right before compilation.

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

Create `.AL-Go/PreCompileApp.ps1` in your repository:

```powershell
Param(
    [ValidateSet('app','testApp')]
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

# PreCompileApp runs once per app group (apps + testApps). Skip the download
# if analyzers are already on disk so we don't re-download for the testApp pass.
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
# Workaround: altool's --customanalyzers forwards a comma-separated list to
# alc.exe and only resolves the FIRST entry against the project root. The rest
# stay as relative paths and alc.exe then resolves them against the per-app
# project folder, where '.alcops' does not exist.
# Rewrite CustomAnalyzers in $compilationParams to absolute paths so alc.exe
# can find every DLL regardless of which project it's compiling.
if ($compilationParams -and $compilationParams.Value.CustomAnalyzers) {
    $resolved = @()
    foreach ($cop in $compilationParams.Value.CustomAnalyzers) {
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
    $compilationParams.Value.CustomAnalyzers = $resolved
    Write-Host "Resolved CustomAnalyzers paths to absolute:"
    $resolved | ForEach-Object { Write-Host "  $_" }
}
```

### How it works

1. AL-Go calls `PreCompileApp.ps1` once per app type (`app`, `testApp`, `bcptApp`) before compiling that group.
2. The script checks whether the `.alcops/` folder already contains DLLs; if so, it skips the download (avoids re-downloading on the `testApp` pass).
3. On the first pass, the script uses [`@alcops/core`](https://www.npmjs.com/package/@alcops/core) via `npx` to:
   - Auto-detect the target framework from the BC artifact URL (`$env:artifact`, set by AL-Go)
   - Download the matching ALCops analyzer package from NuGet
   - Extract the DLLs to `.alcops/` in the workspace
4. AL-Go picks up the DLLs via the `customCodeCops` setting and passes them to the compiler.

The GitHub Actions guard at the top prevents the script from failing when AL-Go invokes `PreCompileApp.ps1` in local development scenarios (e.g., `localDevEnv.ps1`), where `GITHUB_WORKSPACE` and other CI variables are not available.

### Migrating from PipelineInitialize.ps1

Older AL-Go documentation recommended `PipelineInitialize.ps1` for custom analyzer setup. That hook is part of the BcContainerHelper/Run-AlPipeline overrides and is **not called** when workspace compilation is enabled (the default in AL-Go v6+). If your pipeline silently stopped running your initialization script after enabling `workspaceCompilation`, this is why.

`PreCompileApp.ps1` is the replacement. Key differences:

| | PipelineInitialize.ps1 | PreCompileApp.ps1 |
|---|---|---|
| **When it runs** | Once at pipeline start | Once per app type (`app`, `testApp`, `bcptApp`) |
| **Parameters** | None | `[string] $appType`, `[ref] $compilationParams` |
| **Workspace compilation** | Not called | Called |

To migrate: rename your script to `PreCompileApp.ps1`, update its `Param()` block to accept the new parameters, and add the download-caching guard (the script above handles this). See [microsoft/AL-Go#2235](https://github.com/microsoft/AL-Go/issues/2235) for background.

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