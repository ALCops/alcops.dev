---
title: "LinterCop"
type: docs
no_list: true
---

The LinterCop is a code linter for AL, comparable to widely used static analysis linters in other programming languages. It flags code quality issues, measures complexity, and promotes modern AL patterns. The code it flags is technically valid, but can be written in a cleaner, more maintainable or more idiomatic way.

## Rules

| ID | Title | Severity | Enabled | Code Fix |
|---|---|---|---|---|
| [LC0003](lc0003/) | Do not use Object IDs as object references | Warning | ✓ | ✓ |
| [LC0007](lc0007/) | Maintainability Index metric | Hidden | | |
| [LC0008](lc0008/) | Maintainability Index threshold exceeded | Warning | ✓ | |
| [LC0009](lc0009/) | Cyclomatic Complexity metric | Info | ✓ | |
| [LC0010](lc0010/) | Cyclomatic Complexity threshold exceeded | Warning | ✓ | |
| [LC0019](lc0019/) | DataClassification redundancy | Warning | ✓ | ✓ |
| [LC0020](lc0020/) | ApplicationArea redundancy | Info | ✓ | ✓ |
| [LC0028](lc0028/) | Use identifier syntax in event subscriber arguments | Warning | ✓ | |
| [LC0031](lc0031/) | Use ReadIsolation instead of LockTable | Info | ✓ | ✓ |
| [LC0033](lc0033/) | App manifest runtime is behind target version | Warning | ✓ | |
| [LC0040](lc0040/) | Explicitly set RunTrigger | Info | ✓ | |
| [LC0043](lc0043/) | Use SecretText for sensitive text | Warning | ✓ | |
| [LC0048](lc0048/) | Use Error with ErrorInfo or Label instead of Text | Warning | ✓ | |
| [LC0052](lc0052/) | Internal procedure not referenced | Warning | ✓ | |
| [LC0053](lc0053/) | Internal procedure only used in current object | Warning | ✓ | |
| [LC0054](lc0054/) | Interface object name guide | Hidden | | |
| [LC0063](lc0063/) | API page canonical field name guide | Info | ✓ | |
| [LC0081](lc0081/) | Use IsEmpty method instead of Count | Warning | ✓ | |
| [LC0082](lc0082/) | Use Query or Find with Next instead of Count | Info | ✓ | |
| [LC0083](lc0083/) | Use new Date/Time/DateTime methods for extracting parts | Info | ✓ | ✓ |
| [LC0086](lc0086/) | Page style should not use string literal | Warning | ✓ | |
| [LC0088](lc0088/) | Prefer Enum over Option type | Info | ✓ | |
| [LC0089](lc0089/) | Cognitive Complexity metric | Hidden | | |
| [LC0090](lc0090/) | Cognitive Complexity threshold exceeded | Warning | ✓ | |
| [LC0091](lc0091/) | Translatable text should be translated | Warning | ✓ | |
| [LC0092](lc0092/) | Name does not match naming convention | Warning | ✓ | |
| [LC0094](lc0094/) | AllowInCustomizations redundancy | Warning | ✓ | ✓ |
| [LC0095](lc0095/) | Parameter is not referenced | Warning | ✓ | ✓ |
| [LC0096](lc0096/) | Unnecessary record parameter in method call | Warning | ✓ | |
| [LC0097](lc0097/) | Avoid mixing exit() and named return variable assignments | Info | — | |
| [LC0099](lc0099/) | Event subscriber parameter is not referenced | Info | ✓ | ✓ |
