# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Documentation site for [ALCops](https://alcops.dev), a collection of code analyzers for the AL programming language (Microsoft Dynamics 365 Business Central). Built with Hugo (Extended) and the Docsy theme. Deployed to GitHub Pages via `.github/workflows/hugo.yaml` on push to `main`.

## Commands

```bash
npm install          # Install PostCSS dependencies (required once)
hugo server          # Local dev server at http://localhost:1313
hugo --minify        # Production build to public/
```

There are no tests or linters configured. Verify content changes by running `hugo server` (or `hugo --minify` for a build check). On WSL with Hugo installed on the Windows side (WinGet), the binary is `hugo.exe`.

## Architecture

- **Hugo module setup**: Docsy is imported as a Go module (`go.mod`), not a git submodule. Hugo resolves it automatically.
- **Lowercase links**: `layouts/_default/_markup/render-link.html` lowercases all internal link URLs. Cross-references to content pages must use lowercase paths, e.g. `[PC0034](pc0034/)`.
- **Auto-rendered rule properties**: `layouts/partials/rule-properties.html` renders the properties table at the top of each rule page from front matter `[params]`. Never add a properties table manually in the body.
- **Sibling analyzer repo**: the C# analyzer source lives in [ALCops/Analyzers](https://github.com/ALCops/Analyzers), expected at `../Analyzers` next to this repo. It is the source of truth for all rule metadata (severity, category, code fix, obsolete handling) and its test fixtures (`src/ALCops.<Cop>.Test/Rules/<RuleName>/`) show exactly when each rule triggers.

## Content

All documentation lives under `content/docs/`: one page per analyzer rule in `content/docs/analyzers/<AnalyzerName>/`, guides in `content/docs/getting-started/`. Authoring conventions for rule pages (front matter schema, body structure, writing style, index tables) are defined in the imported rules file below and apply to every change under `content/docs/analyzers/`.

@.claude/rules/rule-pages.md

## Skills

- `/new-rule <rule-id>` — draft a complete rule page, deriving metadata from the analyzer C# source and examples from its test fixtures; updates the analyzer's `_index.md` and verifies the Hugo build.
- `/review-rule [rule-id|path] [--fix]` — review one page (or all uncommitted rule pages when no target is given) against the conventions; reports findings and fixes only on request.
