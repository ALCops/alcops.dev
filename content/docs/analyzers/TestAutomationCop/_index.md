---
title: "TestAutomationCop"
type: docs
no_list: true
---

TestAutomationCop inspects test codeunits and is silent on production code. It flags test code whose structure keeps the test runner from executing it as intended, such as a missing `[Test]` attribute or an action invoked on a part page that renders no actions at runtime.

## Rules

| ID | Title | Severity | Enabled | Code Fix |
|---|---|---|---|---|
| [TA0001](ta0001/) | Global procedures in test codeunits must be test methods | Warning | ✓ | |
| [TA0002](ta0002/) | Actions cannot be invoked on a part page opened directly through its own TestPage variable | Warning | ✓ | |
