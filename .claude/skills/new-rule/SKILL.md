---
name: new-rule
description: Draft a complete documentation page for an analyzer rule from its C# source. Use when the user wants to document a rule, add a rule page, or says "new rule XX0000" / "document PC0042".
argument-hint: <rule-id>
---

# New rule page

Create a complete, publication-ready rule page for the rule ID given in `$ARGUMENTS`, deriving every piece of metadata from the analyzer's C# source and drafting the body according to `.claude/rules/rule-pages.md`. Read that file first; it is the contract for everything you produce here.

## Step 1 — Resolve the rule ID

- The rule ID must match `^[A-Z]{2}[0-9]{4}$` (e.g. `PC0042`). If `$ARGUMENTS` doesn't contain one, ask for it.
- Map the prefix to the analyzer: AC → ApplicationCop, DC → DocumentationCop, FC → FormattingCop, LC → LinterCop, PC → PlatformCop, TA → TestAutomationCop.
- If `content/docs/analyzers/<AnalyzerName>/<RULEID>.md` (or a `<RULEID>/` page bundle) already exists, stop and tell the user — suggest `/review-rule <RULEID>` instead.

## Step 2 — Locate the analyzer source

Try the sibling repository first: `../Analyzers/src/ALCops.<AnalyzerName>/` relative to this repo's root.

If that directory doesn't exist, ask the user for the path to their Analyzers clone, or to paste the analyzer class, the descriptor declaration, and (if any) the code fix provider. Never invent metadata when the source is unavailable.

## Step 3 — Extract metadata

All commands below run from the cop project directory `../Analyzers/src/ALCops.<AnalyzerName>/`.

1. **Rule name** — find the property whose value is the rule ID in `DiagnosticIds.cs`:
   `grep -E '= "<RULEID>"' DiagnosticIds.cs` → e.g. `public static readonly string EditableFlowField = "PC0001";` gives rule name `EditableFlowField`.
2. **Descriptor** — in `DiagnosticDescriptors.cs`, read the `DiagnosticDescriptor <RuleName>` declaration:
   - `defaultSeverity: DiagnosticSeverity.X` → `severity` (`Error`, `Warning`, `Info`, `Hidden`)
   - `category: Category.X` → `category` (`Design`, `Naming`, `Style`, `Usage`, `Performance`, `Security`)
   - `isEnabledByDefault` → the Enabled column in `_index.md`
3. **Code fix** — find the provider with `grep -rl '<RuleName>' CodeFixes/` (file names vary: `<RuleName>.cs`, `<RuleName>CodeFixProvider.cs`, ...):
   - No match → `codeAction = false` (omit `codeActionType`, `supportsFixAll`).
   - Match → confirm its `FixableDiagnosticIds` includes this rule, then `codeAction = true`; `Kind => CodeActionKind.QuickFix|Refactor` → `codeActionType`. For `supportsFixAll`, trace the value actually assigned — it is usually a constructor flag, so look at the registration call site (e.g. `CreateCodeAction(..., generateFixAll: true)`), not just the property declaration.
4. **Obsolete handling** — in `Analyzers/<RuleName>.cs`, an early-return guard calling `IsObsolete()` (or checks of `IsObsoletePending`) → `ignoresObsoletePending = true`; otherwise `false`.
5. **Strings** — in `ALCops.<AnalyzerName>Analyzers.resx`, read the `<RuleName>Title`, `<RuleName>MessageFormat`, and `<RuleName>Description` values. The Title becomes the front matter `title` (with ` (<RULEID>)` appended) and the `_index.md` Title column.

## Step 4 — Understand the rule's behavior

Don't write the body from the rule title alone:

- Read the full analyzer implementation `Analyzers/<RuleName>.cs`: what syntax/symbols it inspects, what conditions raise the diagnostic, what conditions bail out early.
- Read the test fixtures in `../Analyzers/src/ALCops.<AnalyzerName>.Test/Rules/<RuleName>/`:
  - `HasDiagnostic/*.al` — exact constructs that trigger; basis for the "bad" example and `### When the diagnostic is reported`.
  - `NoDiagnostic/*.al` — exclusions; basis for `### When the diagnostic is NOT reported`.
  - `HasFix/*.al` — what the code fix changes; basis for `### Code fix`.
  - Not every rule has fixtures (e.g. TA0001). When the folder is missing, derive the trigger conditions from the analyzer implementation alone and say so in the final report.
- If a code fix exists, read `CodeFixes/<RuleName>.cs` to describe precisely what the fix edits.

## Step 5 — Draft the page

Write `content/docs/analyzers/<AnalyzerName>/<RULEID>.md` following `.claude/rules/rule-pages.md` exactly: TOML front matter with the full `[params]` table, the Why section (three beats: mechanism → concrete consequence → fix in one sentence), `### Example` with the diagnostic comment `// <Title> [<RULEID>]` in the bad code, fix text plus fixed code, and the optional sections only where the analyzer logic supports them.

- Adapt fixture code into minimal, realistic AL (`Record Customer`, `Codeunit "Sales-Post"`) rather than copying `MyTable` fixtures verbatim.
- Only state runtime behavior you can ground in the analyzer source, the fixtures, or the resx description. Mark anything you inferred as needing review in the final report — never present a guess as fact in the page.
- For neighboring style reference, read one or two recent pages in the same analyzer folder.

## Step 6 — Update the index

Add a row for the rule to `content/docs/analyzers/<AnalyzerName>/_index.md`, keeping the table sorted by ID:

`| [<RULEID>](<ruleid lowercase>/) | <Title without ID> | <Severity> | <✓ if enabled> | <✓ if code fix> |`

## Step 7 — Verify

- Run `hugo --quiet` (or `hugo --minify`) from the repo root and surface any errors or warnings.
- Confirm the new page and the index row agree on title, severity, and code fix.

## Step 8 — Report

End with a short report: a table of the extracted metadata with the source file each value came from, the fixtures consulted, and a list of statements in the page the user should fact-check (anything not directly grounded in source).
