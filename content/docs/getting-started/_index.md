---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 10
type: docs
---

ALCops is a community-driven collection of code analyzers for the AL programming language of Microsoft Dynamics 365 Business Central. Whether you are setting up ALCops for the first time or integrating it into an existing workflow, the guides below will help you get started.

New to ALCops? Start with the [Quick Start](quick-start/) to see what the analyzers catch before diving into setup.

### Setup

Pick the approach that matches your development workflow:

- [Visual Studio Code](vscode/) — install the extension and get diagnostics in your editor (recommended)
- [Command Line](command-line/) — use `alc.exe` with analyzer DLLs for scripts and automation

### CI/CD Pipelines

Run ALCops as part of your build to catch issues before they reach production:

- [AL-Go for GitHub](cicd/github/) — built-in support for custom code analyzers
- [Azure DevOps](cicd/azure-devops/) — ALOps, custom pipelines, and the upcoming ALCops extension

### AI Tooling

- [MCP Server](ai-tooling/) — let AI assistants like Claude analyze your AL code through the Model Context Protocol

### Configuration

After installation, see [Configuration](configuration/) to learn how to enable, disable, or adjust individual rules using `.ruleset.json`, `alcops.json` and `#pragma` directives.

### Migrating from LinterCop?

If you are coming from BusinessCentral.LinterCop, see [LinterCop Migration](../lintercop-migration/) for a complete diagnostic mapping between LinterCop and ALCops rules.