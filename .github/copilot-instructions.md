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

2. **Body** contains:
   - An explanation of why the rule exists
   - A "bad" AL code example showing the diagnostic (with the diagnostic message as a comment)
   - A "fixed" AL code example
   - Optional sections: exceptions, code actions, "See also" links

Code blocks use either fenced markdown (` ```al `) or the Hugo `{{< highlight al >}}` shortcode when line highlighting is needed.

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
