+++
title = 'ToolTip should not exceed 200 characters (AC0017)'
linkTitle = 'AC0017'

[params]
  severity = 'Info'
  category = 'Design'
  codeAction = false
  ignoreObsolete = true
+++

The [user assistance model](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/user-assistance) recommends keeping ToolTips short so users can scan them and get unblocked quickly. A ToolTip within the 200-character limit fits neatly into the popup:

{{< imgproc "tooltip-short.png" Fit "526x306" >}}
A ToolTip within the recommended length.
{{< /imgproc >}}

The UI *can* render longer text — up to roughly 600 characters before scrolling kicks in — but a wall of text defeats the purpose. The user now has to read instead of scan:

{{< imgproc "tooltip-long.png" Fit "815x400" >}}
A long ToolTip expands into a block that overwhelms the page.
{{< /imgproc >}}

> Try to not exceed 200 characters including spaces.
>
> This makes the tooltip easier to scan so the user can get unblocked quickly. However, the UI will render longer tooltip text if you want to provide more detailed user assistance.
>
> — [Guidelines for tooltip text](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/user-assistance#guidelines-for-tooltip-text) on Microsoft Learn

Shorten the ToolTip to focus on the single most important piece of information the user needs.

### Example

The following page field has a ToolTip that exceeds the recommended length:

{{< highlight al "hl_lines=9" >}}
page 50100 "My Page"
{
    layout
    {
        area(Content)
        {
            field(MyField; Rec.MyField)
            {
                ToolTip = 'Specifies the value that is used for this particular field and contains a very long description that exceeds the recommended maximum length of 200 characters, which makes it harder for users to quickly scan and understand the purpose of this field.'; // ToolTip should not exceed 200 characters [AC0017]
            }
        }
    }
}
{{< /highlight >}}

Shorten the ToolTip to the essential information:

{{< highlight al "hl_lines=9" >}}
page 50100 "My Page"
{
    layout
    {
        area(Content)
        {
            field(MyField; Rec.MyField)
            {
                ToolTip = 'Specifies the value used for this field.';
            }
        }
    }
}
{{< /highlight >}}

### See also

- [AC0014](../ac0014/) - ToolTip must end with a dot
- [AC0015](../ac0015/) - ToolTip should start with 'Specifies'
- [AC0016](../ac0016/) - Do not use line breaks in ToolTip
