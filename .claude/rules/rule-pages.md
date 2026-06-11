# Rule page conventions

Canonical authoring conventions for analyzer rule pages on alcops.dev. The `/new-rule` and `/review-rule` skills treat this file as their checklist; follow it for any manual edit under `content/docs/analyzers/` as well.

## Content layout

Each analyzer rule has its own page under `content/docs/analyzers/<AnalyzerName>/`. There are six analyzers, each with a two-letter rule ID prefix:

| Prefix | Analyzer |
|---|---|
| AC | ApplicationCop |
| DC | DocumentationCop |
| FC | FormattingCop |
| LC | LinterCop |
| PC | PlatformCop |
| TA | TestAutomationCop |

Guides outside the analyzers live in `content/docs/getting-started/` and as standalone pages under `content/docs/`.

## Front matter

- Rule pages use TOML front matter (`+++`).
- Section index pages (`_index.md`) and non-rule pages use YAML front matter (`---`).

Every **new** rule page must include the `[params]` table:

```toml
+++
title = 'Rule description here (XX0000)'
linkTitle = 'XX0000'

[params]
  severity = 'Warning'
  codeAction = true
  codeActionType = 'QuickFix'
  supportsFixAll = false
  ignoresObsoletePending = false
+++
```

| Field | Required | Type | Values |
|-------|----------|------|--------|
| severity | Yes | string | `Error`, `Warning`, `Info`, `Hidden` |
| category | Yes | string | `Design`, `Naming`, `Style`, `Usage`, `Performance`, `Security` |
| codeAction | Yes | bool | `true` if a code action (quick fix or refactoring) is available |
| codeActionType | When codeAction=true | string | `QuickFix` or `Refactor` |
| supportsFixAll | When codeAction=true | bool | `true` if the fix can be applied to all occurrences |
| ignoresObsoletePending | Yes | bool | `true` if the rule skips elements marked as obsolete pending |

- `title` includes the rule description and ID in parentheses, e.g. `'FlowFields should not be editable (PC0001)'`.
- `linkTitle` is just the rule ID, e.g. `'PC0001'`.
- The properties table at the top of each rule page is rendered automatically by `layouts/partials/rule-properties.html` from these params. **Never add a properties table manually in the body.**
- Most pages written before the `[params]` schema existed don't have it yet. When editing such a page, adding `[params]` (with values verified against the analyzer source) is welcome but optional; for new pages it is mandatory.
- All metadata values come from the analyzer C# source, not from memory — see "Analyzer source as source of truth" below.

## Body structure

Sections appear in this order:

1. **Why** (no heading): opening paragraphs explaining the diagnostic. See "Writing the Why section".
2. `### Example`: "bad" AL code with the diagnostic message as an inline comment: `// Rule description [XX0000]`
3. Fix text + "fixed" AL code block. The fixed example shows only the corrected code, without repeating the diagnostic comment.
4. `### When the diagnostic is reported` (optional): bullet list of triggering conditions.
5. `### When the diagnostic is NOT reported` (optional): bullet list of exclusions.
6. `### Code fix` (optional, only when a code action exists): what the automated fix does.
7. `### Exception` (optional): when suppression is valid, with a pragma directive example and a comment explaining the design decision.
8. `### See also` (optional): bullet list of external links.

## Writing the Why section

The opening paragraphs (before `### Example`) explain why the diagnostic exists. Follow this three-beat structure:

1. **What happens**: the mechanism or runtime behavior. State facts about what the platform, compiler, or runtime does with this code.
2. **Why it matters**: the concrete consequence. Not "unexpected behavior" but the specific impact (silent data loss, N+1 SQL queries, compilation failure in consumers, misleading dead code).
3. **What to do instead**: one sentence pointing to the fix approach, before the code example shows it.

Scale depth by rule complexity:

- Naming/style rules: 2-3 sentences covering all three beats.
- Platform correctness rules: 1-2 paragraphs (the mechanism needs explaining).
- Performance rules: explain what is generated (SQL, API calls) and the cost at scale.

## Writing style

The audience is AL developers working with Business Central:

- The reader knows AL, Business Central, records, codeunits, events, pages. Do not explain these.
- The reader does NOT know platform internals (how FlowFields compute, how SQL is generated, how the event subscription queue works). Explain those.
- Lead with mechanism, not imperatives. The reader should understand the system behavior first.

Voice:

- Direct, declarative tone. State what the platform does, what goes wrong, and how to fix it. Do not hedge, qualify, or soften.
  - **Good**: "The platform does not validate, store, or pass that input anywhere on its own."
  - **Bad**: "It is generally recommended to consider whether the input might not be validated."
- Keep sentences short. One idea per sentence. Break long paragraphs into two or three shorter ones.
- When a rule has nuance, address it directly. If there are valid reasons to suppress the diagnostic, show the exception with a pragma directive and a comment explaining the design decision.

What to avoid:

- **Transition filler**: "Furthermore", "Additionally", "It's important to note that", "In conclusion", "It's worth mentioning".
- **Vague warnings**: never "this may lead to unexpected behavior" — describe what actually happens.
- **Selling the rule**: never "This powerful rule ensures best practices" — the reader already has the diagnostic; explain the underlying problem.
- **Repeating the title**: the opening paragraph must not restate the page title as a sentence.
- **Generic advice**: no "Tips" or "Best practices" sections that could appear on any rule page.

## Code examples

- Use realistic AL objects when explaining concepts (e.g., `Record Customer`, `Codeunit "Sales-Post"`, `"G/L Entry"`), not only generic `MyTable`/`MyCodeunit` placeholders.
- In the "bad" example, include the diagnostic message as an inline comment: `// FlowFields should not be editable [PC0001]`
- In the "fixed" example, show only the corrected code without the diagnostic comment.
- Keep examples minimal: enough context to understand the problem, no more.
- Code blocks use fenced markdown (```` ```al ````) or the Hugo `{{< highlight al "hl_lines=3" >}}` shortcode when line highlighting is needed. Highlight line numbers count from the first line *inside* the shortcode.

## Hugo specifics

- **Lowercase internal links**: `layouts/_default/_markup/render-link.html` lowercases all internal link URLs. Cross-references must use lowercase paths: `[PC0034](pc0034/)`, never `[PC0034](PC0034/)`.
- **Images**: use a Hugo page bundle — a folder named after the rule ID containing `index.md` and the image files. Reference images with the `{{< imgproc "filename.png" Fit "800x600" >}}` shortcode. Example bundle: `content/docs/analyzers/LinterCop/LC0028/`.
- **Callouts**: `{{% alert title="..." %}}` for callout boxes.
- Landing/section pages use Docsy blocks: `{{% blocks/cover %}}`, `{{% blocks/lead %}}`, `{{% blocks/section %}}`, `{{% blocks/feature %}}`.

## Analyzer index pages (`_index.md`)

Each analyzer folder has an `_index.md` with a description and a rules table:

```markdown
| ID | Title | Severity | Enabled | Code Fix |
|---|---|---|---|---|
| [PC0001](pc0001/) | FlowFields should not be editable | Warning | ✓ | ✓ |
```

- ID column links to the page with a **lowercase** relative URL.
- Title is the rule description without the ID suffix.
- Enabled and Code Fix use `✓` or empty.
- Rows are sorted by rule ID.

**When adding a new rule page, updating the analyzer's `_index.md` table is mandatory** — a page without an index row is invisible to readers scanning the rule list.

## Analyzer source as source of truth

The C# analyzer implementation lives in the sibling repository, expected at `../Analyzers` relative to this repo. Front matter metadata is always derived from it, never guessed:

- `src/ALCops.<AnalyzerName>/DiagnosticIds.cs` — maps rule ID to rule name.
- `src/ALCops.<AnalyzerName>/DiagnosticDescriptors.cs` — severity, category, enabled-by-default.
- `src/ALCops.<AnalyzerName>/CodeFixes/` — a provider referencing the rule means `codeAction = true` (file names vary: `<RuleName>.cs` or `<RuleName>CodeFixProvider.cs`); `Kind => CodeActionKind.QuickFix|Refactor` gives `codeActionType`; the value flowing into `SupportsFixAll` (often a `generateFixAll` constructor flag) gives `supportsFixAll`.
- `src/ALCops.<AnalyzerName>/Analyzers/<RuleName>.cs` — an `IsObsolete()` guard means `ignoresObsoletePending = true`.
- `src/ALCops.<AnalyzerName>/ALCops.<AnalyzerName>Analyzers.resx` — the diagnostic's title, message format, and description.
- `src/ALCops.<AnalyzerName>.Test/Rules/<RuleName>/{HasDiagnostic,NoDiagnostic,HasFix}/*.al` — real AL fixtures showing exactly when the rule does and does not trigger; the best raw material for examples and the "When the diagnostic is (NOT) reported" sections.

## External references

When a concept has depth (e.g., Cognitive Complexity, performance implications of `Count()`), link to authoritative community sources: Vjeko.com, Erik Hougaard, Microsoft Learn, SonarSource, Dynamics 365 Lab. Place these in a `### See also` section at the bottom, not scattered through the text.
