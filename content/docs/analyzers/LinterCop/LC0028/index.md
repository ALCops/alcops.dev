+++
title = 'Event subscriber arguments should use identifier syntax (LC0028)'
linkTitle = 'LC0028'
+++

Since Business Central 2023 wave 1 (BC22), event subscriber arguments support **identifier syntax** instead of string literals. Using identifiers unlocks powerful Visual Studio Code navigation features such as Go To Definition, Go To References, tooltips, and CodeLens reference counts.

This rule reports a diagnostic when event subscriber attribute parameters still use the old **string literal** syntax (single quotes) instead of the new **identifier** syntax.

## Why this matters

When event subscriber arguments use string literals, the AL Language extension cannot resolve the reference. This means no navigability — you can't jump to the event publisher, see who else subscribes, or get tooltips.

Switching to identifier syntax enables:

- **Go To Definition (F12)** — Jump directly to the event publisher method.
- **Go To References (Shift+F12)** — See all subscribers of a given event.
- **Tooltips** — Hover over the event name to see its definition.
- **CodeLens** — See how many references exist for each event publisher.

{{< imgproc "go-to-definition.png" Fit "800x600" >}}
Go To Definition navigates directly to the event publisher.
{{< /imgproc >}}

## Example

The following event subscriber uses the old **string literal** syntax with single quotes around the event publisher name and the event name:

{{< highlight al "hl_lines=3" >}}
codeunit 50100 MyCodeunit
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnAfterPostSalesDoc', '', false, false)]
    local procedure OnAfterPost(var SalesHeader: Record "Sales Header")
    begin
    end;
}
{{< /highlight >}}

{{< imgproc "old-string-syntax.png" Fit "800x300" >}}
Old syntax: event name in single quotes — no navigability.
{{< /imgproc >}}

To fix this, replace the single-quoted string literals with identifiers. For simple names, remove the quotes entirely. For names containing spaces, use double quotes instead:

{{< highlight al "hl_lines=3" >}}
codeunit 50100 MyCodeunit
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", OnAfterPostSalesDoc, '', false, false)]
    local procedure OnAfterPost(var SalesHeader: Record "Sales Header")
    begin
    end;
}
{{< /highlight >}}


{{< imgproc "new-identifier-syntax.png" Fit "800x300" >}}
New syntax: event name as identifier — full VS Code navigability.
{{< /imgproc >}}

## Code action

A code action is available for this diagnostic. The AL Language extension provides a built-in code action to convert event subscriber parameters from string literal to identifier syntax. This can be applied to a single instance, the active file, the active project, or the entire workspace.

{{< imgproc "code-action.png" Fit "800x300" >}}
Use the AL code action to convert from string literals to identifiers.
{{< /imgproc >}}

## See also

- [Code navigability on event subscribers](https://learn.microsoft.com/en-us/dynamics365/release-plan/2023wave1/smb/dynamics365-business-central/code-navigability-event-subscribers) on Microsoft Learn
- [Subscribing to Events](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-subscribing-to-events) on Microsoft Learn
- [Business Central 2023 wave 1 (BC22) new features: Event navigability](https://yzhums.com/36573/) by ZHU on Dynamics 365 Lab

{{% alert title="Note" %}}
Images on this page are courtesy of [ZHU](https://yzhums.com/36573/) from Dynamics 365 Lab.
{{% /alert %}}
