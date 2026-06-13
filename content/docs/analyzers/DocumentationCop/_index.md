---
title: "DocumentationCop"
type: docs
no_list: true
---

The DocumentationCop ensures that AL code is properly documented and that non-obvious patterns are explained. It requires comments on constructs that would otherwise confuse future developers and validates the presence and correctness of XML documentation. These rules do not change runtime behavior but improve code clarity and maintainability.

## Rules

| ID | Title | Severity | Enabled | Code Fix |
|---|---|---|---|---|
| [DC0001](dc0001/) | Commit requires a comment explaining why | Info | ✓ | |
| [DC0002](dc0002/) | Writing to a FlowField requires a comment explaining why | Info | ✓ | |
| [DC0003](dc0003/) | Empty statements should be removed or documented | Info | ✓ | |
| [DC0004](dc0004/) | Public procedures must have XML documentation | Info | ✓ | |
| [DC0005](dc0005/) | XML documentation must match the procedure signature | Info | ✓ | |