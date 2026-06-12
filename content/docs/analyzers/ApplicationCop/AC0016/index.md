+++
title = 'Do not use line breaks in ToolTip (AC0016)'
linkTitle = 'AC0016'

[params]
  severity = 'Info'
  category = 'Design'
  codeAction = false
  ignoreObsolete = true
+++

Developers sometimes add a `\` escape sequence to split a ToolTip across two lines. Business Central does not render formatting in ToolTips — the backslash appears literally in the client:

{{< imgproc "tooltip-linebreak.png" Fit "815x285" >}}
The backslash renders as literal text instead of a line break.
{{< /imgproc >}}

> Do not use line breaks in the tooltip text.
> The tooltip cannot render formatting or line breaks.
>
> — [Guidelines for tooltip text](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/user-assistance#guidelines-for-tooltip-text) on Microsoft Learn

Remove the escape sequence and keep the text as a single continuous sentence.

### Example

{{< highlight al "hl_lines=9" >}}
page 50100 "My Page"
{
    layout
    {
        area(Content)
        {
            field(MyField; Rec.MyField)
            {
                ToolTip = 'My first line of text\My second line of text'; // Do not use line breaks in ToolTip [AC0016]
            }
        }
    }
}
{{< /highlight >}}

Remove the line break and combine the text into a single sentence:

{{< highlight al "hl_lines=9" >}}
page 50100 "My Page"
{
    layout
    {
        area(Content)
        {
            field(MyField; Rec.MyField)
            {
                ToolTip = 'My first line of text. My second line of text.';
            }
        }
    }
}
{{< /highlight >}}

### See also

- [AC0014](../ac0014/) - ToolTip must end with a dot
- [AC0015](../ac0015/) - ToolTip should start with 'Specifies'
- [AC0017](../ac0017/) - ToolTip should not exceed 200 characters
