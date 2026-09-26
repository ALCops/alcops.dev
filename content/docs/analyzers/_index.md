---
title: "Analyzers"
weight: 20
type: docs
no_list: true
---

Every rule belongs to exactly one analyzer, and the analyzer tells you what kind of problem the rule describes and what happens if you ignore it. The rule ID carries the analyzer as its prefix: PC0003 is a PlatformCop rule, AC0002 an ApplicationCop rule.

The table is ordered by what ignoring a rule costs you, from a runtime failure down to an inconsistent-looking file.

| Analyzer | Inspects | If you ignore it | Example |
|---|---|---|---|
| [PlatformCop](platformcop/) | Constructs the AL runtime rejects, ignores, or executes differently from what the code says | A runtime error, or code that silently does nothing | [PC0003](platformcop/pc0003/): a filter operator inside `SetRange` is matched literally |
| [ApplicationCop](applicationcop/) | How tables, fields, pages, enums, labels and permissions are modeled | The code runs, but the object behaves unlike the standard application | [AC0002](applicationcop/ac0002/): a single-field primary key without `NotBlank` accepts an empty key |
| [LinterCop](lintercop/) | Procedure bodies: complexity, redundancy, patterns with a modern AL replacement | Nothing breaks; the cost lands on whoever maintains the code next | [LC0090](lintercop/lc0090/): cognitive complexity above the threshold |
| [DocumentationCop](documentationcop/) | Comments and XML documentation | Nothing at runtime; the next developer has to reverse-engineer intent | [DC0001](documentationcop/dc0001/): a `Commit` without a comment explaining why |
| [FormattingCop](formattingcop/) | Casing, spacing, blank lines and ordering | Nothing at runtime; the file reads differently from the rest of the codebase | [FC0002](formattingcop/fc0002/): casing mismatch with the declaration |
| [TestAutomationCop](testautomationcop/) | Test codeunits only; silent on production code | A test the runner skips or misreports | [TA0001](testautomationcop/ta0001/): a global procedure in a test codeunit that is not a test method |
| [Common](common/) | Cross-cutting diagnostics from the shared ALCops.Common library, loaded with every cop | A misconfiguration goes unnoticed | [CM0001](common/cm0001/): `alcops.json` cannot be loaded |

## PlatformCop, ApplicationCop or LinterCop?

Three analyzers look at code rather than documentation, layout or tests, and they split along one line. PlatformCop asks whether the code works: the construct is wrong for the AL runtime no matter which application it runs in. ApplicationCop asks whether the code fits Business Central: the construct works, but the resulting table, page or permission set does not behave like the base application. LinterCop asks whether the code is maintainable: it is correct and fits, but a shorter, clearer or faster form exists.

When a rule proposal fits more than one of these, the consequence decides. If ignoring the rule produces a runtime error or a silently wrong result, it is a PlatformCop rule.
