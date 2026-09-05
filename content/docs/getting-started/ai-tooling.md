---
title: "MCP Server"
linkTitle: "MCP Server"
weight: 50
type: docs
---

The [ALCops MCP server](https://github.com/ALCops/mcp-server) brings AL code analysis to AI assistants through the [Model Context Protocol](https://modelcontextprotocol.io/). It lets Claude, Cursor, and other MCP-compatible clients analyze Business Central AL projects, browse rules, and apply code fixes — all without leaving the conversation.

## Install

Install the MCP server as a .NET global tool:

```shell
dotnet tool install -g ALCops.Mcp
```

## Configuration

Add the server to your MCP client configuration.

```json
{
    "mcpServers": {
        "alcops": {
            "command": "alcops-mcp"
        }
    }
}
```

## Available Tools

The MCP server exposes 4 tools (~1,020 tokens of schema overhead):

| Tool | Description |
|------|-------------|
| `analyze` | Run analyzers on an AL project or file. Returns diagnostics with severity, location, and code fix availability. |
| `list_rules` | List all available analyzer rules with metadata (ID, title, severity, category, cop). |
| `get_fixes` | Get available code fixes for a specific diagnostic at a location. |
| `apply_fix` | Apply a code fix and return the modified source — does **not** write to disk. |

## Analyzers

The server always includes ALCops' six built-in analyzers. Additionally, it can load **BC standard analyzers** (`${CodeCop}`, `${UICop}`, `${PerTenantExtensionCop}`, `${AppSourceCop}`) and **third-party analyzers** — auto-discovered from `al.codeAnalyzers` in `.vscode/settings.json`, or passed explicitly via the `analyzers` tool parameter.

## BC Development Tools Resolution

The server needs Microsoft BC Development Tools at runtime (not bundled due to licensing). On startup it searches, in order:

1. `BCDEVELOPMENTTOOLSPATH` environment variable
2. AL Language VS Code extension
3. Local cache (`~/.alcops/cache/devtools/`)
4. .NET global tools cache
5. Auto-download from NuGet (first run only)

Most developers have the AL Language extension installed, so **no extra setup is needed**.

## Other AI Integrations

{{% alert title="Under Investigation" color="warning" %}}
We are exploring additional integration paths for GitHub Copilot and other AI-assisted development tools. This page will be updated as new integrations become available.
{{% /alert %}}
