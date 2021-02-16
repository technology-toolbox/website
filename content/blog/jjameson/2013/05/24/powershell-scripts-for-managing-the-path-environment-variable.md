---
title: "PowerShell Scripts for Managing the Path Environment Variable"
date: 2013-05-24T21:57:40-07:00
lastmod: 2013-05-24T21:57:54-07:00
excerpt: "Even though it doesn't take long to add a folder to %Path% by clicking through Windows, I prefer to do this using a little PowerShell instead."
draft: true
categories: ["Development", "Infrastructure", "My System", "SharePoint"]
tags: ["Infrastructure", "My System", "PowerShell", "SharePoint 
			2010", "Toolbox"]
---

Section 5.8 in my sample
[Installation Guide for SharePoint Server 2010 and Office Web Apps](/blog/jjameson/2013/04/30/installation-guide-for-sharepoint-server-2010-and-office-web-apps) states
the following:

> On each SharePoint server in the farm, append **C:\Program Files\Common
> Files\Microsoft Shared\web server extensions\14\BIN** to the
> **Path** environment variable (in order to run stsadm.exe from
> various folder locations without having to specify the full path).

Adding a folder to the Path environment variable is pretty basic stuff and
it probably doesn't take more than 30 seconds to click through **Control
Panel** → **System and Security** → **System** → **Advanced system settings** →
**Environment Variables** (or one of the equivalent shortcuts)
and add this to %Path%.

Nevertheless, I prefer to use a PowerShell script to complete this task in
a fraction of that time:

```
C:\NotBackedUp\Public\Toolbox\PowerShell\Add-PathFolders.ps1 "C:\Program Files\Common Files\Microsoft Shared\web server extensions\14\BIN" -EnvironmentVariableTarget "Machine"
```

In addition to the script to add folders to the Path environment variable
(Add-PathFolders.ps1), my Toolbox folder also contains a corresponding script
to get the folders in %Path% (Get-PathFolders.ps1) as well as remove folders
from %Path% (Remove-PathFolders.ps1). However, I honestly haven't found much
use for that last one. I created it primarily for the sake of completeness.

> **Note**
>
> You can download these scripts from my Toolbox repository on GitHub:
>
> [https://github.com/jeremy-jameson/Toolbox](https://github.com/jeremy-jameson/Toolbox)

### Get-PathFolders.ps1

```
<#
.SYNOPSIS
Gets the list of folders specified in the Path environment variable.

.PARAMETER EnvironmentVariableTarget
Specifies the "scope" to use when querying the Path environment variable
("Process", "Machine", or "User"). Defaults to "Process" if the parameter is
not specified.

.EXAMPLE
.\Get-PathFolders.ps1
C:\Windows\system32\WindowsPowerShell\v1.0\
C:\Windows\system32
C:\Windows
C:\Windows\System32\Wbem
...

Description
-----------
The output from this example lists each folder in the Path environment variable
for the current process.

.EXAMPLE
.\Get-PathFolders.ps1 User
C:\NotBackedUp\Public\Toolbox

Description
-----------
The output from this example assumes one folder
("C:\NotBackedUp\Public\Toolbox") has previously been added to the user's Path
environment variable.
#>
param(
    [string] $EnvironmentVariableTarget = "Process")

Set-StrictMode -Version Latest
$ErrorActionPreference = "Stop"

[string[]] $pathFolders = [Environment]::GetEnvironmentVariable(
    "Path",
    $EnvironmentVariableTarget) -Split ";"

If ($pathFolders -ne $null)
{
    Write-Output $pathFolders
}
```

### Add-PathFolders.ps1

```
<#
.SYNOPSIS
Adds one or more folders to the Path environment variable.

.PARAMETER Folders
Specifies the folders to add to the Path environment variable..

.PARAMETER EnvironmentVariableTarget
Specifies the "scope" to use for the Path environment variable ("Process",
"Machine", or "User"). Defaults to "Process" if the parameter is not specified.

.EXAMPLE
.\Add-PathFolders.ps1 C:\NotBackedUp\Public\Toolbox
#>
param(
    [parameter(Mandatory = $true, ValueFromPipeline = $true)]
    [string[]] $Folders,
    [string] $EnvironmentVariableTarget = "Process")

begin
{
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"

    Write-Verbose "Path environment variable target: $EnvironmentVariableTarget"

    [bool] $isInputFromPipeline =
        ($PSBoundParameters.ContainsKey("Folders") -eq $false)

    [int] $foldersAdded = 0

    [string[]] $pathFolders = [Environment]::GetEnvironmentVariable(
        "Path",
        $EnvironmentVariableTarget) -Split ";"

    [Collections.ArrayList] $folderList = New-Object Collections.ArrayList

    $pathFolders | foreach {
        $folderList.Add($_) | Out-Null
    }
}

process
{
    If ($isInputFromPipeline -eq $true)
    {
        $items = $_
    }
    Else
    {
        $items = $Folders
    }

    $items | foreach {
        [string] $folder = $_

        [bool] $isFolderInList = $false

        $folderList | foreach {
            If ([string]::Compare($_, $folder, $true) -eq 0)
            {
                Write-Verbose ("The folder ($folder) is already included" `
                    + " in the Path environment variable.")

                $isFolderInList = $true
                return
            }
        }

        If ($isFolderInList -eq $false)
        {
            Write-Verbose ("Adding folder ($folder) to Path environment" `
                + " variable...")

            $folderList.Add($folder) | Out-Null

            $foldersAdded++
        }
    }
}

end
{
    If ($foldersAdded -eq 0)
    {
        Write-Verbose ("No changes to the Path environment variable are" `
            + " necessary.")

        return
    }

    [string] $delimitedFolders = $folderList -Join ";"

    [Environment]::SetEnvironmentVariable(
        "Path",
        $delimitedFolders,
        $EnvironmentVariableTarget)

    Write-Verbose ("Successfully added $foldersAdded folder(s) to Path" `
        + " environment variable.")
}
```

### Remove-PathFolders.ps1

```
<#
.SYNOPSIS
Removes one or more folders from the Path environment variable.

.PARAMETER Folders
Specifies the folders to remove from the Path environment variable..

.PARAMETER EnvironmentVariableTarget
Specifies the "scope" to use for the Path environment variable ("Process",
"Machine", or "User"). Defaults to "Process" if the parameter is not specified.

.EXAMPLE
.\Remove-PathFolders.ps1 C:\NotBackedUp\Public\Toolbox
#>
param(
    [parameter(Mandatory = $true, ValueFromPipeline = $true)]
    [string[]] $Folders,
    [string] $EnvironmentVariableTarget = "Process")

begin
{
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"

    Write-Verbose "Path environment variable target: $EnvironmentVariableTarget"

    [bool] $isInputFromPipeline =
        ($PSBoundParameters.ContainsKey("Folders") -eq $false)

    [int] $foldersRemoved = 0

    [string[]] $pathFolders = [Environment]::GetEnvironmentVariable(
        "Path",
        $EnvironmentVariableTarget) -Split ";"

    [Collections.ArrayList] $folderList = New-Object Collections.ArrayList

    $pathFolders | foreach {
        $folderList.Add($_) | Out-Null
    }
}

process
{
    If ($isInputFromPipeline -eq $true)
    {
        $items = $_
    }
    Else
    {
        $items = $Folders
    }

    $items | foreach {
        [string] $folder = $_

        [bool] $isFolderInList = $false

        for ([int] $i = 0; $i -lt $folderList.Count; $i++)
        {
            If ([string]::Compare($folderList[$i], $folder, $true) -eq 0)
            {
                $isFolderInList = $true

                Write-Verbose ("Removing folder ($folder) from Path" `
                    + " environment variable...")

                $folderList.RemoveAt($i)
                $i--

                $foldersRemoved++
            }
        }

        If ($isFolderInList -eq $false)
        {
            Write-Verbose ("The folder ($folder) is not specified in the Path" `
                + " list.")

        }
    }
}

end
{
    If ($foldersRemoved -eq 0)
    {
        Write-Verbose ("No changes to the Path environment variable are" `
            + " necessary.")

        return
    }

    [string] $delimitedFolders = $folderList -Join ";"

    [Environment]::SetEnvironmentVariable(
        "Path",
        $delimitedFolders,
        $EnvironmentVariableTarget)

    Write-Verbose ("Successfully removed $foldersRemoved folder(s) from Path" `
        + " environment variable.")
}
```

