# PowerShell Script Header Template

Copy this block to the top of every new script. Fill in the bracketed fields.

```powershell
#----------------------------------------------------------------------
# Script:      [Verb-Noun.ps1]
# Description: [One-sentence summary of what this script does.]
# Author:      [Author name or team]
# Created:     [YYYY-MM-DD]
#
# Parameters:
#   -[ParamName]  [Type]  [Description]
#
# Examples:
#   .\Verb-Noun.ps1 -ParamName "value"
#
# Notes:
#   [Any non-obvious constraints, dependencies, or gotchas.]
#----------------------------------------------------------------------

[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory, HelpMessage = "[Description]")]
    [ValidateNotNullOrEmpty()]
    [string] $ParamName
)

Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

#region Functions

function Get-[Something] {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string] $Input
    )

    # implementation
}

#endregion

#region Main

try {
    # Main logic here
    Write-Verbose "Starting [action]..."

} catch {
    Write-Error "Script failed: $_"
    exit 1
}

#endregion
```

## Region conventions

Use `#region` / `#endregion` to group logical sections:

| Region | Purpose |
|--------|---------|
| `#region Functions` | All helper functions |
| `#region Main` | Script entry point |
| `#region Config` | Static configuration values |
| `#region Validation` | Pre-flight checks |

## Encoding and line endings

- Save files as **UTF-8 with BOM** (`utf8bom`) for cross-platform PowerShell compatibility.
- Use **LF** line endings in repositories shared with Linux agents; Windows-only scripts may use CRLF.

```powershell
# Set encoding when writing files
$content | Set-Content -Path $path -Encoding utf8BOM
```
