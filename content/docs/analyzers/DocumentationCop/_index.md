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
| [DC0004](dc0004/) | Public procedures must include XML documentation comments | Info | ✓ | |
| [DC0005](dc0005/) | XML documentation must match the procedure signature | Info | ✓ | |
| [DC0006](dc0006/) | Internal procedures must include XML documentation comments | Hidden | ✓ | |
| [DC0007](dc0007/) | Public objects must include XML documentation comments | Info | ✓ | |
| [DC0008](dc0008/) | Internal objects must include XML documentation comments | Hidden | ✓ | |
| [DC0009](dc0009/) | Events must include XML documentation comments | Hidden | ✓ | |
| [DC0010](dc0010/) | Internal events must include XML documentation comments | Hidden | ✓ | |
