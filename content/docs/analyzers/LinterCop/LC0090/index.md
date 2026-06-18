+++
title = 'Cognitive Complexity threshold exceeded'
linkTitle = 'LC0090'

[params]
  id = 'LC0090'
  severity = 'Warning'
  category = 'Design'
  codeAction = false
  ignoreObsolete = true
+++

Cognitive Complexity measures how difficult a procedure is to understand. Unlike [Cyclomatic Complexity](../lc0010/), which counts every decision point equally, Cognitive Complexity penalizes nesting: each construct inside a nested block costs more than the same construct at the top level. Five nested `if` statements cost 1+2+3+4+5 = 15 points; the same five conditions written as guard clauses cost zero.

A procedure that exceeds the threshold forces any reviewer to hold multiple levels of context in working memory before reasoning about the deepest branches — the kind of code where a small change in one condition has consequences three levels down that are easy to miss. Extract helper procedures or use guard clauses to flatten the nesting.

This diagnostic fires when a procedure's Cognitive Complexity meets or exceeds the configured threshold (default: **15**). For the full scoring model, see [LC0089](../lc0089/).

### Basic criteria and methodology
A Cognitive Complexity score is assessed according to three basic rules:

1. Ignore structures that allow multiple statements to be readably **shorthanded** into one
2. Increment (add one) for each **break** in the linear flow of the code
3. Increment when flow-breaking structures are **nested**

Additionally, a complexity score is made up of four different types of increments:

* Nesting - assessed for nesting control flow structures inside each other
* Structural - assessed on control flow structures that are subject to a nesting increment, and that increase the nesting count
* Fundamental - assessed on statements not subject to a nesting increment
* Hybrid - assessed on control flow structures that are not subject to a nesting increment, but which do increase the nesting count

While the type of an increment makes no difference in the math - each increment adds one to the final score - making a distinction among the 
categories of features being counted makes it easier to understand where nesting increments do and do not apply.

### An illustration of the problem
```AL
procedure SumOfPrimes(Max: Integer): Integer
var
    Total: Integer;
    I, J : Integer;
    IsPrime: Boolean;
begin
    Total := 0;
    for I := 1 to Max do begin
        IsPrime := true;
        for J := 2 to I - 1 do begin
            if (I mod J = 0) then begin
                IsPrime := false;
                break;
            end;
        end;
        if IsPrime then
            Total += I;
    end;
    exit(Total);
end;               // Cyclomatic Complexity: 5
```

```AL
procedure GetWords(Number: Integer): Text
begin
    case Number of
        1:
            exit('one');
        2:
            exit('a couple');
        3:
            exit('a few');
        4:
            exit('a quadruple');
        else
            exit('lots');
    end;
end;               // Cyclomatic Complexity: 5
```

While Cyclomatic Complexity gives equal weight to both the SumOfPrimes and GetWords methods, it is apparent that SumOfPrimes is much more complex and difficult to understand than GetWords. This illustrates that measuring understandability based solely on the paths of a program may not be sufficient.

### Intuitively 'right' complexity scores
```AL
procedure SumOfPrimes(Max: Integer): Integer
var
    Total: Integer;
    I, J : Integer;
    IsPrime: Boolean;
begin
    Total := 0;
    for I := 1 to Max do begin          // +1
        IsPrime := true;
        for J := 2 to I - 1 do begin    // +2
            if (I mod J = 0) then begin // +3
                IsPrime := false;
                break;
            end;
        end;
        if IsPrime then                 // +2
            Total += I;
    end;
    exit(Total);
end;               // Cognitive Complexity: 8
```

```AL
procedure GetWords(Number: Integer): Text
begin
    case Number of                      // +1
        1:
            exit('one');
        2:
            exit('a couple');
        3:
            exit('a few');
        4:
            exit('a quadruple');
        else
            exit('lots');
    end;
end;               // Cognitive Complexity: 1
```

Looking again again at the examples where the Cognitive Complexity algorithm gives these two methods markedly different scores, ones that are far more reflective of their relative understandability.

With Cyclomatic Complexity, an procedure with [early exits](https://thatnavguy.com/early-exit/) and [case statement](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-al-control-statements#case-statements), can have the same number of decision points. However, Cognitive Complexity addresses this limitation by not incrementing for each decision point, making it easier to compare the metric values.

### What is Cognitive?
To understand why the cognitive aspect is so important, check out the excellent article [Cognitive load is what matters](https://minds.md/zakirullin/cognitive), which provides a great introduction to the topic.

Diving deeper into Cognitive Complexity, you can start with [this blog post from Sonar](https://www.sonarsource.com/blog/cognitive-complexity-because-testability-understandability/), which includes a link to the [whitepaper](https://www.sonarsource.com/docs/CognitiveComplexity.pdf).  

Koh Hom's blog also features a great article, [Introducing Code Complexity Metric: Cognitive Complexity](https://clean99.github.io/2023/03/04/Introducing-Code-Complexity-Metric-Cognitive-Complexity/).  

Additionally, the SciTools blog has an excellent article on the [Cognitive Complexity Metric Plugin](https://blog.scitools.com/cognitive-complexity-metric-plugin/) to give you a detailed understanding of Cognitive Complexity metric.

### The AL Language
The following increments are supported for the AL Language extension for Microsoft Dynamics 365 Business Central.

Category | Increment | Nesting Level | Nesting Penalty
|--|:-:|:-:|:-:|
if, ternary operator | X | X | X |
else, else if [*](#compensating-usages) | X | X |
case | X | X | X
for, foreach | X | X | X
while, repeat | X | X | X
sequences of binary logical operators | X |   |
each method in a recursion cycle | X |   |

### Example
```AL
procedure VerifyLineQuantity()
begin
    if Type = Type::Item then begin                 // +1 (1 increment + 0 nesting penalty)
        if "Unit of Measure Code" <> '' then        // +2 (1 increment + 1 nesting penalty)
            if Status = Status::Open then           // +3 (1 increment + 2 nesting penalty)
                repeat                              // +4 (1 increment + 3 nesting penalty)
                    if Quantity < 0 then            // +5 (1 increment + 4 nesting penalty)
                        HandleNegativeQuantity();
                    if Quantity = 0 then            // +5 (1 increment + 4 nesting penalty)
                        HandleZeroQuantity();
                    if Quantity > 0 then            // +5 (1 increment + 4 nesting penalty)
                        HandlePositiveQuantity();
                until AllHandled();
    end;
end;                                                // Cognitive Complexity: 25
```

```AL
procedure VerifyLineQuantity()
begin
    if Type <> Type::Item then
        exit;

    if "Unit of Measure Code" = '' then
        exit;

    if Status <> Status::Open then
        exit;

    repeat                                          // +1 (1 increment + 0 nesting penalty)
        if Quantity < 0 then                        // +2 (1 increment + 1 nesting penalty)
            HandleNegativeQuantity();
        if Quantity = 0 then                        // +2 (1 increment + 1 nesting penalty)
            HandleZeroQuantity();
        if Quantity > 0 then                        // +2 (1 increment + 1 nesting penalty)
            HandlePositiveQuantity();
    until AllHandled();
end;                                                // Cognitive Complexity: 7
```

### Increments
{{< imgproc "cognitive-complexity-increments.png" Fit "800x600" />}}

To view the details of the Cognitive Complexity increments, you can enable the `LC0089i` diagnostic. Together with the [Error Lens extension for VS Code](https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens), you can easily see how the Cognitive Complexity is calculated.

```json
{
    "id": "LC0089i",
    "action": "Info",
    "justification": "Show every individual increment metric of the Cognitive Complexity"
}
```


### Logical operators
Cognitive Complexity does not increment for each binary logical operator. Instead, it assesses a fundamental increment for each sequence of binary logical operators. For instance, consider the following pairs
```AL
a and b
a and b and c and d

a or b
a or b or c or d
```

Understanding the second line in each pair isn’t that much harder than understanding the first. On the other hand, there is a marked difference in the 
effort to understand the following two lines:
```AL
a and b and c and d
a or b and c or d
```

Because boolean expressions become more difficult to understand with mixed operators, Cognitive complexity increments for each new sequence of like 
operators. For instance
```AL
if                  // +1 for 'if'
 a and b and        // +1
   c or d or        // +1
     e and f        // +1
then;


if                  // +1 for 'if'
 a and              // +1
   not (b and c)    // +1
then;
```
While Cognitive Complexity offers a "discount" for like operators relative to Cyclomatic Complexity, it does increment for all sequences of binary boolean operators such as those in variable assignments, method invocations, and return statements.

Note: The AL Language does **not** support [Lazy Evaluation](https://thatnavguy.com/d365-business-central-lazy-evaluation/). While using a sequence of binary logical operators can improve readability, it may come at a performance cost during runtime.

### Recursion
Unlike Cyclomatic Complexity, Cognitive Complexity adds a fundamental increment for each method in a recursion cycle, whether direct or indirect. Because Recursion contribute very similar complexity like Loop.

Nesting penalties do not apply to recursions, so only increments are applied regardless of the nesting level.
```AL
procedure SelfRecursive()
begin
    SelfRecursive();        // +1 (increment)
end;
```

```AL
procedure IndirectRecursiveA()
begin
    IndirectRecursiveB();   // +1 (increment)
end;

procedure IndirectRecursiveB()
begin
    IndirectRecursiveC();   // +1 (increment)
end;

procedure IndirectRecursiveC()
begin
    IndirectRecursiveA();   // +1 (increment)
end;
```



### Threshold
What should the limit be?

> I would say there shouldn't be one. Because the essential complexity for a simple calculator app is far, far lower than for a program on the Space Shuttle. And if you try to make the Space Shuttle program fit inside the calculator threshold, you're absolutely going to break something.

_[Primary author of Cognitive Complexity on Stack Overflow](https://stackoverflow.com/a/45084107)_

If you're uncertain about the right threshold, the matrix below could serve as a starting point.

Cognitive Complexity | Code Quality | Readability | Maintainability
-- | -- | -- | --
1-5 | Simple and easy to follow | High | Easy
6-10 | Somewhat complex | Medium | Moderate
11-20 | Complex | Low | Difficult
21+ | Very complex | Poor | Very difficult

### Configuration

### Adjusting the threshold

Add or update the `CognitiveComplexityThreshold` property in the `alcops.json` file at your project root (alongside `app.json`):

```json
{
    "CognitiveComplexityThreshold": 20
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `CognitiveComplexityThreshold` | `integer` | `15` | Procedures with a Cognitive Complexity at or above this value trigger a warning. |

{{% alert title="Note" %}}
After changing `alcops.json` in VS Code, reload the window (`Ctrl+Shift+P` → **Developer: Reload Window**) for the new settings to take effect.
{{% /alert %}}


### Compensating Usages
For AL, which lacks an `else if` structure, an `if` as the only statement in an `else` clause does not incur a nesting penalty. Additionally, there is no increment for the `else` itself. That is, an `else` followed immediately by an `if` is treated as an `else if`, even though syntactically it is not.

```AL
if condition1 then                  // +1 (1 increment + 0 nesting penalty)
    ...
else
    if condition2 then              // +1 (1 increment + 0 nesting penalty)
        ...
    else
        if condition3 then begin    // +1 (1 increment + 0 nesting penalty)
            statement1
            if condition4 then      // +2 (1 increment + 1 nesting penalty)
                ...
        end;
```

### See also

* [Cognitive Complexity, Because Testability != Understandability | Sonar](https://www.sonarsource.com/blog/cognitive-complexity-because-testability-understandability/)
* [Introducing Code Complexity Metric: Cognitive Complexity | clean99.github.io](https://clean99.github.io/2023/03/04/Introducing-Code-Complexity-Metric-Cognitive-Complexity/)
* [Understanding cognitive complexity in software development | DX](https://getdx.com/blog/cognitive-complexity/)
* [Cognitive Complexity Metric Plugin | SciTools Blog](https://blog.scitools.com/cognitive-complexity-metric-plugin/)