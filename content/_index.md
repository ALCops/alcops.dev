---
title: "ALCops.dev"
description: "The reference for AL code analyzers in Business Central"
---

{{% blocks/cover title="ALCops.dev" height="full" color="primary" %}}

A community driven collection of code analyzers for the AL programming language of Microsoft Dynamics 365 Business Central.
{.display-6}

<div class="td-cta-buttons my-5">
  <a class="btn btn-lg btn-primary" href="docs/">Documentation</a>
  <a class="btn btn-lg btn-secondary" href="https://marketplace.visualstudio.com/items?itemName=Arthurvdv.alcops" target="_blank" rel="noopener">VS Code</a>
</div>

{{% blocks/link-down color="info" %}}

{{% /blocks/cover %}}

{{% blocks/lead color="white" %}}

**ALCops** provides a set of static analysis rules that enforce coding standards, architectural guidance, and best practices at code level. The rules are largely shaped and voted on by the community, translating real world experience and collective expertise into enforceable diagnostics.

{{% /blocks/lead %}}

{{% blocks/section color="dark" type="row" %}}

{{% blocks/feature icon="fa-solid fa-gavel" title="ApplicationCop" url="docs/analyzers/applicationcop/" %}}
Enforces design and coding standards for AL extensions, covering consistency, maintainability, and best practices.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-book" title="DocumentationCop" url="docs/analyzers/documentationcop/" %}}
Ensures proper XML documentation comments and structured inline documentation for public APIs.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-align-left" title="FormattingCop" url="docs/analyzers/formattingcop/" %}}
Checks for consistent code formatting, indentation, and style conventions across your codebase.
{{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="white" type="row" %}}

{{% blocks/feature icon="fa-solid fa-magnifying-glass" title="LinterCop" url="docs/analyzers/lintercop/" %}}
Analyzes code quality metrics, maintainability, and complexity beyond basic design standards.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-microchip" title="PlatformCop" url="docs/analyzers/platformcop/" %}}
Detects common mistakes and anti-patterns when working with the Business Central platform.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-vial" title="TestAutomationCop" url="docs/analyzers/testautomationcop/" %}}
Promotes best practices in AL test codeunits, test isolation, and test automation patterns.
{{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="info" type="row text-center" %}}

## AL coding standards, shaped by the community

6 specialized analyzers that catch bad patterns early, with fix guidance for every diagnostic.

{{% /blocks/section %}}