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
| [PC0002](pc0002/) | AutoIncrement fields are not supported in temporary tables | Error | ✓ | |
| [PC0003](pc0003/) | Filter operators should not be used in SetRange | Error | ✓ | ✓ |
| [PC0004](pc0004/) | List objects are one-based | Error | ✓ | |
| [PC0005](pc0005/) | Explicitly set Extensible property on public objects | Hidden | | ✓ |
| [PC0006](pc0006/) | Every object needs to specify a value for the Access property | Hidden | | |
| [PC0007](pc0007/) | AutoCalcFields should only be used for FlowFields or Blob fields | Error | ✓ | |
| [PC0008](pc0008/) | Avoid unsupported operators with placeholders in SetFilter expressions | Error | ✓ | ✓ |
| [PC0010](pc0010/) | Event subscriber var keyword mismatch | Warning | ✓ | ✓ |
| [PC0011](pc0011/) | Handled parameters in event signatures should be passed by var | Warning | ✓ | ✓ |
| [PC0012](pc0012/) | Set values for FlowFilter fields using filtering methods | Warning | ✓ | |
| [PC0013](pc0013/) | Record.Get procedure arguments | Error | ✓ | |
| [PC0014](pc0014/) | Use two single quotes instead of double quotes in JPath expressions | Warning | ✓ | ✓ |
| [PC0015](pc0015/) | Guid empty string comparison | Error | ✓ | ✓ |
| [PC0016](pc0016/) | Clear/ClearAll does not reset SingleInstance codeunit state | Warning | ✓ | |
| [PC0017](pc0017/) | Page record argument does not match source table | Warning | ✓ | |
| [PC0018](pc0018/) | Page record methods require a SourceTable | Error | ✓ | |
| [PC0019](pc0019/) | Invalid single-quote escaping in filter string | Warning | ✓ | ✓ |
| [PC0020](pc0020/) | Incompatible field types across TransferFields | Warning | ✓ | |
| [PC0021](pc0021/) | Inconsistent field names across TransferFields | Warning | ✓ | |
| [PC0022](pc0022/) | Possible overflow assigning | Warning | ✓ | ✓ |
| [PC0023](pc0023/) | IsHandled parameter assignment | Warning | ✓ | |
| [PC0024](pc0024/) | ApplicationArea property is not applicable to API pages | Info | ✓ | ✓ |
| [PC0025](pc0025/) | ODataKeyFields should use SystemId | Error | ✓ | |
| [PC0026](pc0026/) | Mandatory field missing on API page | Warning | ✓ | ✓ |
| [PC0027](pc0027/) | Avoid triggering table logic on temporary records | Warning | ✓ | |
| [PC0028](pc0028/) | Table relation field length mismatch | Warning | ✓ | |
| [PC0029](pc0029/) | Use CreateSequentialGuid for key fields | Info | ✓ | ✓ |
| [PC0030](pc0030/) | Use partial records on read operation | Info | ✓ | ✓ |
| [PC0031](pc0031/) | Partial records cause JIT load on full-field operations | Warning | ✓ | ✓ |
| [PC0032](pc0032/) | Report layout property exceeds maximum length | Error | ✓ | |
| [PC0033](pc0033/) | Duplicate OData EntityName on page controls | Warning | ✓ | |
| [PC0034](pc0034/) | Placeholder argument count mismatch | Warning | ✓ | |
| [PC0035](pc0035/) | Use SetAutoCalcFields for loops | Warning | ✓ | ✓ |
| [PC0036](pc0036/) | SetRecord() does not support temporary records | Warning | ✓ | |
| [PC0037](pc0037/) | Use Validate() instead of direct field assignment | Warning | ✓ | ✓ |
| [PC0038](pc0038/) | Not all code paths return a value | Info | ✓ | |
