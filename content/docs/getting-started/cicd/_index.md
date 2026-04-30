---
title: "CI/CD Pipelines"
linkTitle: "CI/CD"
weight: 40
type: docs
---

ALCops analyzers run anywhere the AL compiler runs. In CI/CD pipelines, the compiler is invoked either directly (`alc` / `dotnet al`) or through a helper tool. The setup differs depending on your platform and tooling.

| Platform | Approach | Page |
|----------|----------|------|
| **Azure DevOps** | ALCops extension (recommended) | [Azure DevOps](azure-devops/) |
| **Azure DevOps** | ALOps extension + ALCops | [Azure DevOps](azure-devops/) |
| **Azure DevOps** | Custom pipelines with `alc` or `dotnet al` | [Azure DevOps](azure-devops/) |
| **Any CI** | `@alcops/core` CLI (npm) | [CLI](cli/) |
| **GitHub** | AL-Go for GitHub | [GitHub](github/) |
| **GitHub** | Custom GitHub Actions | [GitHub](github/) |

All approaches ultimately invoke the compiler with the `/analyzer` flag. If you are unfamiliar with the underlying mechanism, read [Command Line](../command-line/) first.

## Choosing an approach

**Azure DevOps**: Use the [ALCops extension](azure-devops/). It provides the `ALCopsDownloadAnalyzers` task that handles TFM detection and analyzer download in a single step. Works with ALOps, BcContainerHelper, NuGet DevTools, or direct compiler invocation.

**GitHub (AL-Go)**: Use the [AL-Go integration](github/). AL-Go has native support for custom code cops through its settings file.

**Any other CI system**: Use the [`@alcops/core` CLI](cli/). It runs on any platform with Node.js 20+ and outputs JSON for easy integration with shell scripts.
