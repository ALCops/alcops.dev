---
title: "Quick Start"
linkTitle: "Quick Start"
weight: 1
type: docs
---

ALCops is a community-driven collection of code analyzers for the AL programming language. It catches common mistakes, enforces best practices, and helps maintain consistent code quality across your Business Central projects.

## What ALCops Catches

Here is a small sample of what ALCops detects. Browse the full [rules reference](../../analyzers/) for the complete list.

### Missing Caption on user-facing controls (AC0011)

User-facing controls need explicit captions for a consistent and localizable UI.

{{< tabpane >}}
{{< tab header="Before" lang="al" >}}
action(MyAction)
{
    trigger OnAction()
    begin
    end;
}
{{< /tab >}}
{{< tab header="After" lang="al" >}}
action(MyAction)
{
    Caption = 'My Action';

    trigger OnAction()
    begin
    end;
}
{{< /tab >}}
{{< /tabpane >}}

### Primary key field missing NotBlank (AC0002)

Single-field primary keys of type Code or Text must set `NotBlank` to prevent empty key values.

{{< tabpane >}}
{{< tab header="Before" lang="al" >}}
table 50100 MyTable
{
    fields
    {
        field(1; "No."; Code[20]) { }
    }
    keys
    {
        key(PK; "No.") { Clustered = true; }
    }
}
{{< /tab >}}
{{< tab header="After" lang="al" >}}
table 50100 MyTable
{
    fields
    {
        field(1; "No."; Code[20])
        {
            NotBlank = true;
        }
    }
    keys
    {
        key(PK; "No.") { Clustered = true; }
    }
}
{{< /tab >}}
{{< /tabpane >}}

## Ready to Set Up?

Choose the approach that fits your workflow:

- **[VS Code](../vscode/)** — Install the extension and start getting diagnostics immediately (recommended for development)
- **[Command Line](../command-line/)** — Use `alc.exe` with analyzer DLLs directly
- **[CI/CD Pipelines](../cicd/)** — Run ALCops in GitHub Actions or Azure DevOps builds
- **[MCP Server](../ai-tooling/)** — Let AI assistants analyze your AL code through the Model Context Protocol

Already using BusinessCentral.LinterCop? See [LinterCop Migration](../../lintercop-migration/) for a complete diagnostic mapping.
