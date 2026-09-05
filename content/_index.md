---
title: "ALCops.dev"
description: "A community driven collection of code analyzers for the AL programming language of Microsoft Dynamics 365 Business Central."
---

{{% blocks/cover title="You write the code, we write the rules" height="full" color="primary" %}}

Code analyzers for AL, built by the Business Central community
{.display-6}

<div style="margin-top: 7rem;">
{{% blocks/link-down color="info" %}}
</div>

{{% /blocks/cover %}}

{{% blocks/section color="white" type="row" %}}

<div class="col-12 text-center mb-4">
<h2>A collection of code analyzers</h2>
<p>Enable the ones that work for you, from coding style and preferences to project guidelines.</p>
</div>

{{% blocks/feature icon="fa-solid fa-microchip" title="PlatformCop" url="docs/analyzers/platformcop/" url_text="Browse PlatformCop rules" %}}
Does it work? Constructs the platform rejects, ignores, or executes differently from what the code says.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-gavel" title="ApplicationCop" url="docs/analyzers/applicationcop/" url_text="Browse ApplicationCop rules" %}}
Does it fit Business Central? How tables, fields, pages, enums, labels and permissions are modeled.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-magnifying-glass" title="LinterCop" url="docs/analyzers/lintercop/" url_text="Browse LinterCop rules" %}}
Is it maintainable? Procedure bodies that are valid but harder to read, change or keep fast than they need to be.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-book" title="DocumentationCop" url="docs/analyzers/documentationcop/" url_text="Browse DocumentationCop rules" %}}
Is it explained? Comments and XML documentation on public procedures and on constructs whose intent is not stated in code.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-align-left" title="FormattingCop" url="docs/analyzers/formattingcop/" url_text="Browse FormattingCop rules" %}}
Does it read the same everywhere? Casing, spacing, blank lines and ordering. Purely visual, never changes behavior.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-vial" title="TestAutomationCop" url="docs/analyzers/testautomationcop/" url_text="Browse TestAutomationCop rules" %}}
Will the test runner run it? Test codeunits only; silent on production code.
{{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="dark" type="row" %}}

<div class="col-12 text-center mb-4">
<h2>Where it runs</h2>
</div>

{{% blocks/feature icon="fa-solid fa-puzzle-piece" title="VS Code" url="docs/getting-started/vscode/" url_text="Install the extension" %}}
The extension downloads the analyzer build that matches your AL Language version and registers it with the compiler. Diagnostics show up as you type, and the status bar lets you switch individual analyzers on and off.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-gears" title="CI/CD pipelines" url="docs/getting-started/cicd/" url_text="Set up your pipeline" %}}
GitHub with AL-Go and Azure DevOps with the ALCops pipeline task both run the same analyzers on every build. A rule that shows a warning in the editor fails the build, or is reported, according to your ruleset.
{{% /blocks/feature %}}

{{% blocks/feature icon="fa-solid fa-robot" title="AI assistants" url="docs/getting-started/ai-tooling/" url_text="Connect the MCP server" %}}
The MCP server exposes the analyzers to Claude, Cursor and other assistants. An assistant can analyze a file, look up a rule and apply its code fix without leaving the conversation.
{{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="info" type="row text-center" %}}

<div class="col-12">

## Built in the open

ALCops is open source under the MIT license. Our rules are created collaboratively by developers and we welcome everyone to share their thoughts, ideas and improvements.

ALCops is a continuation of <a href="https://github.com/StefanMaron/BusinessCentral.LinterCop" target="_blank" rel="noopener">BusinessCentral.LinterCop</a> and this project wouldn't exist without the foundation built the community.<br>
A heartfelt thank you to every contributor who invested their time, ideas and code into the original LinterCop. Your work didn't end there, it lives on and grows further here in ALCops.

</div>

{{% /blocks/section %}}
