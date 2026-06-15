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
   - No match → `codeAction = false`.
   - Match → confirm its `FixableDiagnosticIds` includes this rule, then `codeAction = true`.
4. **Obsolete handling** — in `Analyzers/<RuleName>.cs`, an early-return guard calling `IsObsolete()` (or checks of `IsObsoletePending`) → `ignoreObsolete = true`; otherwise `false`.
5. **Strings** — in `ALCops.<AnalyzerName>Analyzers.resx`, read the `<RuleName>Title`, `<RuleName>MessageFormat`, and `<RuleName>Description` values. The Title becomes the front matter `title` (without the rule ID — the ID goes in `[params].id`) and the `_index.md` Title column.

## Step 4 — Understand the rule's behavior

Don't write the body from the rule title alone:

- Read the full analyzer implementation `Analyzers/<RuleName>.cs`: what syntax/symbols it inspects, what conditions raise the diagnostic, what conditions bail out early.
- Read the test fixtures in `../Analyzers/src/ALCops.<AnalyzerName>.Test/Rules/<RuleName>/`:
  - `HasDiagnostic/*.al` — exact constructs that trigger; basis for the "bad" example and `### When the diagnostic is reported`.
  - `NoDiagnostic/*.al` — exclusions the analyzer deliberately skips. Use these to inform `### Exception` when the exclusion is surprising or non-obvious (e.g. temporary records, specific field names). Do not create a "When the diagnostic is NOT reported" section. Also mine these for *alternative idiomatic fixes*: a NoDiagnostic fixture often shows a second valid way to write the code that the fixed example should mention.
  - `HasFix/*.al` — what the code fix changes; basis for `### Code fix`.
  - Not every rule has fixtures (e.g. TA0001). When the folder is missing, derive the trigger conditions from the analyzer implementation alone and say so in the final report.
- If a code fix exists, read `CodeFixes/<RuleName>.cs` to describe precisely what the fix edits.

## Step 5 — Draft the page

First decide the page tier per `.claude/rules/rule-pages.md` (Body structure): standard (one example pair explains the fix) or concept-heavy (a metric, methodology, or platform machinery must be taught first — free-form outline, only front matter + Why + one example pair mandatory).

Write `content/docs/analyzers/<AnalyzerName>/<RULEID>.md` following `.claude/rules/rule-pages.md` exactly: TOML front matter with the full `[params]` table, the Why section (three beats: developer intent → one concrete named failure → fix in one sentence), `### Example` with the diagnostic comment `// <Title> [<RULEID>]` in the bad code, fix text plus fixed code, and the optional sections only where the analyzer logic supports them.

- Open from the developer's intent — what the reader was trying to do when they wrote the flagged code — and name exactly one concrete failure (the stale field, the runtime error, the generated SQL). Never a categorical consequence list; the Voice exemplars in the conventions file show the difference.
- When the fixtures or the platform offer more than one idiomatic fix, show each in the fixed example with a sentence on when to pick which.
- If the recommended fix has a cost (performance, verbosity), state it plainly next to the recommendation.
- Use the Hugo `{{</* highlight al "hl_lines=N" */>}}` shortcode for all code examples. In the "bad" example, highlight the line(s) where the diagnostic is raised. In the "fixed" example, highlight the line(s) that changed. Line numbers count from the first line inside the shortcode.
- Adapt fixture code into minimal, realistic AL (`Record Customer`, `Codeunit "Sales-Post"`) rather than copying `MyTable` fixtures verbatim.
- Only state runtime behavior you can ground in the analyzer source, the fixtures, or the resx description. Mark anything you inferred as needing review in the final report — never present a guess as fact in the page.
- For neighboring style reference, read one or two recent pages in the same analyzer folder.

## Step 6 — Search for external references

Determine whether the rule's concept benefits from a `### See also` section. Rules about platform behavior, performance, SQL generation, permissions, events, or complex concepts almost always benefit. Simple naming or style convention rules usually do not — skip this step for those and proceed to Step 7.

1. **Microsoft Learn first** — use `microsoft_docs_search` with the underlying concept and "Business Central" (e.g., "FlowFields editable property Business Central", "InherentPermissions Business Central"). Search for the AL property, method, or platform feature the rule addresses, not the rule ID.
2. **Community content** — use `WebSearch` for the concept plus "Business Central".
3. **Verify every candidate** — use `WebFetch` (or `microsoft_docs_fetch` for learn.microsoft.com URLs) to confirm each link exists and is directly relevant to the rule's concept. Discard pages that merely mention the term in passing.
4. **Add to the draft** — a verified source can be used two ways:
   - When it pins down the precise behavior the page describes, quote the decisive sentence(s) inline as a `>` blockquote with an attribution link, at the point in the prose where the behavior is discussed.
   - Otherwise, list it in a `### See also` section at the bottom of the page (after `### Exception` if present). Follow the existing format: `[Title](URL) on Microsoft Learn`, `[Title](URL) by Author Name`.
   Links that define a term (a related rule, the Learn page for the property under discussion) may also appear inline where the term first occurs. Include internal cross-references to related rules in the same analyzer if they exist. Never put the same URL both inline and in See also.
5. **Quality bar** — include 1–5 links. Prefer one authoritative Microsoft Learn page over three tangential ones. Omit the section entirely rather than filling it with low-relevance links.

## Step 7 — Update the index

Add a row for the rule to `content/docs/analyzers/<AnalyzerName>/_index.md`, keeping the table sorted by ID:

`| [<RULEID>](<ruleid lowercase>/) | <Title without ID> | <Severity> | <✓ if enabled> | <✓ if code fix> |`

## Step 8 — Verify

- Run `hugo --quiet` (or `hugo --minify`) from the repo root and surface any errors or warnings.
- Confirm the new page and the index row agree on title, severity, and code fix.

## Step 9 — Report

End with a short report: a table of the extracted metadata with the source file each value came from, the fixtures consulted, the external links included in the See also section (or a note that the concept was too simple to warrant one), and a list of statements in the page the user should fact-check (anything not directly grounded in source).
