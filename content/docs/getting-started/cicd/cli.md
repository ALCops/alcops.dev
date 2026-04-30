---
title: "CLI"
linkTitle: "CLI"
weight: 30
type: docs
---

The [`@alcops/core`](https://www.npmjs.com/package/@alcops/core) npm package provides a command-line tool for TFM detection and analyzer download. It works on any platform with Node.js 20+ and in any CI system (Azure DevOps, GitHub Actions, GitLab CI, Jenkins, etc.).

The [ALCops Azure DevOps extension](azure-devops/) uses this package internally. The CLI gives you the same capabilities without depending on a marketplace extension.

## Installation

```bash
# Install globally
npm install -g @alcops/core

# Or run without installing
npx @alcops/core --help
```

## Commands

### `detect-tfm`

Detect the target framework moniker from a source:

```bash
# From VS Marketplace (latest stable AL Language)
alcops detect-tfm marketplace

# From VS Marketplace (pre-release channel)
alcops detect-tfm marketplace preview

# From NuGet DevTools (latest)
alcops detect-tfm nuget-devtools

# From NuGet DevTools (specific version)
alcops detect-tfm nuget-devtools 26.0.12345.0

# From a BC artifact URL
alcops detect-tfm bc-artifact "https://bcartifacts.azureedge.net/sandbox/26.0.12345.0/us"

# From a local compiler directory
alcops detect-tfm compiler-path ./path/to/compiler
```

Output (JSON on stdout):

```json
{
  "tfm": "net8.0",
  "source": "marketplace",
  "details": "AL Language extension v14.0.12345"
}
```

### `download`

Detect TFM and download analyzer DLLs in one step:

```bash
# Auto-detect from NuGet DevTools (latest)
alcops download --detect-using latest --output ./analyzers

# Auto-detect from a BC artifact URL
alcops download --detect-using "https://bcartifacts.azureedge.net/sandbox/26.0.12345.0/us" --output ./analyzers

# Force detection source
alcops download --detect-using preview --detect-from nuget-devtools --output ./analyzers

# Explicit TFM (skips detection)
alcops download --tfm net8.0 --output ./analyzers

# Specific ALCops version
alcops download --detect-using latest --output ./analyzers --version 1.0.0

# Verbose logging (logs on stderr)
alcops download --detect-using latest --output ./analyzers --verbose
```

Output (JSON on stdout):

```json
{
  "version": "1.0.0",
  "tfm": "net8.0",
  "outputDir": "/absolute/path/to/analyzers",
  "files": [
    "/absolute/path/to/analyzers/ALCops.ApplicationCop.dll",
    "/absolute/path/to/analyzers/ALCops.DocumentationCop.dll",
    "/absolute/path/to/analyzers/ALCops.FormattingCop.dll",
    "/absolute/path/to/analyzers/ALCops.LinterCop.dll",
    "/absolute/path/to/analyzers/ALCops.PlatformCop.dll",
    "/absolute/path/to/analyzers/ALCops.TestAutomationCop.dll",
    "/absolute/path/to/analyzers/ALCops.Common.dll"
  ]
}
```

## CLI options

| Option | Description |
|--------|-------------|
| `--output <dir>` | Required for `download`. Directory to extract analyzer DLLs into. |
| `--detect-using <input>` | TFM detection input (URL, path, channel, or version). |
| `--detect-from <source>` | Force detection source: `bc-artifact`, `marketplace`, `nuget-devtools`, `compiler-path`. |
| `--tfm <tfm>` | Explicit TFM, skips auto-detection. |
| `--version <ver>` | ALCops package version: `latest` (default), `preview`, or pinned. |
| `--verbose` | Enable debug logging on stderr. |
| `--help` | Show help. |

## Piping output

Logs go to stderr, JSON results go to stdout. This makes the CLI pipe-friendly:

```bash
TFM=$(alcops detect-tfm marketplace | jq -r '.tfm')
echo "Building with TFM: $TFM"
```

```bash
OUTPUT_DIR=$(alcops download --detect-using latest --output ./analyzers | jq -r '.outputDir')
alc /project:. /analyzer:$OUTPUT_DIR/ALCops.LinterCop.dll
```

## CI/CD examples

### Azure DevOps (without the extension)

Use the CLI as an alternative to the ALCops extension:

```yaml
steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'

  - pwsh: |
      $result = npx @alcops/core download --detect-using latest --output ./analyzers | ConvertFrom-Json
      Write-Host "Downloaded ALCops $($result.version) for $($result.tfm)"
      Write-Host "##vso[task.setvariable variable=alcopsDir]$($result.outputDir)"
    displayName: Download ALCops Analyzers

  - pwsh: |
      dotnet al compile `
          "/project:$(Build.SourcesDirectory)" `
          "/analyzer:$(alcopsDir)/ALCops.LinterCop.dll" `
          "/analyzer:$(alcopsDir)/ALCops.ApplicationCop.dll" `
          "/analyzer:$(alcopsDir)/ALCops.Common.dll"
    displayName: Compile
```

### GitHub Actions

```yaml
- name: Download ALCops Analyzers
  id: alcops
  run: |
    result=$(npx @alcops/core download --detect-using latest --output ./analyzers)
    echo "dir=$(echo "$result" | jq -r '.outputDir')" >> "$GITHUB_OUTPUT"

- name: Compile
  run: |
    dotnet al compile \
      "/project:." \
      "/analyzer:${{ steps.alcops.outputs.dir }}/ALCops.LinterCop.dll" \
      "/analyzer:${{ steps.alcops.outputs.dir }}/ALCops.ApplicationCop.dll" \
      "/analyzer:${{ steps.alcops.outputs.dir }}/ALCops.Common.dll"
```

### Generic shell script

```bash
#!/bin/bash
set -euo pipefail

# Download analyzers
result=$(npx @alcops/core download --detect-using latest --output ./analyzers)
output_dir=$(echo "$result" | jq -r '.outputDir')

# Build analyzer flags
analyzer_flags=""
for dll in "$output_dir"/ALCops.*.dll; do
  analyzer_flags="$analyzer_flags /analyzer:$dll"
done

# Compile
alc /project:. $analyzer_flags
```

## Programmatic API

The package also exports TypeScript functions for use as a library:

```typescript
import { download, createConsoleLogger } from '@alcops/core';

const logger = createConsoleLogger();
const result = await download({
  detectUsing: 'latest',
  output: './analyzers'
}, logger);

console.log(result.tfm);       // "net8.0"
console.log(result.outputDir); // "/absolute/path/to/analyzers"
```

## Links

- [npm package](https://www.npmjs.com/package/@alcops/core)
- [Source code on GitHub](https://github.com/ALCops/npm-package)
- [Report issues](https://github.com/ALCops/npm-package/issues)
