---
title: "Azure DevOps"
linkTitle: "Azure DevOps"
weight: 20
type: docs
---

The [ALCops extension for Azure DevOps](https://marketplace.visualstudio.com/items?itemName=Arthurvdv.alcops-ado) provides pipeline tasks that download, install and configure the ALCops Code Analyzers for your Business Central AL projects. It handles NuGet package resolution, target framework detection and analyzer extraction for your pipelines.

## Installation

Install the extension from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Arthurvdv.alcops-ado) into your Azure DevOps organization. Once installed, the four ALCops tasks become available in all pipelines across the organization.

## Quick Start

The simplest way to get started is to add the `ALCopsInstallAnalyzers` task and then pass the analyzer DLLs to the AL compiler:

```yaml
steps:
  - task: ALCopsInstallAnalyzers@0
    inputs:
      tfm: "net8.0"

  # Simplified example.
  # Demonstrates that custom code analyzers must be referenced during compilation
  # so their diagnostics are executed as part of the build process.
  - pwsh: |
        al compile `
            "/project:$(Build.SourcesDirectory)" `
            "/analyzer:$(Build.SourcesDirectory)/.alcops/ALCops.ApplicationCop.dll" `
            "/analyzer:$(Build.SourcesDirectory)/.alcops/ALCops.DocumentationCop.dll" `
            "/analyzer:$(Build.SourcesDirectory)/.alcops/ALCops.FormattingCop.dll" `
            "/analyzer:$(Build.SourcesDirectory)/.alcops/ALCops.LinterCop.dll" `
            "/analyzer:$(Build.SourcesDirectory)/.alcops/ALCops.PlatformCop.dll" `
            "/analyzer:$(Build.SourcesDirectory)/.alcops/ALCops.TestAutomationCop.dll" `
            "/analyzer:$(Build.SourcesDirectory)/.alcops/ALCops.Common.dll"
```

{{% alert title="Do not hardcode the target framework" color="warning" %}}
Hardcoding `tfm: "net8.0"` might work today, but it **will** break when Microsoft updates the AL Language to a newer .NET version (e.g., `net10.0`). When that happens, your pipeline will silently install analyzers built for the wrong runtime, causing load failures or incorrect analysis results.

**Use one of the DetectTfm tasks** (described below) to determine the correct target framework automatically. This keeps your pipeline resilient across AL Language updates without manual intervention.
{{% /alert %}}

## Tasks

The extension ships four pipeline tasks:

| Task | Description |
|------|-------------|
| **ALCopsInstallAnalyzers** | Download and install ALCops analyzer DLLs |
| **ALCopsDetectTfmFromBCArtifact** | Detect the target framework from a BC artifact URL |
| **ALCopsDetectTfmFromNuGetDevTools** | Detect the target framework from the BC DevTools NuGet package |
| **ALCopsDetectTfmFromMarketplace** | Detect the target framework from the AL Language VS Code extension |

## Recommended Patterns

Instead of hardcoding the TFM, use one of the three DetectTfm tasks to determine the correct value automatically. Pick the approach that matches the data already available in your pipeline.

### Auto-detect from BC Artifact

If your pipeline already resolves a BC artifact URL (e.g., via `Get-BCArtifactUrl` in PowerShell), pass it to `ALCopsDetectTfmFromBCArtifact`. The task reads the artifact manifest to determine the .NET version used by that BC build.

```yaml
steps:
  - task: ALCopsDetectTfmFromBCArtifact@0
    name: detectTfm
    inputs:
      artifactUrl: "$(bcArtifactUrl)"

  - task: ALCopsInstallAnalyzers@0
    inputs:
      tfm: "$(detectTfm.tfm)"
```

### Auto-detect from NuGet DevTools

If you use the `Microsoft.Dynamics.BusinessCentral.Development.Tools` NuGet package for compilation, the DevTools version maps directly to a target framework. Set `version` to `latest`, `prerelease`, or a specific version number.

```yaml
steps:
  - task: ALCopsDetectTfmFromNuGetDevTools@0
    name: detectTfm
    inputs:
      version: "latest"

  - task: ALCopsInstallAnalyzers@0
    inputs:
      tfm: "$(detectTfm.tfm)"
```

### Auto-detect from VS Marketplace

When you do not have a BC artifact URL or NuGet DevTools version handy, you can detect the TFM from the AL Language VS Code extension published on the Visual Studio Marketplace. The task downloads the extension metadata and inspects the compiler DLL inside.

```yaml
steps:
  - task: ALCopsDetectTfmFromMarketplace@0
    name: detectTfm
    inputs:
      channel: "current"

  - task: ALCopsInstallAnalyzers@0
    inputs:
      tfm: "$(detectTfm.tfm)"
```

Set `channel` to `prerelease` to match a pre-release AL Language version, or pin to a specific version with the `extensionVersion` input.

## Task Reference

### ALCopsInstallAnalyzers

Download and install ALCops analyzer DLLs.

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `version` | No | `latest` | ALCops version to install: `latest`, `prerelease`, or an exact version (e.g., `1.2.3`). |
| `packageSource` | No | `nuget` | Where to download the package: `nuget` (nuget.org) or `local` (a `.nupkg` file on disk). |
| `localPackagePath` | No | | Path to a local `.nupkg` file. Only used when `packageSource` is `local`. |
| `tfm` | No | | Target framework (`net8.0`, `netstandard2.1`, `net10.0`). Leave empty and provide `compilerPath` for auto-detection. |
| `compilerPath` | No | | Path to the directory containing `Microsoft.Dynamics.Nav.CodeAnalysis.dll`. Used for auto-detection when `tfm` is not set. The directory is searched recursively if the DLL is not in the root. |
| `outputPath` | No | `$(Build.SourcesDirectory)/.alcops` | Where to place the extracted analyzer DLLs. |

#### Output Variables

| Variable | Description |
|----------|-------------|
| `alcopsVersion` | The installed ALCops analyzer version. |
| `tfm` | The target framework that was used (detected or specified). |
| `analyzerPath` | Full path to the directory containing the extracted analyzer DLLs. |
| `analyzers` | Semicolon-separated list of analyzer DLL paths. |

### ALCopsDetectTfmFromBCArtifact

Detect the target framework from a Business Central artifact URL by reading its manifest.

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `artifactUrl` | Yes | | Business Central artifact URL (e.g., from `Get-BCArtifactUrl`). |

#### Output Variables

| Variable | Description |
|----------|-------------|
| `tfm` | The detected target framework moniker (e.g., `net8.0`, `netstandard2.1`). |
| `dotNetVersion` | The .NET version string from the artifact manifest (e.g., `8.0.24`). |

### ALCopsDetectTfmFromNuGetDevTools

Detect the target framework from the `Microsoft.Dynamics.BusinessCentral.Development.Tools` NuGet package version.

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `version` | No | `latest` | DevTools version: `latest`, `prerelease`, or a specific version (e.g., `26.0.12345.0`). |

#### Output Variables

| Variable | Description |
|----------|-------------|
| `tfm` | The detected target framework moniker (e.g., `net8.0`, `netstandard2.1`). |
| `devToolsVersion` | The resolved DevTools package version. |

### ALCopsDetectTfmFromMarketplace

Detect the target framework from the AL Language VS Code extension on the Visual Studio Marketplace.

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `channel` | No | `current` | Which channel to check: `current` (latest stable) or `prerelease`. |
| `extensionVersion` | No | | Pin to a specific AL Language extension version (e.g., `15.0.12345.0`). Overrides `channel`. |

#### Output Variables

| Variable | Description |
|----------|-------------|
| `tfm` | The detected target framework moniker (e.g., `net8.0`, `netstandard2.1`). |
| `extensionVersion` | The resolved AL Language extension version. |
| `assemblyVersion` | The assembly version of the CodeAnalysis DLL inside the extension. |

## Example Pipelines

For complete, working pipeline examples that use ALCops, visit [dev.azure.com/Arthurvdv/ALCops](https://dev.azure.com/Arthurvdv/ALCops). These pipelines demonstrate real-world setups including TFM auto-detection, version pinning, and integration with BcContainerHelper.

## Links

- [ALCops extension on Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Arthurvdv.alcops-ado)
- [Extension source code on GitHub](https://github.com/ALCops/Analyzers/tree/main/azure-devops-extension)
- [Report issues](https://github.com/ALCops/Analyzers/issues)