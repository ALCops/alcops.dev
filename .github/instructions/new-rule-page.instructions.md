---
applyTo: 'content/docs/analyzers/**'
---

# Adding a New Rule Page

When adding a new rule page (e.g., `PC0034.md`) to an analyzer folder, you **must also update the `_index.md`** in that same folder to include the new rule in the rules table.

## Checklist for adding a new rule page

1. Create the rule page: `content/docs/analyzers/<AnalyzerName>/<RULEID>.md`
2. **Update `content/docs/analyzers/<AnalyzerName>/_index.md`** - add a row to the rules table with: ID (linked), Title, Severity, Enabled, Code Fix columns
3. Verify the link in the table uses lowercase: `[PC0034](pc0034/)`

## Rules table format

```markdown
| [PC0034](pc0034/) | Placeholder argument count mismatch | Warning | ✓ | |
```

Columns: ID (linked to lowercase path with trailing slash), Title, Severity, Enabled (✓ if enabled by default), Code Fix (✓ if a code fix exists).

## Common mistake

Forgetting step 2. The rule page will exist but won't appear in the analyzer's index, making it undiscoverable from the docs navigation.
