---
title: "PowerShell Scripts for Managing MaxPatchCacheSize"
date: 2013-05-06T22:25:04+08:00
excerpt: "Here's a pair of PowerShell scripts for quickly setting and verifying the MaxPatchCacheSize registry setting."
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "PowerShell", "Toolbox"]
---

Several years ago I wrote a post about[saving signficant disk space by setting MaxPatchCacheSize to 0](/blog/jjameson/archive/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0.aspx). More recently, you may have read my sample[Installation Guide for SharePoint Server 2010 and Office Web Apps](/blog/jjameson/archive/2013/04/30/installation-guide-for-sharepoint-server-2010-and-office-web-apps.aspx), in which case you noticed this is included there as well (section 4.3).

While the effectiveness of this registry setting has unfortunately diminished over the years, I still typically configure this immediately after booting up a new VM created from a SysPrep'ed image.

Even though it's easy to configure this by copying and pasting the following into a command prompt...

<kbd>reg add HKLM\Software\Policies\Microsoft\Windows\Installer /v MaxPatchCacheSize 	/t REG_DWORD /d 0 /f</kbd>

...sometime last year I decided to create a pair of PowerShell scripts to make this even easier. I keep these scripts in my[Toolbox](/blog/jjameson/archive/2007/03/22/backedup-and-notbackedup.aspx) (which is a folder I keep synchronized across the multitude of servers and workstations that I use).

The following illustrates these scripts in action:



    PS C:\NotBackedUp\Toolbox\PowerShell>.\Set-MaxPatchCacheSize.ps1 0
    Successfully set MaxPatchCacheSize to 0.
    PS C:\NotBackedUp\Toolbox\PowerShell>.\Get-MaxPatchCacheSize.ps1
    0




> **Note**
> 
> 
> You can download these scripts from my Toolbox repository on GitHub:
> 
> [https://github.com/jeremy-jameson/Toolbox](https://github.com/jeremy-jameson/Toolbox)


### Get-MaxPatchCacheSize.ps1



    <#
    .SYNOPSIS
    Gets the maximum percentage of disk space the Windows installer can use for the
    cache of old files.
    
    .DESCRIPTION
    If this per-machine system policy is set to a value greater than 0, Windows
    Installer saves older versions of files in a cache when a patch is applied to an
    application.
    
    .LINK
    http://msdn.microsoft.com/en-us/library/windows/desktop/aa369798.aspx
    
    .LINK
    https://www.technologytoolbox.com/blog/jjameson/archive/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0.aspx
    
    .EXAMPLE
    .\Get-MaxPatchCacheSize.ps1
    0
    
    Description
    -----------
    In this example, the MaxPatchCacheSize policy has previously been set to 0 (to
    indicate that no additional files should be saved).
    #>
    
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"
    
    [string] $installerPath = "HKLM:\Software\Policies\Microsoft\Windows\Installer"
    
    Get-ItemProperty -Path $installerPath -Name MaxPatchCacheSize |
        ForEach-Object { $_.MaxPatchCacheSize }



### Set-MaxPatchCacheSize.ps1



    <#
    .SYNOPSIS
    Sets the maximum percentage of disk space the Windows installer can use for the
    cache of old files.
    
    .DESCRIPTION
    If this per-machine system policy is set to a value greater than 0, Windows
    Installer saves older versions of files in a cache when a patch is applied to an
    application.
    
    .PARAMETER MaxPercentageOfDiskSpace
    The maximum percentage of disk space the installer can use for the cache of old
    files.
    
    .LINK
    http://msdn.microsoft.com/en-us/library/windows/desktop/aa369798.aspx
    
    .LINK
    https://www.technologytoolbox.com/blog/jjameson/archive/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0.aspx
    
    .EXAMPLE
    .\Set-MaxPatchCacheSize.ps1 0
    
    Description
    -----------
    Sets the value of the MaxPatchCacheSize policy to 0 to indicate that no
    additional files should be saved.
    
    .NOTES
    This script must be run with administrator privileges.
    #>
    param(
        [parameter(Mandatory=$true)]
        [int] $MaxPercentageOfDiskSpace)
    
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"
    
    [int] $maxPatchCacheSize = $MaxPercentageOfDiskSpace
    
    [string] $installerPath = "HKLM:\Software\Policies\Microsoft\Windows\Installer"
    
    $installerKey = Get-Item -Path $installerPath -EA 0
    
    If ($installerKey -eq $null)
    {
        $installerKey = New-Item -Path $installerPath
    }
    
    $currentMaxPatchCacheSize = $installerKey.GetValue("MaxPatchCacheSize")
    
    If ($currentMaxPatchCacheSize -eq $null)
    {
        New-ItemProperty -Path $installerPath -Name MaxPatchCacheSize `
            -PropertyType DWord -Value $maxPatchCacheSize | Out-Null
    }
    ElseIf ($currentMaxPatchCacheSize -eq $maxPatchCacheSize)
    {
        Write-Verbose ("MaxPatchCacheSize is already set to {0}." -f $maxPatchCacheSize)
        Exit
    }
    Else
    {
        Set-ItemProperty -Path $installerPath -Name MaxPatchCacheSize `
            -Value $maxPatchCacheSize | Out-Null
    }
    
    Write-Host -Fore Green ("Successfully set MaxPatchCacheSize to {0}." `
        -f $maxPatchCacheSize)

