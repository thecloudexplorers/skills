---
name: powershell-style
description: Write idiomatic, readable PowerShell scripts following consistent style and structure conventions. Use when authoring or reviewing PowerShell scripts, modules, or pipeline tasks.
---

# PowerShell Style

## Quick start

Every script starts with a standard header block, uses `try/catch` for error handling, and follows verb-noun function names.

```powershell
#----------------------------------------------------------------------
# Script: Do-Something.ps1
# Description: One-line summary of what this script does.
#----------------------------------------------------------------------

[CmdletBinding()]
param(
    [Parameter(Mandatory)]
    [string] $TargetPath
)

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

try {
    Write-Host "Processing $TargetPath..."
    # ... logic here
} catch {
    Write-Error "Failed: $_"
    exit 1
}
```

See [references/script-header-template.md](references/script-header-template.md) for the full header template.

## Conventions

### Naming

| Item | Convention | Example |
|------|-----------|---------|
| Functions | `Verb-Noun` (approved verb) | `Get-Config`, `Set-Policy` |
| Parameters | PascalCase | `$TargetPath`, `$MaxRetries` |
| Local variables | camelCase | `$configData`, `$retryCount` |
| Constants | `UPPER_SNAKE` | `$MAX_TIMEOUT = 30` |
| Scripts | `Verb-Noun.ps1` | `Deploy-App.ps1` |

Use `Get-Verb` to validate that a verb is on the approved list.

### Error handling

```powershell
$ErrorActionPreference = 'Stop'   # at top of script

try {
    Invoke-RestMethod -Uri $url
} catch [System.Net.WebException] {
    Write-Warning "Network error: $_"
    throw
} catch {
    Write-Error "Unexpected error: $_"
    exit 1
}
```

### Output

- Use `Write-Host` for user-facing progress messages.
- Use `Write-Verbose` for debug detail (respects `-Verbose` flag).
- Use `Write-Warning` for non-fatal issues.
- Return objects from functions, not formatted strings.

### Parameters

```powershell
[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory, HelpMessage = "Path to deploy")]
    [ValidateNotNullOrEmpty()]
    [string] $Path,

    [Parameter()]
    [int] $TimeoutSeconds = 60
)
```

### Pipeline compatibility

Write functions that accept pipeline input when the operation is naturally collection-based:

```powershell
function Format-Record {
    param(
        [Parameter(ValueFromPipeline)]
        [object] $InputObject
    )
    process {
        [PSCustomObject]@{ Name = $InputObject.Name; Id = $InputObject.Id }
    }
}
```

## Constraints

- Always set `Set-StrictMode -Version Latest` and `$ErrorActionPreference = 'Stop'`.
- Never use `Write-Host` to return data — use `return` or `Write-Output`.
- Avoid `Invoke-Expression`; it is a security risk.
- Prefer `[CmdletBinding()]` on all functions to enable `-Verbose` and `-WhatIf`.
