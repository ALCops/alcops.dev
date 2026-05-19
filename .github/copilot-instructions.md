# Copilot Instructions for alcops.dev

## Project overview

This is the documentation site for [ALCops](https://alcops.dev), a collection of code analyzers for the AL programming language (Microsoft Dynamics 365 Business Central). Built with [Hugo](https://gohugo.io/) and the [Docsy](https://www.docsy.dev/) theme. Deployed to GitHub Pages via the `hugo.yaml` workflow on push to `main`.

## Commands

```bash
npm install          # Install PostCSS dependencies (required once)
hugo server          # Local dev server at http://localhost:1313
hugo --minify        # Production build to public/
```

There are no tests or linters configured for this repository.

## Architecture

- **Hugo module setup**: Docsy is imported as a Go module (`go.mod`), not a git submodule. Hugo resolves it automatically.
- **Custom layout**: `layouts/_default/_markup/render-link.html` lowercases all internal link URLs. This means content file paths must use lowercase in cross-references.
- **Deployment**: The GitHub Actions workflow (`hugo.yaml`) installs Hugo Extended + Dart Sass, runs `npm ci`, builds with `hugo --minify`, and deploys to GitHub Pages.

## Content conventions

### Rule pages

Each analyzer rule has its own page under `content/docs/analyzers/<AnalyzerName>/`. There are six analyzers: ApplicationCop, DocumentationCop, FormattingCop, LinterCop, PlatformCop, and TestAutomationCop.

Rule IDs use a two-letter prefix matching the analyzer: `AC`, `DC`, `FC`, `LC`, `PC`, `TA`.

A rule page follows this pattern:

1. **Front matter** using TOML (`+++`):
   - `title` includes the rule description and ID in parentheses, e.g. `'FlowFields should not be editable (PC0001)'`
   - `linkTitle` is just the rule ID, e.g. `'PC0001'`
   - `[params]` table with rule metadata (see "Front matter schema" below)

2. **Body** contains sections in this order:
   - **Why** (no heading): 1-3 paragraphs explaining the diagnostic. Structure: what happens at the platform/runtime level → why it matters (concrete consequence) → what to do instead (one sentence).
   - `### Example`: "Bad" AL code with the diagnostic message as an inline comment.
   - Fix text + "fixed" AL code block.
   - `### When the diagnostic is reported` (optional): Bullet list of triggering conditions.
   - `### When the diagnostic is NOT reported` (optional): Bullet list of exclusions.
   - `### Code fix` (optional, only when a code action exists): What the automated fix does.
   - `### Exception` (optional): When suppression is valid, with pragma example.
   - `### See also` (optional): Bullet list of external links.

The properties table at the top of each rule page is rendered automatically by Hugo from the front matter `[params]`. Do not add a properties table manually in the body.

Code blocks use either fenced markdown (` ```al `) or the Hugo `{{< highlight al >}}` shortcode when line highlighting is needed.

### Front matter schema for rule pages

All rule pages must include the `[params]` table with these fields:

```toml
+++
title = 'FlowFields should not be editable (PC0001)'
linkTitle = 'PC0001'

[params]
  severity = 'Warning'
  category = 'Design'
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

### Writing the "Why" section

The opening paragraphs (before `### Example`) explain why the diagnostic exists. Follow this three-beat structure:

1. **What happens**: Describe the mechanism or runtime behavior. State facts about what the platform, compiler, or runtime does with this code.
2. **Why it matters**: The concrete consequence. Not "unexpected behavior" but the specific impact (silent data loss, N+1 SQL queries, compilation failure in consumers, misleading dead code).
3. **What to do instead**: One sentence pointing to the fix approach, before the code example shows it.

Scale depth by rule complexity:
- Naming/style rules: 2-3 sentences covering all three beats.
- Platform correctness rules: 1-2 paragraphs (the mechanism needs explaining).
- Performance rules: explain what is generated (SQL, API calls) and the cost at scale.

Audience assumptions:
- The reader knows AL, Business Central, records, codeunits, events, pages. Do not explain these.
- The reader does NOT know platform internals (how FlowFields compute, how SQL is generated, how the event subscription queue works). Explain those.
- Lead with mechanism, not imperatives. The reader should understand the system behavior first.

### Rule pages with images

When a rule page includes images, use a Hugo [page bundle](https://gohugo.io/content-management/page-bundles/): create a folder named after the rule ID containing `index.md` and the image files. Reference images with the `{{< imgproc >}}` shortcode. See `content/docs/analyzers/LinterCop/LC0028/` for an example.

### Analyzer index pages

Each analyzer folder has an `_index.md` with a description and a rules table. Columns: ID (linked), Title, Severity, Enabled, CodeFix. When adding a new rule, update this table.

### Front matter format

- Rule pages use TOML front matter (`+++`).
- Section index pages (`_index.md`) and non-rule pages use YAML front matter (`---`).

### Hugo shortcodes in use

- `{{% blocks/cover %}}`, `{{% blocks/lead %}}`, `{{% blocks/section %}}`, `{{% blocks/feature %}}` (Docsy layout)
- `{{% alert title="..." %}}` for callout boxes
- `{{< highlight al "hl_lines=3" >}}` for code blocks with line highlighting
- `{{< imgproc "filename.png" Fit "800x600" >}}` for images in page bundles

## Writing style for rule pages

The audience is AL developers working with Business Central. They know what a codeunit is, what a FlowField does, and how events work. Do not explain foundational AL or Business Central concepts.

### Voice

Write in a direct, declarative tone. State what the platform does, what goes wrong, and how to fix it. Do not hedge, qualify, or soften.

- **Good**: "The platform does not validate, store, or pass that input anywhere on its own."
- **Bad**: "It is generally recommended to consider whether the input might not be validated."

Keep sentences short. Prefer one idea per sentence. Break long paragraphs into two or three shorter ones.

### Structure of explanations

Lead with what happens at the platform or runtime level, not with what the developer "should" do. Explain the mechanism first, then the consequence.

- **Good**: "FlowFields are calculated fields. Their values come from other data, and they are never stored directly on the record."
- **Bad**: "You should avoid making FlowFields editable because it is considered best practice."

When a rule has nuance, address it directly. If there are valid reasons to suppress a diagnostic, show the exception with a pragma directive and a comment explaining the design decision.

### Code examples

- Use realistic AL objects when explaining concepts (e.g., `Record Customer`, `Codeunit "Sales-Post"`, `"G/L Entry"`), not only generic `MyTable`/`MyCodeunit` placeholders.
- In the "bad" example, include the diagnostic message as an inline comment: `// FlowFields should not be editable [PC0001]`
- In the "fixed" example, show only the corrected code without repeating the diagnostic message.
- Keep examples minimal. Show enough context to understand the problem, no more.

### What to avoid

- **Transition filler**: Do not use "Furthermore", "Additionally", "It's important to note that", "In conclusion", "It's worth mentioning".
- **Vague warnings**: Do not write "this may lead to unexpected behavior". Describe what actually happens.
- **Selling the rule**: Do not write "This powerful rule ensures best practices". The reader already has the diagnostic; explain the underlying problem.
- **Repeating the title**: The opening paragraph should not restate the page title as a sentence.
- **Generic advice**: Do not add "Tips" or "Best practices" sections that could appear on any rule page.

### External references

When a concept has depth (e.g., Cognitive Complexity, performance implications of `Count()`), link to authoritative community sources: Vjeko.com, Erik Hougaard, Microsoft Learn, SonarSource, Dynamics 365 Lab. Place these in a "See also" section at the bottom, not scattered through the text.
