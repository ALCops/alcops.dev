---
title: "DocumentationCop"
type: docs
no_list: true
---

DocumentationCop inspects comments and XML documentation. It flags public procedures without XML documentation and constructs whose intent is not stated in code, such as a `Commit` without a comment explaining why. Ignoring these rules changes nothing at runtime; it leaves the next developer to reverse-engineer what the code is for.

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
