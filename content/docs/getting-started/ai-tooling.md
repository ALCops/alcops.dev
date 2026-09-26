---
title: "MCP Server"
linkTitle: "MCP Server"
weight: 50
type: docs
---

The [ALCops MCP server](https://github.com/ALCops/mcp-server) gives AI assistants — Claude Code, GitHub Copilot, Cursor, Codex and any other [MCP](https://modelcontextprotocol.io/) client — the ALCops code fixes and rule lookup, and proxies Microsoft's own AL MCP server (`almcp`) so compiling, diagnostics, symbols, publishing and tests come through the same connection. One server entry in your client, everything an assistant needs to fix AL code.

## Prerequisites

- [.NET 10](https://dotnet.microsoft.com/download/dotnet/10.0) SDK or runtime.
- Microsoft BC Development Tools **v18.0 or later**. The recommended way to get them is the dotnet tool:

  ```shell
  dotnet tool install -g Microsoft.Dynamics.BusinessCentral.Development.Tools
  ```

  If you already have the [AL Language](https://marketplace.visualstudio.com/items?itemName=ms-dynamics-smb.al) VS Code extension, that install is optional — the server detects the extension's tools automatically.

v17 and earlier are not supported. The server checks the dotnet tool store before the AL extension, takes the newest version it finds there, and logs which directory it used; `--devtools-path` overrides the search. See [how the server finds the tools](https://github.com/ALCops/mcp-server#bc-devtools-resolution) for the full probe order.

## Install

```shell
dotnet tool install -g ALCops.Mcp
```

Upgrade later with `dotnet tool update -g ALCops.Mcp`.

## Connect your assistant

The server speaks MCP over stdio, so every client is a one-line entry.

{{< tabpane persist=false >}}
{{< tab header="Claude Code" lang="shell" >}}
claude mcp add --scope project alcops -- alcops-mcp

# writes .mcp.json:
# { "mcpServers": { "alcops": { "type": "stdio", "command": "alcops-mcp" } } }
{{< /tab >}}
{{< tab header="VS Code / Copilot" lang="json" >}}
{
  "servers": {
    "alcops": {
      "type": "stdio",
      "command": "alcops-mcp"
    }
  }
}
{{< /tab >}}
{{< tab header="Cursor" lang="json" >}}
{
  "mcpServers": {
    "alcops": {
      "command": "alcops-mcp"
    }
  }
}
{{< /tab >}}
{{< tab header="Codex" lang="toml" >}}
[mcp_servers.alcops]
command = "alcops-mcp"
{{< /tab >}}
{{< /tabpane >}}

Claude Code writes the entry to `.mcp.json` in the project. VS Code reads `.vscode/mcp.json` — note that its top-level key is `servers`, not `mcpServers`. Cursor reads `.cursor/mcp.json` for a single project or `~/.cursor/mcp.json` globally, and Codex reads `~/.codex/config.toml`, which `codex mcp add alcops -- alcops-mcp` writes for you.

Start the client from the folder that holds your AL project, or point the server at it with `--projects`. The server discovers `app.json` downward from the working directory, reads that project's `.vscode/settings.json`, and starts `almcp` with the same configuration.

## What the assistant gets

Four tools are served by ALCops itself:

| Tool | Description |
|------|-------------|
| `list_rules` | List analyzer rules with metadata (ID, title, severity, category, cop). |
| `get_fixes` | Get available code fixes for a diagnostic at a location. |
| `apply_fix` | Apply a code fix. Writes the fixed content to the file on disk. |
| `apply_fix_all` | Apply a fix to every occurrence of a rule across a project or file (like VS Code's "Fix all in workspace"). Writes to disk unless `dryRun` is set. |

Alongside them come Microsoft's `al_*` tools, proxied from `almcp`: `al_compile`, `al_build`, `al_getdiagnostics`, `al_symbolsearch`, `al_publish`, `al_run_tests`, translations, object IDs and more. The exact set depends on your installed BC Development Tools version; the [README](https://github.com/ALCops/mcp-server#proxied-from-microsofts-almcp) lists them all. If your client already registers `almcp` itself, start the server with `--no-proxy` so the `al_*` tools do not show up twice.

## Analyzers

Nothing is bundled. The server loads exactly the analyzers your project configures in `.vscode/settings.json` — ALCops cops and BC's standard cops through `al.codeAnalyzers`, severities through `al.ruleSetPath`, symbols through `al.packageCachePath` — and passes the same configuration to `almcp`, so `al_compile` and `get_fixes` agree about which rules run and which are suppressed.

```json
{
  "al.codeAnalyzers": [
    "${CodeCop}",
    "${analyzerFolder}ALCops.LinterCop.dll"
  ]
}
```

The [VS Code](../vscode/#manual-setup-without-the-extension) page shows the full `al.codeAnalyzers` list, and [Configuration](../configuration/#ruleset-files-rulesetjson) covers rulesets and `alcops.json`.

## Recommended agent workflow

Drop this into your `AGENTS.md` or `CLAUDE.md` so the assistant uses the tools in the right order:

```markdown
After changing AL code:

1. Run `al_compile` with `onlyErrors: false` and fix any errors.
2. For every ALCops warning (rule IDs like LC0020, AC0012, ...), call `get_fixes` at its
   location and apply the fix with `apply_fix`; use `apply_fix_all` when the same rule
   repeats across the project.
3. Re-run `al_compile` until it is clean.
```

{{% alert title="al_compile hides warnings by default" color="warning" %}}
`al_compile` defaults to `onlyErrors: true`, and nearly every ALCops rule is a *warning*. Pass `onlyErrors: false` or the assistant sees no cop diagnostics at all.
{{% /alert %}}
