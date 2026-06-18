+++
title = 'Use identifier syntax in event subscriber arguments'
linkTitle = 'LC0028'

[params]
  id = 'LC0028'
  severity = 'Warning'
  category = 'Design'
  codeAction = false
  ignoreObsolete = false
+++

Event subscribers declare their target through parameters in the `[EventSubscriber]` attribute. Before Business Central 2023 wave 1 (BC22), these parameters required string literals — single-quoted names that the AL Language extension cannot resolve. With string literals, Go To Definition (F12) does not work on the event name, reference counts are missing, and hovering shows no tooltip. The subscriber is effectively invisible to VS Code's navigation.

Since BC22, the same parameters accept identifier syntax. Removing the single quotes (or replacing them with double quotes for names that contain spaces) enables full VS Code navigation: Go To Definition, Go To References, tooltips, and CodeLens reference counts.

{{< imgproc "go-to-definition.png" Fit "800x600" >}}
Identifier syntax enables Go To Definition, Go To References, tooltips, and CodeLens.
{{< /imgproc >}}

### Example

{{< highlight al "hl_lines=3" >}}
codeunit 50100 MyCodeunit
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", 'OnAfterPostSalesDoc', '', false, false)] // Use identifier syntax in event subscriber arguments [LC0028]
    local procedure OnAfterPost(var SalesHeader: Record "Sales Header")
    begin
    end;
}
{{< /highlight >}}

{{< imgproc "old-string-syntax.png" Fit "800x600" >}}
String syntax: the event name in single quotes has no navigability.
{{< /imgproc >}}

Replace the single-quoted string literal with an identifier — remove the quotes for simple names, or use double quotes for names that contain spaces:

{{< highlight al "hl_lines=3" >}}
codeunit 50100 MyCodeunit
{
    [EventSubscriber(ObjectType::Codeunit, Codeunit::"Sales-Post", OnAfterPostSalesDoc, '', false, false)]
    local procedure OnAfterPost(var SalesHeader: Record "Sales Header")
    begin
    end;
}
{{< /highlight >}}

{{< imgproc "new-identifier-syntax.png" Fit "800x600" >}}
Identifier syntax: full VS Code navigation on the event name.
{{< /imgproc >}}

{{% alert title="Tip" %}}
The AL Language extension provides a built-in code action to convert event subscriber parameters from string literals to identifiers. This can be applied to a single instance, the active file, the active project, or the entire workspace.
{{% /alert %}}

{{< imgproc "code-action.png" Fit "800x600" >}}
AL Language extension code action for converting string literals to identifiers.
{{< /imgproc >}}

### See also

- [Code navigability on event subscribers](https://learn.microsoft.com/en-us/dynamics365/release-plan/2023wave1/smb/dynamics365-business-central/code-navigability-event-subscribers) on Microsoft Learn
- [Subscribing to Events](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-subscribing-to-events) on Microsoft Learn
- [Business Central 2023 wave 1 (BC22) new features: Event navigability](https://yzhums.com/36573/) by ZHU on Dynamics 365 Lab

{{% alert title="Note" %}}
Images on this page are courtesy of [ZHU](https://yzhums.com/36573/) from Dynamics 365 Lab.
{{% /alert %}}
