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
  category = 'Design'
  codeAction = true
  ignoreObsolete = false
+++
```

| Field | Required | Type | Values |
|-------|----------|------|--------|
| severity | Yes | string | `Error`, `Warning`, `Info`, `Hidden` |
| category | Yes | string | `Design`, `Naming`, `Style`, `Usage`, `Performance`, `Security` |
| codeAction | Yes | bool | `true` if a code action is available |
| ignoreObsolete | Yes | bool | `true` if the rule skips elements marked as obsolete |

- `title` includes the rule description and ID in parentheses, e.g. `'FlowFields should not be editable (PC0001)'`.
- `linkTitle` is just the rule ID, e.g. `'PC0001'`.
- The properties strip at the top of each rule page is rendered automatically by `layouts/partials/rule-properties.html` from these params. **Never add a properties table or list manually in the body.**
- When a rule is **disabled by default** (`isEnabledByDefault: false` in `DiagnosticDescriptors.cs`), set `severity = 'Hidden'` instead of the rule's default severity. This is sufficient — do not add a "disabled by default" note in the page body.
- Most pages written before the `[params]` schema existed don't have it yet. When editing such a page, adding `[params]` (with values verified against the analyzer source) is welcome but optional; for new pages it is mandatory.
- All metadata values come from the analyzer C# source, not from memory — see "Analyzer source as source of truth" below.

## Body structure

Rule pages come in two tiers. Decide the tier before drafting.

### Standard rules

The vast majority. Sections appear in this order:

1. **Why** (no heading): opening paragraphs explaining the diagnostic. See "Writing the Why section".
2. `### Example`: "bad" AL code with the diagnostic message as an inline comment: `// Rule description [XX0000]`
3. Fix text + "fixed" AL code block. The fixed example shows only the corrected code, without repeating the diagnostic comment. When more than one fix is idiomatic, show each of them, with a sentence on when to pick which.
4. `### When the diagnostic is reported` (optional): bullet list of triggering conditions. Only include this section when the conditions are not obvious from the example alone — e.g. the rule has scope limits (same module, specific loop types) or non-trivial preconditions. Skip it for simple rules where the example says it all.
5. `### Code fix` (optional, only when a code action exists): what the automated fix does.
6. `### Exception` (optional): when suppression is valid, or when a surprising exclusion exists (e.g. the rule skips temporary records, or certain field names). Include a pragma directive example with a comment explaining the design decision.
7. `### See also` (optional): bullet list of external links.

### Concept-heavy rules

The rule documents a metric, a methodology, or platform machinery that must be taught before any example makes sense — Cognitive Complexity-class rules. These pages are free-form: structure the page however the concept demands. A custom H2/H3 outline, named nuance subsections (`### Date2DWY(MyDate, 3) vs MyDate.Year()`), comparison and threshold tables, and worked walkthroughs with annotated code are all legitimate. Only three things stay mandatory:

- TOML front matter with the `[params]` table.
- An opening Why without a heading.
- At least one bad/fixed example pair following the diagnostic-comment and `hl_lines` conventions.

**Tier test**: if the reader can understand the fix from one example pair, it is a standard rule. If you would need to teach a scoring model, an algorithm, or non-obvious platform machinery before the example means anything, it is concept-heavy. When in doubt, write standard — depth can grow later.

### For both tiers

Do **not** add a `### When the diagnostic is NOT reported` section. Exclusions that are the logical negation of the triggering conditions add no value. Surprising or non-obvious exclusions belong in `### Exception`.

## Writing the Why section

The opening paragraphs (before the first example) explain why the diagnostic exists. Write them from the reader's seat: they just wrote the flagged code on purpose, believing it works. Follow this three-beat structure:

1. **The intent**: what the developer was trying to do when they wrote this code, and what the platform actually does with it instead. Anchor the mechanism to that moment — "Many AL developers attempt to set an *is not empty* filter this way; the filter silently matches nothing" — not a textbook fact stated in a vacuum.
2. **One concrete failure**: name the specific thing that breaks — the field that stays stale, the exact runtime error, the SQL that gets generated. One named failure the reader can picture beats any number of categories.
3. **What to do instead**: one sentence pointing to the fix approach, before the code example shows it.

Never write a categorical consequence list — a chain of abstract harm categories ("data integrity issues, business rules not enforced, compliance checks skipped") that could be pasted onto any rule in the analyzer. If you cannot name one concrete failure, you do not understand the rule well enough to write the page; go back to the analyzer source and its test fixtures.

Scale depth to the concept:

- Naming/style rules: 2-3 sentences covering all three beats.
- Platform correctness rules: 1-2 paragraphs (the mechanism needs explaining).
- Performance rules: explain what is generated (SQL, API calls) and the cost at scale.
- Concept-heavy rules: no ceiling. The Why opens the page; the teaching continues in the free-form sections that follow (see "Body structure").

## Writing style

The audience is AL developers working with Business Central:

- The reader knows AL, Business Central, records, codeunits, events, pages. Do not explain these.
- The reader does NOT know platform internals (how FlowFields compute, how SQL is generated, how the event subscription queue works). Explain those.
- Lead with mechanism, not imperatives — but anchor the mechanism in the developer's intent (see "Writing the Why section"). The reader should understand the system behavior before being told what to change.

Voice:

- Direct, declarative tone. State what the platform does, what goes wrong, and how to fix it. Do not hedge, qualify, or soften.
  - **Good**: "The platform does not validate, store, or pass that input anywhere on its own."
  - **Bad**: "It is generally recommended to consider whether the input might not be validated."
- Keep sentences short. One idea per sentence. Break long paragraphs into two or three shorter ones.
- When a rule has nuance, address it directly. If there are valid reasons to suppress the diagnostic, show the exception with a pragma directive and a comment explaining the design decision. When the nuance is a subtle difference between two approaches, give it a named subsection (`### Date2DWY(MyDate, 3) vs MyDate.Year()`) instead of burying it mid-paragraph.
- When Microsoft Learn, a whitepaper, or the platform source states the precise behavior, quote it in a `>` blockquote with an attribution link. A verbatim quote from the authority carries more weight than a weak paraphrase with the link buried in See also.
- When the recommended fix has a cost — performance, verbosity, an extra dependency — state it plainly next to the recommendation. The reader trusts a page that admits tradeoffs.

What to avoid:

- **Transition filler**: "Furthermore", "Additionally", "It's important to note that", "In conclusion", "It's worth mentioning".
- **Vague warnings**: never "this may lead to unexpected behavior" — describe what actually happens.
- **Categorical consequence lists**: never a chain of abstract harm categories ("data integrity issues, constraints not enforced, subscribers never fire") in place of one concrete, named failure. See "Writing the Why section".
- **Selling the rule**: never "This powerful rule ensures best practices" — the reader already has the diagnostic; explain the underlying problem.
- **Repeating the title**: the opening paragraph must not restate the page title as a sentence.
- **Generic advice**: no "Tips" or "Best practices" sections that could appear on any rule page.

### Voice exemplars

Calibrate against these pairs. The good lines come from hand-written rule documentation that sets the bar for this site; match their structure, not just their grammar.

**Intent-anchored opening vs textbook opening**

- **Good**: "At some point, many AL developers attempt to set an *is not empty* filter as shown above. Unfortunately, this approach does not produce the expected result."
- **Bad**: "The `SetFilter` method accepts a filter expression as its second parameter. Incorrect escaping of single quotes can lead to unintended filter behavior."

**Concrete failure vs category list**

- **Good**: "Directly comparing a GUID to an empty string (`''`) causes a runtime error, because `''` is not a valid GUID format."
- **Bad**: "Bypassing validation can cause data integrity issues that are difficult to trace: values remain stale, cross-field constraints are not enforced, and downstream subscribers never fire."

**Inline authoritative quote vs weak paraphrase**

- **Good**:

  > When the input date to the **Date2DWY** method is in a week that spans two years, the **Date2DWY** method computes the output year as the year that has the most days.

  — [System.Date2DWY(Date, Integer) Method](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/methods-auto/system/system-date2dwy-method#remarks) on Microsoft Learn

- **Bad**: "Note that `Date2DWY` may return a different year in some edge cases (see Microsoft Learn for details)."

**Honest tradeoff vs unconditional recommendation**

- **Good**: "The AL language does not support lazy evaluation. While a sequence of binary logical operators can improve readability, it may come at a performance cost at runtime."
- **Bad**: "Combining conditions into a single expression is always cleaner and should be preferred."

**Named nuance subsection vs buried caveat**

- **Good**: a dedicated `### Date2DWY(MyDate, 3) vs MyDate.Year()` heading when two approaches differ in a way the reader must understand before choosing.
- **Bad**: appending "however, note that the two methods differ slightly in week-spanning years" to the end of an unrelated paragraph.

## Code examples

- Use realistic AL objects when explaining concepts (e.g., `Record Customer`, `Codeunit "Sales-Post"`, `"G/L Entry"`), not only generic `MyTable`/`MyCodeunit` placeholders.
- In the "bad" example, include the diagnostic message as an inline comment: `// FlowFields should not be editable [PC0001]`
- Do not add a separate rendered diagnostic message block after the bad example — the inline comment is sufficient.
- In the "fixed" example, show only the corrected code without the diagnostic comment.
- When more than one fix is idiomatic, show each of them in the fixed example (e.g. both `SetFilter("Field", '<>%1', '')` and `SetFilter("Field", '<>''''')`), with a sentence on when to pick which. Do not pretend a single canonical fix exists when the community uses several.
- For rules about a metric or a computation (complexity scores, counts), annotate the example per line with comments showing how the value builds up (`// +2 (1 increment + 1 nesting penalty)`), so the reader can re-derive the number.
- When the rule maps an old API to a new one method-by-method, use a comparison table (old call, new call, what it returns) instead of prose pairs.
- Keep examples minimal: enough context to understand the problem, no more.
- Only include the surrounding object declaration (`codeunit`, `table`, `page`, etc.) when the object type is relevant to the rule — e.g., the rule checks a table property, an object-level attribute, or the diagnostic applies to a specific object kind. When the rule targets a method call, variable usage, or code pattern inside a procedure, show only the procedure body.
- Use the Hugo highlight shortcode with `hl_lines` for all example code blocks. In the "bad" example, highlight the line(s) where the diagnostic is raised. In the "fixed" example, highlight the line(s) that changed. Line numbers count from the first line *inside* the shortcode. Use ranges for consecutive lines (e.g. `"hl_lines=3-4"`). Example:

  ```
  {{</* highlight al "hl_lines=5" */>}}
  table 50100 "Item Category"
  {
      fields
      {
          field(1; "Code"; Code[20]) // NotBlank required [AC0002]
          {
          }
      }
  }
  {{</* /highlight */>}}
  ```

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
- `src/ALCops.<AnalyzerName>/CodeFixes/` — a provider referencing the rule means `codeAction = true` (file names vary: `<RuleName>.cs` or `<RuleName>CodeFixProvider.cs`).
- `src/ALCops.<AnalyzerName>/Analyzers/<RuleName>.cs` — an `IsObsolete()` guard means `ignoreObsolete = true`.
- `src/ALCops.<AnalyzerName>/ALCops.<AnalyzerName>Analyzers.resx` — the diagnostic's title, message format, and description.
- `src/ALCops.<AnalyzerName>.Test/Rules/<RuleName>/{HasDiagnostic,NoDiagnostic,HasFix}/*.al` — real AL fixtures showing exactly when the rule does and does not trigger; the best raw material for examples and the "When the diagnostic is (NOT) reported" sections.

## External references

When a concept has depth (e.g., Cognitive Complexity, performance implications of `Count()`), link to authoritative community sources: Vjeko.com, Erik Hougaard, Microsoft Learn, SonarSource, Dynamics 365 Lab.

Two placements, both valid on the same page:

- **Inline**, where a term first appears, when the link defines or explains that term — a related rule (`[lc0010](lc0010/)`), the Learn page for the property or method under discussion, the article that coined a concept. The reader should be able to click at the moment of confusion, not hunt at the bottom.
- **`### See also`** at the bottom for further reading that does not anchor to a specific sentence.

Do not duplicate the same URL in both places, and do not turn prose into a link farm — an inline link earns its place only when the sentence is about the thing it links to.
