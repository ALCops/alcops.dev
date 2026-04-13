---
title: "CI/CD Pipelines"
linkTitle: "CI/CD"
weight: 40
type: docs
---

ALCops analyzers run anywhere the AL compiler runs. In CI/CD pipelines, the compiler is invoked either directly (`alc.exe /analyzer`) or through a helper tool like BcContainerHelper. The setup differs depending on your platform and tooling.

| Platform | Approach | Page |
|----------|----------|------|
| **GitHub** | AL-Go for GitHub (recommended) | [GitHub](github/) |
| **GitHub** | Custom GitHub Actions with BcContainerHelper or `alc.exe` | [GitHub](github/) |
| **Azure DevOps** | ALCops extension | [Azure DevOps](azure-devops/) |
| **Azure DevOps** | ALOps extension | [Azure DevOps](azure-devops/) |
| **Azure DevOps** | Custom pipelines with BcContainerHelper or `alc.exe` | [Azure DevOps](azure-devops/) |

All CI/CD approaches ultimately invoke the compiler with the `/analyzer` flag. If you are unfamiliar with the underlying mechanism, read [Command Line](../command-line/) first.
