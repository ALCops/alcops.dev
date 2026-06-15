+++
title = 'Procedure or trigger declaration should not end with semicolon'
linkTitle = 'FC0001'

[params]
  id = 'FC0001'
  severity = 'Warning'
  category = 'Design'
  codeAction = false
  ignoreObsolete = true
+++

In AL the semicolon is a statement separator, not a statement terminator. A semicolon after a procedure or trigger declaration suggests that another statement follows — but what follows is the `begin...end` body, not a statement. The compiler accepts the trailing semicolon, but it contradicts the language's own syntax rules.

Remove the trailing semicolon from the declaration.

### Example

{{< highlight al "hl_lines=3" >}}
codeunit 50100 MyCodeunit
{
    procedure CalculateTotal(Quantity: Decimal; UnitPrice: Decimal): Decimal; // Procedure or trigger declaration should not end with semicolon [FC0001]
    begin
        exit(Quantity * UnitPrice);
    end;
}
{{< /highlight >}}

Remove the semicolon from the declaration:

{{< highlight al "hl_lines=3" >}}
codeunit 50100 MyCodeunit
{
    procedure CalculateTotal(Quantity: Decimal; UnitPrice: Decimal): Decimal
    begin
        exit(Quantity * UnitPrice);
    end;
}
{{< /highlight >}}

### Why

We'll have to go all-the-way back to the [C/SIDE Introduction in Microsoft Dynamics NAV 2009](cside-nav2009.png):

> A semicolon (statement separator) suggests that a new statement is coming up.

And if we even go further back, on the [C/SIDE Application Designer's Guide, dated 1996](cside-1996.png), we find:

> A common error for the C/AL newcomers is to put an extraneous semicolon at the end of a line before an ELSE. As mentioned above, this is not valid according to the syntax of C/AL, as the semicolon is used as a statement separator.

So based on the documentation, syntactically the ";" is not something that ends a statement in (Pasc)AL, but it actually is a separator symbol to separate statements. That's why procedure declaration should not end with semicolon and also the reason why you can omit it for the last statement.

### Exception

Interface methods, control add-in procedures, and control add-in events have no body — the semicolon is required by the compiler to terminate the declaration:

{{< highlight al "hl_lines=3" >}}
interface MyInterface
{
    procedure MyProcedure();
}
{{< /highlight >}}
