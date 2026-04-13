---
title: "PlatformCop"
type: docs
no_list: true
---

The PlatformCop detects code that is technically broken, dangerous, or silently ignored at the AL platform level. It flags constructs where the runtime behaves differently from what the developer likely intended. Unlike the ApplicationCop, which focuses on Business Central application conventions, PlatformCop targets AL platform and runtime correctness regardless of the application context.

## Rules

| ID | Title | Severity | Enabled | Code Fix |
|---|---|---|---|---|
| [PC0001](pc0001/) | FlowFields should not be editable | Warning | ✓ | ✓ |
| [PC0002](pc0002/) | AutoIncrement in temporary table | Warning | ✓ | |
| [PC0003](pc0003/) | SetRange with filter operators | Warning | ✓ | ✓ |
| [PC0004](pc0004/) | List objects are one-based | Warning | ✓ | |
| [PC0005](pc0005/) | Extensible property explicitly set | Info | ✓ | ✓ |
| [PC0006](pc0006/) | Access property explicitly set | Info | ✓ | |
| [PC0007](pc0007/) | AutoCalcFields only on FlowFields | Warning | ✓ | |
| [PC0008](pc0008/) | Operator and placeholder in filter expression | Warning | ✓ | ✓ |
| [PC0010](pc0010/) | EventSubscriber var keyword | Warning | ✓ | ✓ |
| [PC0011](pc0011/) | EventPublisher IsHandled by var | Warning | ✓ | ✓ |
| [PC0012](pc0012/) | FlowFilter field assignment | Warning | ✓ | |
| [PC0013](pc0013/) | Record.Get procedure arguments | Error | ✓ | |
| [PC0014](pc0014/) | JsonToken JPath uses double quotes | Warning | ✓ | ✓ |
| [PC0015](pc0015/) | Guid empty string comparison | Warning | ✓ | ✓ |
| [PC0016](pc0016/) | Clear codeunit SingleInstance | Warning | ✓ | |
| [PC0017](pc0017/) | Page record argument mismatch | Warning | ✓ | |
| [PC0018](pc0018/) | Page record method requires SourceTable | Warning | ✓ | |
| [PC0019](pc0019/) | Filter string single quote escaping | Warning | ✓ | ✓ |
| [PC0020](pc0020/) | TransferFields type mismatch | Warning | ✓ | |
| [PC0021](pc0021/) | TransferFields name mismatch | Info | ✓ | |
| [PC0022](pc0022/) | Possible overflow assigning | Warning | ✓ | ✓ |
| [PC0023](pc0023/) | IsHandled parameter assignment | Warning | ✓ | |
| [PC0024](pc0024/) | ApplicationArea on API page | Warning | ✓ | ✓ |
| [PC0025](pc0025/) | ODataKeyFields should use SystemId | Info | ✓ | |
| [PC0026](pc0026/) | Mandatory field missing on API page | Warning | ✓ | ✓ |
| [PC0027](pc0027/) | Temporary record trigger invocation | Warning | ✓ | |
| [PC0028](pc0028/) | TableRelation field length | Warning | ✓ | |
