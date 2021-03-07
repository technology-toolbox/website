---
title: "PowerShell Scripts for Managing BackConnectionHostNames (KB 896861)"
date: 2013-05-24T07:34:01-06:00
excerpt: "Here's a set of scripts to make it easier to view the items in the \"BackConnectionHostNames\" registry key, as well as add and remove hostnames."
aliases: ["/blog/jjameson/archive/2013/05/24/powershell-scripts-for-managing-backconnectionhostnames-kb-896861.aspx"]
draft: true
categories: ["Development", "Infrastructure", "My System", "SharePoint"]
tags: ["Infrastructure", "My System", "PowerShell", "SharePoint 2010", "Toolbox"]
---

In [my previous post](/blog/jjameson/2013/05/07/powershell-scripts-for-managing-the-hosts-file), I shared some PowerShell scripts for managing hosts files. If you're a SharePoint developer, then you know that adding a hostname to the hosts file is only part of the process you need to complete whenever creating a new Web application in development environments.

After reading my previous post, you may have immediately asked something like "What about the BackConnectionHostNames registry entry?" This, of course, assumes you don't use the "sledge hammer" approach listed in [KB 896861](http://support.microsoft.com/kb/896861) (a.k.a. "Method 2: Disable the loopback check (less-recommended method)").

Or perhaps you already noticed the Add-BackConnectionHostNames.ps1 script in my [sample solution for an extranet site based on SharePoint Server 2010](/blog/jjameson/2013/04/30/installation-guide-for-sharepoint-server-2010-and-office-web-apps).

> **Note**
>
> In the Fabrikam Extranet solution, the "Create Web Application.ps1" script checks if the environment is configured to use [http://extranet-local.fabrikam.com](http://extranet-local.fabrikam.com). If it is, the Add-BackConnectionHostNames.ps1 script is used to add **extranet-local.fabrikam.com** to the **HKLM:\System\CurrentControlSet\Control\Lsa\MSV1\_0\BackConnectionHostNames** registry key.
>
> That way, when people want to deploy the solution with as little effort as possible, they can simply run the "Rebuild Web Application.ps1" script and not have to worry about manually performing the steps in section 4.10 (Allow specific host names mapped to 127.0.0.1).

The following illustrates these scripts in action:

```
PS C:\...\Toolbox\PowerShell>{{< kbd ".\Get-BackConnectionHostNames.ps1" >}}
eln-local.dow.com
researchportal-local.dow.com
PS C:\...\Toolbox\PowerShell>{{< kbd ".\Add-BackConnectionHostNames.ps1 extranet-local.fabrikam.com" >}}
PS C:\NotBackedUp\...\Toolbox\PowerShell>{{< kbd ".\Get-BackConnectionHostNames.ps1" >}}
eln-local.dow.com
extranet-local.fabrikam.com
researchportal-local.dow.com
PS C:\...\Toolbox\PowerShell>{{< kbd ".\Remove-BackConnectionHostNames.ps1 extranet-local.fabrikam.com" >}}
PS C:\NotBackedUp\...\Toolbox\PowerShell>{{< kbd ".\Get-BackConnectionHostNames.ps1" >}}
eln-local.dow.com
researchportal-local.dow.com
```

> **Note**
>
> You can download these scripts from my Toolbox repository on GitHub:
>
> [https://github.com/jeremy-jameson/Toolbox](https://github.com/jeremy-jameson/Toolbox)

> **Update (2013-06-04)**
>
> I fixed a bug in the following scripts when more than one hostname is specified in the registry value. See my GitHub repository for details on the changes made to the original versions of the scripts.

### Get-BackConnectionHostNames.ps1

```
<#
.SYNOPSIS
Gets the list of host names specified in the BackConnectionHostNames registry
value.

.DESCRIPTION
The BackConnectionHostNames registry value is used to bypass the loopback
security check for specific host names.

.LINK
http://support.microsoft.com/kb/896861

.EXAMPLE
.\Get-BackConnectionHostNames.ps1
fabrikam-local
www-local.fabrikam.com

Description
-----------
The output from this example assumes two host names ("fabrikam-local" and
"www-local.fabrikam.com") have previously been added to the
BackConnectionHostNames registry value .
#>

Set-StrictMode -Version Latest
$ErrorActionPreference = "Stop"

[string] $registryPath = "HKLM:\System\CurrentControlSet\Control\Lsa\MSV1_0"

$registryKey = Get-Item -Path $registryPath

$backConnectionHostNames = $registryKey.GetValue("BackConnectionHostNames")

If ($backConnectionHostNames -ne $null)
{
    $backConnectionHostNames | Write-Output
}
```

### Add-BackConnectionHostNames.ps1

```
<#
.SYNOPSIS
Adds one or more host names to the BackConnectionHostNames registry value.

.DESCRIPTION
The BackConnectionHostNames registry value is used to bypass the loopback
security check for specific host names.

.LINK
http://support.microsoft.com/kb/896861

.EXAMPLE
.\Add-BackConnectionHostNames.ps1 fabrikam-local, www-local.fabrikam.com
#>
param(
    [parameter(Mandatory = $true, ValueFromPipeline = $true)]
    [string[]] $HostNames)

begin
{
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"

    function GetBackConnectionHostNameList
    {
        [Collections.ArrayList] $hostNameList = New-Object Collections.ArrayList

        [string] $registryPath =
            "HKLM:\System\CurrentControlSet\Control\Lsa\MSV1_0"

        $registryKey = Get-Item -Path $registryPath

        $registryValue = $registryKey.GetValue("BackConnectionHostNames")

        $registryValue |
            ForEach-Object {
                $hostNameList.Add($_) | Out-Null
            }

        # HACK: Return an array (containing the ArrayList) to avoid issue with
        # PowerShell returning a string (when registry value only contains one
        # item)
        return ,$hostNameList
    }

    function SetBackConnectionHostNamesRegistryValue(
        [Collections.ArrayList] $hostNameList =
            $(Throw "Value cannot be null: hostNameList"))
    {
        $hostNameList.Sort()

        for ([int] $i = 0; $i -lt $hostNameList.Count; $i++)
        {
            If ([string]::IsNullOrEmpty($hostNameList[$i]) -eq $true)
            {
                $hostNameList.RemoveAt($i)
                $i--
            }
        }

        [string] $registryPath =
            "HKLM:\System\CurrentControlSet\Control\Lsa\MSV1_0"

        $registryKey = Get-Item -Path $registryPath

        $registryValue = $registryKey.GetValue("BackConnectionHostNames")

        If ($hostNameList.Count -eq 0)
        {
            Remove-ItemProperty -Path $registryPath `
                -Name BackConnectionHostNames
        }
        ElseIf ($registryValue -eq $null)
        {
            New-ItemProperty -Path $registryPath -Name BackConnectionHostNames `
                -PropertyType MultiString -Value $hostNameList | Out-Null
        }
        Else
        {
            Set-ItemProperty -Path $registryPath -Name BackConnectionHostNames `
                -Value $hostNameList | Out-Null
        }
    }

    [bool] $isInputFromPipeline =
        ($PSBoundParameters.ContainsKey("HostNames") -eq $false)

    [int] $hostNamesAdded = 0

    [Collections.ArrayList] $hostNameList = GetBackConnectionHostNameList
}

process
{
    If ($isInputFromPipeline -eq $true)
    {
        $items = $_
    }
    Else
    {
        $items = $HostNames
    }

    $items | foreach {
        [string] $hostName = $_

        [bool] $isHostNameInList = $false

        $hostNameList | foreach {
            If ([string]::Compare($_, $hostName, $true) -eq 0)
            {
                Write-Verbose ("The host name ($hostName) is already" `
                    + " specified in the BackConnectionHostNames list.")

                $isHostNameInList = $true
                return
            }
        }

        If ($isHostNameInList -eq $false)
        {
            Write-Verbose ("Adding host name ($hostName) to" `
                + " BackConnectionHostNames list...")

            $hostNameList.Add($hostName) | Out-Null

            $hostNamesAdded++
        }
    }
}

end
{
    If ($hostNamesAdded -eq 0)
    {
        Write-Verbose ("No changes to the BackConnectionHostNames registry" `
            + " value are necessary.")

        return
    }

    SetBackConnectionHostNamesRegistryValue $hostNameList

    Write-Verbose ("Successfully added $hostNamesAdded host name(s) to" `
        + " the BackConnectionHostNames registry value.")
}
```

### Remove-BackConnectionHostNames.ps1

```
<#
.SYNOPSIS
Removes one or more host names from the BackConnectionHostNames registry value.

.DESCRIPTION
The BackConnectionHostNames registry value is used to bypass the loopback
security check for specific host names.

.LINK
http://support.microsoft.com/kb/896861

.EXAMPLE
.\Remove-BackConnectionHostNames.ps1 fabrikam-local, www-local.fabrikam.com
#>
param(
    [parameter(Mandatory = $true, ValueFromPipeline = $true)]
    [string[]] $HostNames)

begin
{
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"

    function GetBackConnectionHostNameList
    {
        [Collections.ArrayList] $hostNameList = New-Object Collections.ArrayList

        [string] $registryPath =
            "HKLM:\System\CurrentControlSet\Control\Lsa\MSV1_0"

        $registryKey = Get-Item -Path $registryPath

        $registryValue = $registryKey.GetValue("BackConnectionHostNames")

        $registryValue |
            ForEach-Object {
                $hostNameList.Add($_) | Out-Null
            }

        # HACK: Return an array (containing the ArrayList) to avoid issue with
        # PowerShell returning a string (when registry value only contains one
        # item)
        return ,$hostNameList
    }

    function SetBackConnectionHostNamesRegistryValue(
        [Collections.ArrayList] $hostNameList =
            $(Throw "Value cannot be null: hostNameList"))
    {
        $hostNameList.Sort()

        for ([int] $i = 0; $i -lt $hostNameList.Count; $i++)
        {
            If ([string]::IsNullOrEmpty($hostNameList[$i]) -eq $true)
            {
                $hostNameList.RemoveAt($i)
                $i--
            }
        }

        [string] $registryPath =
            "HKLM:\System\CurrentControlSet\Control\Lsa\MSV1_0"

        $registryKey = Get-Item -Path $registryPath

        $registryValue = $registryKey.GetValue("BackConnectionHostNames")

        If ($hostNameList.Count -eq 0)
        {
            Remove-ItemProperty -Path $registryPath `
                -Name BackConnectionHostNames
        }
        ElseIf ($registryValue -eq $null)
        {
            New-ItemProperty -Path $registryPath -Name BackConnectionHostNames `
                -PropertyType MultiString -Value $hostNameList | Out-Null
        }
        Else
        {
            Set-ItemProperty -Path $registryPath -Name BackConnectionHostNames `
                -Value $hostNameList | Out-Null
        }
    }

    [bool] $isInputFromPipeline =
        ($PSBoundParameters.ContainsKey("HostNames") -eq $false)

    [int] $hostNamesRemoved = 0

    [Collections.ArrayList] $hostNameList = GetBackConnectionHostNameList
}

process
{
    If ($isInputFromPipeline -eq $true)
    {
        $items = $_
    }
    Else
    {
        $items = $HostNames
    }

    $items | foreach {
        [string] $hostName = $_

        [bool] $isHostNameInList = $false

        for ([int] $i = 0; $i -lt $hostNameList.Count; $i++)
        {
            If ([string]::Compare($hostNameList[$i], $hostName, $true) -eq 0)
            {
                Write-Verbose ("Removing host name ($hostName) from" `
                    + " BackConnectionHostNames list...")

                $hostNameList.RemoveAt($i)
                $i--

                $hostNamesRemoved++

                $isHostNameInList = $true
            }
        }

        If ($isHostNameInList -eq $false)
        {
            Write-Verbose ("The host name ($hostName) is not" `
                + " specified in the BackConnectionHostNames list.")

        }
    }
}

end
{
    If ($hostNamesRemoved -eq 0)
    {
        Write-Verbose ("No changes to the BackConnectionHostNames registry" `
            + " value are necessary.")

        return
    }

    SetBackConnectionHostNamesRegistryValue $hostNameList

    Write-Verbose ("Successfully removed $hostNamesRemoved host name(s)" `
        + " from the BackConnectionHostNames registry value.")
}
```

