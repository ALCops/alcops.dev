---
title: "Task Reference"
linkTitle: "Task Reference"
weight: 10
type: docs
---

## ALCopsDownloadAnalyzers@1

Download ALCops analyzer DLLs with automatic target framework detection.

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `detectUsing` | No | | Input for TFM auto-detection. Accepts a BC artifact URL, a local compiler path, a channel (`latest`, `preview`, `current`), or a specific version number. |
| `detectFrom` | No | *(auto)* | Force the detection source: `bc-artifact`, `marketplace`, `nuget-devtools`, or `compiler-path`. When omitted, the task infers the source from the format of `detectUsing`. |
| `tfm` | No | | Explicit target framework: `netstandard2.1`, `net8.0`, or `net10.0`. Skips detection entirely. |
| `version` | No | `latest` | ALCops package version to download: `latest`, `preview`, or a pinned version (e.g., `1.2.3`). |
| `outputPath` | No | `$(Build.SourcesDirectory)/.alcops` | Directory where analyzer DLLs are extracted. |

### Output variables

Reference these using the task's `name` attribute (e.g., `$(ALCopsDownload.version)`).

| Variable | Description |
|----------|-------------|
| `version` | The downloaded ALCops analyzer version. |
| `tfm` | The target framework that was used (detected or specified). |
| `outputDir` | Absolute path to the directory containing the extracted analyzer DLLs. |
| `files` | Semicolon-separated list of absolute paths to all analyzer DLLs. |

### Detection sources

#### `bc-artifact`

Reads the .NET version from a Business Central artifact manifest. Use when your pipeline already resolves a BC artifact URL (e.g., via `Get-BCArtifactUrl`).

**Input format**: A full BC artifact URL such as `https://bcartifacts.azureedge.net/sandbox/26.0.12345.0/us`

#### `marketplace`

Queries the Visual Studio Marketplace for the AL Language extension and inspects its compiler binary. Use when you compile with a VSIX downloaded from the marketplace.

**Input format**: `current` (latest stable) or `preview` (pre-release channel), or a specific extension version number.

#### `nuget-devtools`

Resolves the `Microsoft.Dynamics.BusinessCentral.Development.Tools` NuGet package and maps its version to a TFM. Use when you compile with `dotnet al` from the NuGet DevTools.

**Input format**: `latest` (latest stable), `preview` (including pre-releases), or a specific version number (e.g., `26.0.12345.0`).

#### `compiler-path`

Inspects a local directory containing `Microsoft.Dynamics.Nav.CodeAnalysis.dll` to determine the TFM from the binary metadata. Use when you already have a compiler folder on disk (e.g., from `New-BcCompilerFolder`).

**Input format**: An absolute or relative path to a directory. The task searches recursively for the CodeAnalysis DLL.

### Version channels

The `version` input controls which ALCops release is downloaded:

| Value | Behavior |
|-------|----------|
| `latest` | Latest stable release from NuGet (default). |
| `preview` | Latest pre-release (alpha/beta) from NuGet. Use to test upcoming rules. |
| `1.2.3` | A specific pinned version. Use for reproducible builds. |

### Example

```yaml
steps:
  - task: ALCopsDownloadAnalyzers@1
    name: ALCopsDownload
    displayName: ALCops - Download Analyzers
    inputs:
      detectUsing: "latest"
      detectFrom: "nuget-devtools"
      version: "latest"
      outputPath: "$(Build.SourcesDirectory)/.alcops"
```

After the task runs, the output variables are available:

```yaml
  - script: echo "Downloaded ALCops $(ALCopsDownload.version) for $(ALCopsDownload.tfm)"
  - script: echo "Analyzers at $(ALCopsDownload.outputDir)"
```

## Links

- [ALCops extension on Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Arthurvdv.alcops-ado)
- [Extension source code on GitHub](https://github.com/ALCops/azure-devops-extension)
- [Report issues](https://github.com/ALCops/azure-devops-extension/issues)
