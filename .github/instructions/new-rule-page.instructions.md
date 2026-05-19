---
applyTo: 'content/docs/analyzers/**'
---

# Adding a New Rule Page

When adding a new rule page (e.g., `PC0034.md`) to an analyzer folder, you **must also update the `_index.md`** in that same folder to include the new rule in the rules table.

## Checklist for adding a new rule page

1. Create the rule page: `content/docs/analyzers/<AnalyzerName>/<RULEID>.md`
2. **Update `content/docs/analyzers/<AnalyzerName>/_index.md`** - add a row to the rules table with: ID (linked), Title, Severity, Enabled, Code Fix columns
3. Verify the link in the table uses lowercase: `[PC0034](pc0034/)`

## Front matter template

Every rule page must use this TOML front matter structure with the `[params]` table:

```toml
+++
title = 'Rule description here (XX0000)'
linkTitle = 'XX0000'

[params]
  severity = 'Warning'
  category = 'Design'
  codeAction = false
  codeActionType = ''
  supportsFixAll = false
  ignoresObsoletePending = false
+++
```

- `severity`: `Error`, `Warning`, `Info`, or `Hidden`
- `category`: `Design`, `Naming`, `Style`, `Usage`, `Performance`, or `Security`
- `codeAction`: `true` if a quick fix or refactoring is available
- `codeActionType`: `QuickFix` or `Refactor` (only when codeAction is true)
- `supportsFixAll`: `true` if the fix supports "Fix All" (only when codeAction is true)
- `ignoresObsoletePending`: `true` if the rule skips elements marked obsolete pending

The properties table is rendered automatically by Hugo from these params. Do not add a properties table in the body.

## Body structure

After the front matter, the body follows this section order:

1. **Why** (no heading): Opening paragraphs explaining the diagnostic.
   - What happens at the platform/runtime level (mechanism)
   - Why it matters (concrete consequence)
   - What to do instead (one sentence)
2. `### Example`: Bad code with diagnostic comment inline: `// Description [XX0000]`
3. Fix explanation + fixed code block.
4. `### When the diagnostic is reported` (optional): Bullet list of conditions.
5. `### When the diagnostic is NOT reported` (optional): Bullet list of exclusions.
6. `### Code fix` (optional, only if codeAction=true): What the fix does.
7. `### Exception` (optional): When suppression is valid, with pragma directive.
8. `### See also` (optional): Bullet list of external links.

## Rules table format

```markdown
| [PC0034](pc0034/) | Placeholder argument count mismatch | Warning | ✓ | |
```

Columns: ID (linked to lowercase path with trailing slash), Title, Severity, Enabled (✓ if enabled by default), Code Fix (✓ if a code fix exists).

## Common mistakes

- Forgetting to update `_index.md`. The rule page will exist but won't appear in the docs navigation.
- Omitting the `[params]` table from front matter. The properties table won't render.
- Using YAML front matter (`---`) instead of TOML (`+++`) for rule pages.
