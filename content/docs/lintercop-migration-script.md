---
title: "LinterCop Migration Script"
linkTitle: "LinterCop Migration Script"
weight: 31
---

This PowerShell script automates migration from `LCxxxx` rule IDs to ALCops rule IDs.

It updates:

- `#pragma warning disable|enable|restore ...` rule IDs in `.al` files
- Rule IDs in `*.ruleset.json` files

Run it against a folder (recursive) or a single file:

```powershell
# Run in current folder and subfolders
Convert-Analyzer -LiteralPath .

# Update a single file
Convert-Analyzer -LiteralPath 'SomeCodeunit.Codeunit.al'
```

```powershell
#Requires -Version 7

function Convert-Analyzer {
    [CmdletBinding()]
    param (
        [Parameter()]
        [string] $LiteralPath
    )

    Set-StrictMode -Version Latest
    $ErrorActionPreference = 'Stop'
    $ProgressPreference = 'Continue'

    $RuleMapping = @{
        'LC0001' = 'PC0001'
        'LC0002' = 'DC0001'
        'LC0003' = 'LC0003'
        'LC0004' = 'AC0001'
        'LC0005' = 'FC0002'
        'LC0006' = 'PC0002'
        'LC0007' = 'AC0008'
        'LC0008' = 'PC0003'
        'LC0009' = 'LC0007, LC0009'
        'LC0010' = 'LC0008, LC0010'
        'LC0011' = 'PC0006'
        'LC0012' = 'LC0003'
        'LC0013' = 'AC0002'
        'LC0014' = 'AC0009'
        'LC0015' = 'AC0010'
        'LC0016' = 'AC0011'
        'LC0017' = 'DC0002'
        'LC0018' = 'AC0012'
        'LC0019' = 'LC0019'
        'LC0020' = 'LC0020'
        'LC0021' = 'AC0004'
        'LC0022' = 'AC0005'
        'LC0023' = 'AC0013'
        'LC0024' = 'FC0001'
        'LC0025' = 'DC0004'
        'LC0026' = 'AC0014'
        'LC0027' = 'AC0006'
        'LC0028' = 'LC0028'
        'LC0029' = 'REMOVED_LC0029'
        'LC0030' = 'AC0007'
        'LC0031' = 'LC0031'
        'LC0032' = 'PC0016'
        'LC0033' = 'LC0033'
        'LC0034' = 'PC0005'
        'LC0035' = 'AC0026'
        'LC0036' = 'AC0015'
        'LC0037' = 'AC0016'
        'LC0038' = 'AC0017'
        'LC0039' = 'PC0017'
        'LC0040' = 'LC0040'
        'LC0041' = 'AC0018'
        'LC0042' = 'PC0007'
        'LC0043' = 'LC0043'
        'LC0044' = 'PC0020, PC0021'
        'LC0045' = 'AC0019'
        'LC0046' = 'AC0020'
        'LC0047' = 'AC0021'
        'LC0048' = 'LC0048'
        'LC0049' = 'PC0018'
        'LC0050' = 'PC0008'
        'LC0051' = 'PC0022'
        'LC0052' = 'LC0052'
        'LC0053' = 'LC0053'
        'LC0054' = 'LC0054'
        'LC0055' = 'AC0027'
        'LC0056' = 'AC0022'
        'LC0057' = 'AC0023'
        'LC0058' = 'PC0036'
        'LC0059' = 'PC0019'
        'LC0060' = 'PC0024'
        'LC0061' = 'PC0025'
        'LC0062' = 'PC0026'
        'LC0063' = 'LC0063'
        'LC0064' = 'AC0028'
        'LC0065' = 'PC0010'
        'LC0066' = 'AC0029'
        'LC0067' = 'AC0003'
        'LC0068' = 'AC0031'
        'LC0069' = 'DC0003'
        'LC0070' = 'PC0004'
        'LC0071' = 'PC0023'
        'LC0072' = 'DC0005'
        'LC0073' = 'PC0011'
        'LC0074' = 'PC0012'
        'LC0075' = 'PC0013'
        'LC0076' = 'PC0028'
        'LC0077' = 'FC0003'
        'LC0078' = 'PC0027'
        'LC0079' = 'AC0024'
        'LC0080' = 'PC0014'
        'LC0081' = 'LC0081'
        'LC0082' = 'LC0082'
        'LC0083' = 'LC0083'
        'LC0084' = 'AC0030'
        'LC0085' = 'AC0025'
        'LC0086' = 'LC0086'
        'LC0087' = 'PC0015'
        'LC0088' = 'LC0088'
        'LC0089' = 'LC0089'
        'LC0090' = 'LC0090'
        'LC0091' = 'LC0091'
        'LC0092' = 'LC0092'
        'LC0093' = 'TA0001'
        'LC0094' = 'LC0096'
    }

    $FilePaths = @(
        if (Test-Path -LiteralPath $LiteralPath -PathType Container) {
            Get-ChildItem -LiteralPath $LiteralPath -Recurse -Include '*.al', 'al.ruleset.json', '*.al.ruleset.json' |
            Select-Object -ExpandProperty FullName
        }
        else {
            $LiteralPath
        }
    )

    $Activity = @{ Activity = 'Convert analyzer IDs in AL source code files and AL ruleset files' }
    Write-Progress @Activity
    foreach ($FilePath in $FilePaths) {
        $FileActivity = $Activity + @{ Status = "Converting analyzer IDs in file '$FilePath'..." }
        Write-Progress @FileActivity

        $OldContent = [System.IO.File]::ReadAllText($FilePath)
        $NewContent = $(
            switch -Regex ($FilePath) {
                '\.al$' {
                    $OldContent -creplace '(?<Prefix>\s*#pragma\s+warning\s+(?:disable|enable|restore)\s+)(?<AnalyzerIds>\w+(\s*,\s*\w+)*)', {
                        $Prefix = $_.Groups['Prefix'].Value
                        $OldAnalyzerIds = $_.Groups['AnalyzerIds'].Value

                        $NewAnalyzerIds = $(
                            $OldAnalyzerIds -replace '\w+', {
                                $RuleMapping[$_.Value] ?? $_.Value
                            }
                        )

                        $Prefix + $NewAnalyzerIds
                    }
                }
                '\.ruleset\.json$' {
                    $RuleSet = $OldContent | ConvertFrom-Json -AsHashtable

                    <# NOTE: Uncomment and edit to remap included ruleset paths.
                    if ($RuleSet.ContainsKey('includedRuleSets')) {
                        foreach ($IncludedRuleSet in $RuleSet.includedRuleSets) {
                            $IncludedRuleSet.path = $(
                                $IncludedRuleSet.path.Replace(
                                    'https://softerabalticeune.blob.core.windows.net/al-rulesets/',
                                    'https://softerabalticeune.blob.core.windows.net/al-rulesets-v2/')
                            )
                        }
                    }
                    #>

                    if ($RuleSet.ContainsKey('rules')) {
                        $OldRules = $RuleSet['rules']

                        $NewRules = @(
                            foreach ($OldRule in $OldRules) {
                                $OldAnalyzerId = $OldRule.id

                                $NewAnalyzerIdsText = $RuleMapping[$OldAnalyzerId] ?? $OldAnalyzerId

                                if ($NewAnalyzerIdsText -cne $OldAnalyzerId) {
                                    $NewAnalyzerIds = $NewAnalyzerIdsText -split ', '
                                    foreach ($NewAnalyzerId in $NewAnalyzerIds) {
                                        $NewRule = $OldRule.Clone()
                                        $NewRule.id = $NewAnalyzerId
                                        $NewRule
                                    }
                                }
                                else {
                                    $OldRule
                                }
                            }
                        )

                        $RuleSet['rules'] = $NewRules
                    }

                    $JsonSerializerOptions = New-Object -TypeName System.Text.Json.JsonSerializerOptions -ArgumentList @(
                        [System.Text.Json.JsonSerializerOptions]::Web
                    )
                    $JsonSerializerOptions.Encoder = [System.Text.Encodings.Web.JavaScriptEncoder]::UnsafeRelaxedJsonEscaping
                    $JsonSerializerOptions.WriteIndented = $true
                    $JsonSerializerOptions.IndentCharacter = ' '
                    $JsonSerializerOptions.IndentSize = 4

                    [System.Text.Json.JsonSerializer]::Serialize($RuleSet, $JsonSerializerOptions) + "`n"
                }
                default {
                    $OldContent
                }
            }
        )

        if ($NewContent -cne $OldContent) {
            [System.IO.File]::WriteAllText($FilePath, $NewContent)
        }
    }
    Write-Progress @Activity -Completed
}
```

Source: [Migration from LinterCop #317](https://github.com/ALCops/Analyzers/discussions/317)
