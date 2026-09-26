---
title: "Azure DevOps"
linkTitle: "Azure DevOps"
weight: 20
type: docs
---

The [ALCops extension for Azure DevOps](https://marketplace.visualstudio.com/items?itemName=Arthurvdv.alcops-ado) provides a single pipeline task that downloads the ALCops analyzers with automatic target framework detection. It resolves the correct .NET version, downloads the matching analyzer package from NuGet, and extracts the DLLs to your pipeline workspace.

## Installation

Install the extension from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Arthurvdv.alcops-ado) into your Azure DevOps organization. Once installed, the `ALCopsDownloadAnalyzers` task becomes available in all pipelines across the organization.

## Quick start

Add the `ALCopsDownloadAnalyzers@1` task to your pipeline. It detects the target framework and downloads the analyzers in one step:

```yaml
steps:
  - task: ALCopsDownloadAnalyzers@1
    name: ALCopsDownload
    displayName: ALCops - Download Analyzers
    inputs:
      detectUsing: "latest"
      outputPath: "$(Build.SourcesDirectory)/.alcops"
```

Then pass the analyzer DLLs to the AL compiler using the task's output variable:

```yaml
  - pwsh: |
      $alcopsPath = "$(ALCopsDownload.outputDir)"
      dotnet al compile `
          "/project:$(Build.SourcesDirectory)" `
          "/analyzer:$alcopsPath/ALCops.ApplicationCop.dll" `
          "/analyzer:$alcopsPath/ALCops.DocumentationCop.dll" `
          "/analyzer:$alcopsPath/ALCops.FormattingCop.dll" `
          "/analyzer:$alcopsPath/ALCops.LinterCop.dll" `
          "/analyzer:$alcopsPath/ALCops.PlatformCop.dll" `
          "/analyzer:$alcopsPath/ALCops.TestAutomationCop.dll" `
          "/analyzer:$alcopsPath/ALCops.Common.dll"
```

## How detection works

The task needs to know which .NET target framework your AL compiler uses so it can download the matching analyzer binaries. There are three ways to provide this:

### 1. Auto-detect (recommended)

Pass a value to `detectUsing` and the task figures out the correct detection source automatically:

- A **URL** routes to BC artifact detection
- A **local path** routes to compiler-path detection
- A **version number** (e.g., `26.0.12345.0`) routes to NuGet DevTools first, then falls back to VS Marketplace if not found
- A **string** (`latest`, `current`, `preview`, etc.) routes to NuGet DevTools detection

```yaml
inputs:
  detectUsing: "$(artifactUrl)"    # URL → bc-artifact
  detectUsing: "$(compilerFolder)" # Path → compiler-path
  detectUsing: "26.0.12345.0"      # Version number → nuget-devtools → marketplace (fallback)
  detectUsing: "latest"            # String → nuget-devtools
```

### 2. Explicit detection source

Set `detectFrom` to force a specific detection source. This is useful when the auto-routing heuristic cannot distinguish your input:

```yaml
inputs:
  detectUsing: "preview"
  detectFrom: "marketplace"
```

### 3. Explicit TFM

```yaml
inputs:
  tfm: "net8.0"
```

{{% alert title="Avoid hardcoding the target framework" color="warning" %}}
Hardcoding `tfm` works today but **could break** when Microsoft updates the AL Language to a newer .NET version. When that happens, your pipeline will download analyzers built for the wrong runtime, causing load failures or incorrect results.

Use auto-detection or an explicit detection source instead.
{{% /alert %}}

## ALOps integration

If you use [ALOps](https://marketplace.visualstudio.com/items?itemName=Hodor.hodor-alops) for compilation, add `ALCopsDownloadAnalyzers@1` before the ALOps compile task. Pass the analyzer DLLs via the `al_analyzer` (v1) or `alcodeanalyzer` (v2/v3) parameter. See [Examples](examples/) for working snippets.

## Alternative: CLI

The extension wraps the [`@alcops/core`](https://www.npmjs.com/package/@alcops/core) npm package. If you prefer a tool-agnostic approach or need ALCops in a non-Azure DevOps CI system, you can use the CLI directly. See [CLI](../cli/).

## Next steps

- [Task reference](extension-task/) for all inputs and outputs
- [Examples](examples/) for integration with ALOps, NuGet DevTools, BC artifacts, and more
- [Troubleshooting](../troubleshooting/) for common issues
