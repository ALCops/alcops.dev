---
title: "GitHub"
linkTitle: "GitHub"
weight: 10
type: docs
---

There are two main approaches to running ALCops in GitHub: using AL-Go for GitHub (the recommended path) or building your own GitHub Actions workflow.

## AL-Go for GitHub

AL-Go for GitHub has built-in support for custom code analyzers through its settings file. ALCops integrates via the `PipelineInitialize` hook, which runs before compilation starts.

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

Create `.AL-Go/PipelineInitialize.ps1` in your repository:

```powershell
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
```

### How it works

1. AL-Go calls `PipelineInitialize.ps1` before compilation starts.
2. The script uses [`@alcops/core`](https://www.npmjs.com/package/@alcops/core) via `npx` to:
   - Auto-detect the target framework from the BC artifact URL (`$env:artifact`, set by AL-Go)
   - Download the matching ALCops analyzer package from NuGet
   - Extract the DLLs to `.alcops/` in the workspace
3. AL-Go picks up the DLLs via the `customCodeCops` setting and passes them to the compiler.

The GitHub Actions guard at the top prevents the script from failing when AL-Go invokes `PipelineInitialize.ps1` in local development scenarios (e.g., `localDevEnv.ps1`), where `GITHUB_WORKSPACE` and other CI variables are not available.

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