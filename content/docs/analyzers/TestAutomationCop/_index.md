---
title: "TestAutomationCop"
type: docs
no_list: true
---

TestAutomationCop inspects test codeunits and is silent on production code. It flags test procedures whose structure or attributes keep the test runner from executing them as intended, such as a global procedure in a test codeunit that is not marked as a test method.

## Rules

| ID | Title | Severity | Enabled | Code Fix |
|---|---|---|---|---|
| [TA0001](ta0001/) | Global procedures in test codeunits must be test methods | Warning | ✓ | |
