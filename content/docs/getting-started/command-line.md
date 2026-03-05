---
title: "Command Line"
linkTitle: "Command Line"
weight: 20
type: docs
---

ALCops analyzers work with the AL compiler (alc.exe) through the standard `/analyzer` flag. This is the foundation that all other integration methods build on.

## AL Compiler (alc.exe)

Pass one or more analyzer DLLs to the compiler using the `/analyzer` flag:

```PowerShell
alc.exe /project:"C:\Source\MyApp" /packagecachepath:"C:\Source\MyApp\.alpackages" /analyzer:"C:\Analyzers\ALCops.ApplicationCop.dll" /analyzer:"C:\Analyzers\ALCops.PlatformCop.dll"
```

Each `/analyzer` flag accepts a full path to a single DLL. Repeat the flag for each analyzer you want to include.

{{% alert title="Important" %}}
When running from the command line, you must also include `ALCops.Common.dll` in the `customCodeCops` array. This shared dependency is required by all ALCops analyzers.

If you are not loading any Microsoft code analyzers (CodeCop, UICop, etc.), you must also place `Microsoft.Dynamics.Nav.Analyzers.Common.dll` alongside the ALCops DLLs.
{{% /alert %}}

## Obtaining the DLLs

ALCops distributes analyzer DLLs through NuGet. Download the package from the [NuGet feed](https://www.nuget.org/packages/ALCops.Analyzers) and extract the DLLs for the correct target framework.

For detailed download instructions, target framework selection, and package contents, see [Downloading Analyzers](../../downloading-analyzers/).