---
title: "Downloading Analyzers"
linkTitle: "Downloading Analyzers"
weight: 40
type: docs
---

{{% alert title="Note" %}}
Most users do not need to download DLLs manually. The [VS Code extension](getting-started/vscode/), [AL-Go helper](getting-started/cicd/github/), and the upcoming Azure DevOps extension handle this automatically.
{{% /alert %}}

This page explains how to obtain ALCops analyzer DLLs directly from NuGet for use with `alc.exe` or custom build pipelines.

## NuGet Package

ALCops distributes analyzer DLLs through NuGet. The package is a container for the DLL files — it is not a standard .NET library or tool reference.

**Package:** [ALCops.Analyzers on NuGet](https://www.nuget.org/packages/ALCops.Analyzers)

## Download and Extract

Use the NuGet CLI or download the package manually:

```PowerShell
# Download the package
nuget install ALCops.Analyzers -OutputDirectory ./packages
```

Alternatively, download the `.nupkg` file directly from the NuGet website and rename it to `.zip` to extract its contents.

## Target Framework Selection

The NuGet package contains DLLs for two target frameworks. Choose the correct one based on your AL Language extension version:

| Target Framework | AL Language Version | Folder in Package |
|------------------|-------------------|-------------------|
| **net8.0** | v16.0 and later (current) | `lib/net8.0/` |
| **netstandard2.1** | Below v16.0 | `lib/netstandard2.1/` |

Copy all DLLs from the appropriate framework folder to your analyzer directory.
