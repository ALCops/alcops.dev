---
title: "Troubleshooting"
linkTitle: "Troubleshooting"
weight: 90
type: docs
---

Common issues when running ALCops analyzers in CI/CD pipelines.

## Analyzer DLLs fail to load

**Symptom**: The compiler runs without errors but ALCops diagnostics are missing, or you see warnings like "Could not load analyzer assembly."

**Cause**: The analyzer DLLs were built for a different .NET target framework than the compiler expects.

**Fix**: Ensure the TFM used to download analyzers matches your compiler version. Use auto-detection (`detectUsing`) rather than hardcoding a TFM. If you recently updated your BC version or compiler tools, re-run the pipeline to pick up the new TFM.

Verify the detected TFM in the task output:

```yaml
- script: echo "TFM is $(ALCopsDownload.tfm)"
```

## Wrong target framework detected

**Symptom**: The task downloads `netstandard2.1` analyzers when you need `net8.0`, or vice versa.

**Cause**: The `detectUsing` input does not match your actual compiler source. For example, using `detectUsing: "latest"` with NuGet DevTools while your pipeline compiles with an older BC artifact.

**Fix**: Match the detection source to your compiler source:

| You compile with... | Use `detectFrom` | Use `detectUsing` |
|---------------------|-----------------|-------------------|
| `dotnet al` (NuGet DevTools) | `nuget-devtools` | `latest` or `preview` |
| BC artifact VSIX (`alc`) | `bc-artifact` | Your `$(artifactUrl)` variable |
| VS Marketplace VSIX (`alc`) | `marketplace` | `current` or `preview` |
| `New-BcCompilerFolder` | `compiler-path` | Your `$(compilerFolder)` variable |

## Permission denied on Linux/macOS

**Symptom**: `alc` binary fails with "Permission denied" after extracting the VSIX.

**Cause**: ZIP extraction strips execute permissions from binaries.

**Fix**: Set the execute bit after extraction:

```powershell
if ($IsLinux -or $IsMacOS) {
    chmod +x "$alcBinPath/alc"
}
```

The ALCops analyzer DLLs themselves do not need execute permissions. This issue only affects the AL compiler binary.

## "Analyzer not found" in compiler output

**Symptom**: The compiler reports that an analyzer file does not exist at the specified path.

**Cause**: The path passed to `/analyzer:` does not resolve correctly. Common reasons:
- Using relative paths in a context where the working directory is different
- Variable expansion failed (empty variable)

**Fix**: Use the `$(ALCopsDownload.outputDir)` output variable for absolute paths. Check that the task name matches your variable reference:

```yaml
- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload        # ← This name must match
  inputs:
    detectUsing: "latest"

- script: ls "$(ALCopsDownload.outputDir)"   # Verify files exist
```

For ALOps v1/v2, use relative paths from the source directory (e.g., `.alcops/ALCops.LinterCop.dll`) and set `additionalprobingpaths: '.alcops'`.

## Debug logging

### Azure DevOps extension

Enable verbose logging by setting the pipeline variable `System.Debug` to `true`:

```yaml
variables:
  System.Debug: true
```

The task emits detection details, HTTP requests, and version resolution steps in debug output.

### CLI

Use the `--verbose` flag:

```bash
alcops download --detect-using latest --output ./analyzers --verbose
```

Verbose output goes to stderr, so it does not interfere with JSON output on stdout.

## Version mismatch between detection and compilation

**Symptom**: TFM detection succeeds but the analyzers produce unexpected results or fail at runtime.

**Cause**: The detection source resolves a different version than the one actually used for compilation. For example, `detectUsing: "latest"` resolves the newest NuGet DevTools version, but your pipeline has an older version pinned in a `dotnet tool install` command.

**Fix**: Keep detection and compilation in sync:
- If you pin a specific DevTools version in your pipeline, pass that same version to `detectUsing`.
- If you use `latest` for detection, also use `latest` in your tool installation.

## Pipeline caching

The `ALCopsDownloadAnalyzers` task downloads from NuGet on every run. If you want to cache the result:

```yaml
- task: Cache@2
  inputs:
    key: 'alcops | "$(Agent.OS)" | latest'
    path: '$(Build.SourcesDirectory)/.alcops'
    cacheHitVar: ALCOPS_CACHE_HIT

- task: ALCopsDownloadAnalyzers@1
  name: ALCopsDownload
  condition: ne(variables.ALCOPS_CACHE_HIT, 'true')
  inputs:
    detectUsing: "latest"
    outputPath: "$(Build.SourcesDirectory)/.alcops"
```

{{% alert title="Cache invalidation" color="warning" %}}
If you use `latest` as the cache key, the cache will not invalidate when a new ALCops version is published. Consider including a date or version in the key if you want automatic updates.
{{% /alert %}}
