---
title: "Examples"
linkTitle: "Examples"
weight: 20
type: docs
---

Each example shows the ALCops-relevant steps. Full working pipelines are available in the [demo repository](https://dev.azure.com/Arthurvdv/ALCops).

All examples use the same pattern:

1. **Download analyzers** with `ALCopsDownloadAnalyzers@1`
2. **Pass DLLs to the compiler** via `/analyzer` flags or your build tool's analyzer parameter

## NuGet DevTools (`dotnet al`)

Cross-platform. Uses the BC Development Tools from NuGet for compilation. No Docker, no BC artifacts required for the compiler itself.

```yaml
- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload
  displayName: ALCops - Download Analyzers
  inputs:
    detectUsing: "latest"
    detectFrom: "nuget-devtools"

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

[Full pipeline](https://dev.azure.com/Arthurvdv/ALCops/_git/ALCops?path=/.devops/NuGetDevTools.yml)

## BC Artifact URL (`alc`)

Cross-platform. Downloads BC artifacts, extracts the ALLanguage VSIX, and compiles with the native `alc` binary.

```yaml
- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload
  displayName: ALCops - Download Analyzers
  inputs:
    detectUsing: "$(artifactUrl)"
    detectFrom: "bc-artifact"

- pwsh: |
    $alcopsPath = "$(ALCopsDownload.outputDir)"

    $compileArgs = @(
        "/project:$(Build.SourcesDirectory)",
        "/analyzer:$alcopsPath/ALCops.ApplicationCop.dll",
        "/analyzer:$alcopsPath/ALCops.DocumentationCop.dll",
        "/analyzer:$alcopsPath/ALCops.FormattingCop.dll",
        "/analyzer:$alcopsPath/ALCops.LinterCop.dll",
        "/analyzer:$alcopsPath/ALCops.PlatformCop.dll",
        "/analyzer:$alcopsPath/ALCops.TestAutomationCop.dll",
        "/analyzer:$alcopsPath/ALCops.Common.dll"
    )

    Push-Location -Path "$(alcBinPath)"
    & "./alc" $compileArgs
    Pop-Location
```

[Full pipeline](https://dev.azure.com/Arthurvdv/ALCops/_git/ALCops?path=/.devops/BcArtifact.yml)

## VS Marketplace (`alc`)

Cross-platform. Downloads the AL Language extension VSIX directly from the VS Marketplace. No BcContainerHelper required.

```yaml
- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload
  displayName: ALCops - Download Analyzers
  inputs:
    detectUsing: "current"
    detectFrom: "marketplace"

- pwsh: |
    $alcopsPath = "$(ALCopsDownload.outputDir)"

    $compileArgs = @(
        "/project:$(Build.SourcesDirectory)",
        "/analyzer:$alcopsPath/ALCops.ApplicationCop.dll",
        "/analyzer:$alcopsPath/ALCops.DocumentationCop.dll",
        "/analyzer:$alcopsPath/ALCops.FormattingCop.dll",
        "/analyzer:$alcopsPath/ALCops.LinterCop.dll",
        "/analyzer:$alcopsPath/ALCops.PlatformCop.dll",
        "/analyzer:$alcopsPath/ALCops.TestAutomationCop.dll",
        "/analyzer:$alcopsPath/ALCops.Common.dll"
    )

    Push-Location -Path "$(alcBinPath)"
    & "./alc" $compileArgs
    Pop-Location
```

[Full pipeline](https://dev.azure.com/Arthurvdv/ALCops/_git/ALCops?path=/.devops/Marketplace.yml)

## BcContainerHelper (`Compile-AppWithBcCompilerFolder`)

Cross-platform. Uses `New-BcCompilerFolder` and `Compile-AppWithBcCompilerFolder` from BcContainerHelper. Detects TFM from the compiler folder on disk.

```yaml
- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload
  displayName: ALCops - Download Analyzers
  inputs:
    detectUsing: "$(compilerFolder)"
    detectFrom: "compiler-path"

- pwsh: |
    Import-Module BcContainerHelper -Force -DisableNameChecking

    $customCops = @(
        "$(ALCopsDownload.outputDir)/ALCops.ApplicationCop.dll",
        "$(ALCopsDownload.outputDir)/ALCops.DocumentationCop.dll",
        "$(ALCopsDownload.outputDir)/ALCops.FormattingCop.dll",
        "$(ALCopsDownload.outputDir)/ALCops.LinterCop.dll",
        "$(ALCopsDownload.outputDir)/ALCops.PlatformCop.dll",
        "$(ALCopsDownload.outputDir)/ALCops.TestAutomationCop.dll",
        "$(ALCopsDownload.outputDir)/ALCops.Common.dll"
    )

    Compile-AppWithBcCompilerFolder `
        -compilerFolder "$(compilerFolder)" `
        -appProjectFolder "$(Build.SourcesDirectory)" `
        -appSymbolsFolder "$(Build.SourcesDirectory)/.alpackages" `
        -appOutputFolder "$(Build.ArtifactStagingDirectory)" `
        -EnableCodeCop `
        -EnableUICop `
        -EnablePerTenantExtensionCop `
        -CustomCodeCops $customCops `
        -AzureDevOps
```

[Full pipeline](https://dev.azure.com/Arthurvdv/ALCops/_git/ALCops?path=/.devops/BcContainerHelper-CompilerPath.yml)

## ALOps v1 (Docker)

Windows only. Uses ALOps Docker tasks to run a BC container and compile inside it.

```yaml
- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload
  displayName: ALCops - Download Analyzers
  inputs:
    detectUsing: "$(artifactUrl)"
    detectFrom: "bc-artifact"
    outputPath: "$(Build.SourcesDirectory)/.alcops"

- task: ALOpsAppCompiler@1
  displayName: ALOps - Compile Extension
  inputs:
    usedocker: true
    targetproject: 'app.json'
    al_analyzer: CodeCop,UICop,PerTenantExtensionCop,.alcops/ALCops.ApplicationCop.dll,.alcops/ALCops.DocumentationCop.dll,.alcops/ALCops.FormattingCop.dll,.alcops/ALCops.LinterCop.dll,.alcops/ALCops.PlatformCop.dll,.alcops/ALCops.TestAutomationCop.dll,.alcops/ALCops.Common.dll
    additionalprobingpaths: '.alcops'
```

ALOps v1 uses relative paths from the source directory. The `additionalprobingpaths` input tells the container where to find the DLLs.

[Full pipeline](https://dev.azure.com/Arthurvdv/ALCops/_git/ALCops?path=/.devops/ALOpsV1.yml)

## ALOps v2 (No Docker)

Windows only. Uses ALOps with the marketplace VSIX compiler, no container.

```yaml
- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload
  displayName: ALCops - Download Analyzers
  inputs:
    detectUsing: "current"
    detectFrom: "marketplace"
    outputPath: "$(Build.SourcesDirectory)/.alcops"

- task: ALOpsAppCompiler@2
  displayName: ALOps - Compile Extension
  inputs:
    alternativevsixurl: 'Latest'
    alcodeanalyzer: CodeCop,UICop,PerTenantExtensionCop,ALCops.ApplicationCop.dll,ALCops.DocumentationCop.dll,ALCops.FormattingCop.dll,ALCops.LinterCop.dll,ALCops.PlatformCop.dll,ALCops.TestAutomationCop.dll,ALCops.Common.dll
    additionalprobingpaths: '.alcops'
```

[Full pipeline](https://dev.azure.com/Arthurvdv/ALCops/_git/ALCops?path=/.devops/ALOpsV2.yml)

## ALOps v3 (NuGet altool)

Windows only. Uses ALOps with the NuGet DevTools (`altool.exe`), no container.

```yaml
- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload
  displayName: ALCops - Download Analyzers
  inputs:
    detectUsing: "preview"
    detectFrom: "nuget-devtools"
    outputPath: "$(Build.SourcesDirectory)/.alcops"

- task: ALOpsAppCompiler@3
  displayName: ALOps - Compile Extension
  inputs:
    compilation_mode: Serial
    alcodeanalyzer: CodeCop,UICop,PerTenantExtensionCop,$(ALCopsDownload.outputDir)/ALCops.ApplicationCop.dll,$(ALCopsDownload.outputDir)/ALCops.DocumentationCop.dll,$(ALCopsDownload.outputDir)/ALCops.FormattingCop.dll,$(ALCopsDownload.outputDir)/ALCops.LinterCop.dll,$(ALCopsDownload.outputDir)/ALCops.PlatformCop.dll,$(ALCopsDownload.outputDir)/ALCops.TestAutomationCop.dll,$(ALCopsDownload.outputDir)/ALCops.Common.dll
```

ALOps v3 in serial mode supports absolute paths, so use the `$(ALCopsDownload.outputDir)` output variable directly.

[Full pipeline](https://dev.azure.com/Arthurvdv/ALCops/_git/ALCops?path=/.devops/ALOpsV3.yml)

## See also

- [ALOps documentation](https://marketplace.visualstudio.com/items?itemName=Hodor.hodor-alops)
- [BcContainerHelper documentation](https://github.com/microsoft/navcontainerhelper)
- [BC Development Tools NuGet package](https://www.nuget.org/packages/Microsoft.Dynamics.BusinessCentral.Development.Tools)
- [Demo repository with all pipelines](https://dev.azure.com/Arthurvdv/ALCops)
