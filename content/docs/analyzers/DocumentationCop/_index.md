---
title: "DocumentationCop"
type: docs
no_list: true
---

The DocumentationCop ensures that AL code is properly documented and that non-obvious patterns are explained. It requires comments on constructs that would otherwise confuse future developers and validates the presence and correctness of XML documentation. These rules do not change runtime behavior but improve code clarity and maintainability.

## Rules

| ID | Title | Severity | Enabled | Code Fix |
|---|---|---|---|---|
| [DC0001](dc0001/) | Commit requires a comment explaining why | Warning | ✓ | |
| [DC0002](dc0002/) | Writing to a FlowField requires a comment explaining why | Warning | ✓ | |
| [DC0003](dc0003/) | Empty statement requires a comment explaining why | Warning | ✓ | |
| [DC0004](dc0004/) | Public procedure requires XML documentation | Info | — | |
| [DC0005](dc0005/) | XML documentation must match procedure signature | Warning | ✓ | |

**Note:** Rules marked with "—" in the Enabled column are disabled by default and must be explicitly enabled in your project's `.editorconfig` or ruleset file.
