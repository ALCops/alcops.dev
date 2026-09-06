+++
title = 'Set NotBlank property to false when No. Series TableRelation exists'
linkTitle = 'AC0003'

[params]
  id = 'AC0003'
  severity = 'Warning'
  category = 'Design'
  codeAction = true
  ignoreObsolete = true
+++

When a table includes a field with a `TableRelation` to "No. Series", the primary key value is typically assigned in the `OnInsert` trigger through number series management. The platform validates `NotBlank` on the primary key field *before* the `OnInsert` trigger runs.

If `NotBlank = true` on the primary key, creating a new record from the client fails immediately. The user sees a validation error before the number series logic gets a chance to assign a value.

{{< imgproc "notblank-error.png" Fit "560x152" >}}
The NotBlank validation fires before the OnInsert trigger assigns a number.
{{< /imgproc >}}

Set `NotBlank = false` on the primary key field, or remove the property entirely.

### Example

The following table has a "No. Series" relationship but incorrectly sets `NotBlank = true` on the primary key:

{{< highlight al "hl_lines=7" >}}
table 50100 "Service Request"
{
    fields
    {
        field(1; "No."; Code[20])
        {
            NotBlank = true; // Set NotBlank property to false when 'No. Series' TableRelation exists [AC0003]
        }
        field(2; Description; Text[100]) { }
        field(3; Type; Enum MyType) { }
        field(4; "No. Series"; Code[20])
        {
            TableRelation = "No. Series";
        }
    }

    keys
    {
        key(PK; "No.")
        {
            Clustered = true;
        }
    }

    trigger OnInsert()
    var
        NoSeries: Codeunit "No. Series";
    begin
        Rec."No." := NoSeries.GetNextNo('MyNumberSeriesFromSetup');
    end;
}
{{< /highlight >}}

### When the diagnostic is reported

- The table has a single-field primary key of type Code or Text with `NotBlank = true`
- Any field in the table has a `TableRelation` pointing to "No. Series"
- The primary key field is not named "Name" (see exception below)

### Journal Templates

There is an exception to this rule, for (custom) Journal Template tables.

Journal Template tables use “Name” as their primary key rather than “No.” and require NotBlank = true to prevent blank template names, even though they include a “No. Series” TableRelation. The rule excludes tables where the primary key field is named “Name”.

{{< highlight al "hl_lines=1" >}}
field(1; Name; Code[10])
{
    Caption = 'Name';
    NotBlank = true;
}
...
field(16; "No. Series"; Code[20])
{
    Caption = 'No. Series';
    TableRelation = "No. Series";

    trigger OnValidate()
    begin
        if "No. Series" <> '' then begin
            if Recurring then
                Error(
                    Text000,
                    FieldCaption("Posting No. Series"));
            if "No. Series" = "Posting No. Series" then
                "Posting No. Series" := '';
        end;
    end;
}
{{< /highlight >}}

### See also

- [NotBlank Property](https://learn.microsoft.com/dynamics365/business-central/dev-itpro/developer/properties/devenv-notblank-property) on Microsoft Learn
- [AC0002](../ac0002/) — Single-field primary key requires the NotBlank property
