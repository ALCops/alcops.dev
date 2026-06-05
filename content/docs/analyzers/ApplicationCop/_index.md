---
title: "ApplicationCop"
type: docs
no_list: true
---

The ApplicationCop enforces Business Central application conventions and design standards. It validates how tables, pages, enums, labels, permissions, and metadata should be modeled. Violating these rules won’t break your code at the platform level, but leads to inconsistent user experience, non-standard extensions, or objects that don’t integrate well with the broader Business Central application framework

## Rules

| ID | Title | Severity | Enabled | CodeFix |
|---|---|---|---|---|
| [AC0001](ac0001/) | DrillDownPageId and LookupPageId must be defined for tables used in list pages | Info | ✓ | |
| [AC0002](ac0002/) | Single-field primary key requires the NotBlank property | Warning | ✓ | ✓ |
| [AC0003](ac0003/) | Set NotBlank property to false when No. Series TableRelation exists | Warning | ✓ | ✓ |
| [AC0004](ac0004/) | Confirm() must be implemented through the Confirm Management codeunit | Info | ✓ | ✓ |
| [AC0005](ac0005/) | GlobalLanguage() must be implemented through the Translation Helper codeunit | Info | ✓ | ✓ |
| [AC0006](ac0006/) | Use the Page Management codeunit instead of invoking Page.Run directly | Warning | ✓ | ✓ |
| [AC0007](ac0007/) | Install and Upgrade codeunits should have Access set to Internal | Warning | ✓ | ✓ |
| [AC0008](ac0008/) | DataPerCompany must be explicitly set on table objects | Info | — | |
| [AC0009](ac0009/) | The Caption of permissionset objects should not exceed the maximum length | Warning | ✓ | ✓ |
| [AC0010](ac0010/) | All application objects must be covered by a PermissionSet | Warning | ✓ | |
| [AC0011](ac0011/) | Captions must be defined on user-facing objects and controls | Info | ✓ | |
| [AC0012](ac0012/) | Integration events must not be declared in codeunits with Access set to Internal | Warning | ✓ | ✓ |
| [AC0013](ac0013/) | DropDown and Brick fieldgroups must be defined | Info | — | |
| [AC0014](ac0014/) | ToolTip must end with a dot | Info | ✓ | |
| [AC0015](ac0015/) | ToolTip should start with Specifies | Info | ✓ | |
| [AC0016](ac0016/) | Do not use line breaks in ToolTip | Info | ✓ | |
| [AC0017](ac0017/) | ToolTip should not exceed 200 characters | Info | ✓ | |
| [AC0018](ac0018/) | Empty captions should be locked | Warning | ✓ | ✓ |
| [AC0019](ac0019/) | Reserve Enum value zero (0) for empty value | Info | ✓ | |
| [AC0020](ac0020/) | Labels suffixed with Tok must be locked | Warning | ✓ | ✓ |
| [AC0021](ac0021/) | Locked Label must have a suffix Tok | Info | — | |
| [AC0022](ac0022/) | Empty Enum value should not have a Caption property specified | Warning | ✓ | |
| [AC0023](ac0023/) | Enum value must have non-empty Caption to be selectable in the client | Warning | ✓ | |
| [AC0024](ac0024/) | Event publisher methods should not be public | Warning | ✓ | ✓ |
| [AC0025](ac0025/) | Use the (CR)LFSeparator from the Type Helper codeunit | Info | ✓ | |
| [AC0026](ac0026/) | Explicitly set AllowInCustomizations for excluded fields | Info | ✓ | |
| [AC0027](ac0027/) | Use the Tok suffix for token labels | Info | ✓ | |
| [AC0028](ac0028/) | Table field must define a ToolTip | Info | ✓ | |
| [AC0029](ac0029/) | Duplicate ToolTip between page and table field | Info | ✓ | |
| [AC0030](ac0030/) | Use return value for better error handling | Info | ✓ | |
| [AC0031](ac0031/) | Informs the user that there are missing permission to access tabledata | Info | ✓ | ✓ |
| [AC0032](ac0032/) | Unused permission declared | Info | ✓ | ✓ |

**Note:** Rules marked with "—" in the Enabled column are disabled by default and must be explicitly enabled in your project's ruleset file.