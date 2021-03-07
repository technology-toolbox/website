---
title: "PowerShell Scripts for Managing the Hosts File"
date: 2013-05-07T23:01:18-06:00
excerpt: "Here's a set of scripts to make it easier to view the hostname mappings in the hosts file, as well as add and remove hostnames."
aliases: ["/blog/jjameson/archive/2013/05/07/powershell-scripts-for-managing-the-hosts-file.aspx"]
draft: true
categories: ["Infrastructure", "My System", "SharePoint", "Development"]
tags: ["Infrastructure", "My System", "PowerShell", "SharePoint 2010", "Toolbox"]
---

Around the same time I created my
[PowerShell scripts for managing the MaxPatchCacheSize registry setting](/blog/jjameson/2013/05/06/powershell-scripts-for-managing-maxpatchcachesize),
I also created a set of scripts to automate the process of managing hosts files.

You may have noticed the Add-Hostnames.ps1 script in my
[sample solution for an extranet site based on SharePoint Server 2010](/blog/jjameson/2013/04/30/installation-guide-for-sharepoint-server-2010-and-office-web-apps).

> **Note**
>
> In the Fabrikam Extranet solution, the "Create Web Application.ps1" script checks if the environment is configured to use [http://extranet-local.fabrikam.com](http://extranet-local.fabrikam.com) and, if it is, it then checks for name resolution for that hostname. If name resolution fails, the Add-Hostnames.ps1 script is used to map **extranet-local.fabrikam.com** to the loopback address (127.0.0.1). That way, when people want to deploy the solution with as little effort as possible, they can simply run the "Rebuild Web Application.ps1" script and not have to worry about manually performing the steps in section 4.9 (DEV – Map Web application to loopback address in Hosts file).

The following illustrates these scripts in action:

```
PS C:\NotBackedUp\Toolbox\PowerShell>{{< kbd ".\Get-Hostnames.ps1 | sort Hostname" >}}

Hostname                                                    IpAddress
--------                                                    ---------
eln-local.dow.com                                           127.0.0.1
extranet-local.fabrikam.com                                 127.0.0.1
fabrikam-local                                              127.0.0.1
researchportal-local.dow.com                                127.0.0.1
tugboatcoffee-local                                         127.0.0.1
www-local.fabrikam.com                                      127.0.0.1
www-local.technologytoolbox.com                             127.0.0.1
www-local.tugboatcoffee.com                                 127.0.0.1

PS C:\NotBackedUp\Toolbox\PowerShell>{{< kbd ".\Add-Hostnames.ps1 192.168.10.119 foobar" >}}
PS C:\NotBackedUp\Toolbox\PowerShell>{{< kbd ".\Get-Hostnames.ps1 | sort Hostname" >}}

Hostname                                                    IpAddress
--------                                                    ---------
eln-local.dow.com                                           127.0.0.1
extranet-local.fabrikam.com                                 127.0.0.1
fabrikam-local                                              127.0.0.1
foobar                                                      192.168.10.119
researchportal-local.dow.com                                127.0.0.1
tugboatcoffee-local                                         127.0.0.1
www-local.fabrikam.com                                      127.0.0.1
www-local.technologytoolbox.com                             127.0.0.1
www-local.tugboatcoffee.com                                 127.0.0.1

PS C:\NotBackedUp\Toolbox\PowerShell>{{< kbd ".\Remove-Hostnames.ps1 foobar" >}}
PS C:\NotBackedUp\Toolbox\PowerShell>{{< kbd ".\Get-Hostnames.ps1 | sort Hostname" >}}

Hostname                                                    IpAddress
--------                                                    ---------
eln-local.dow.com                                           127.0.0.1
extranet-local.fabrikam.com                                 127.0.0.1
fabrikam-local                                              127.0.0.1
researchportal-local.dow.com                                127.0.0.1
tugboatcoffee-local                                         127.0.0.1
www-local.fabrikam.com                                      127.0.0.1
www-local.technologytoolbox.com                             127.0.0.1
www-local.tugboatcoffee.com                                 127.0.0.1

```

> **Note**
>
> You can download these scripts from my Toolbox repository on GitHub:
>
> [https://github.com/jeremy-jameson/Toolbox](https://github.com/jeremy-jameson/Toolbox)

### Get-Hostnames.ps1

```
<#
.SYNOPSIS
Gets the hostnames (and corresponding IP addresses) specified in the hosts file.

.DESCRIPTION
The hosts file is used to map hostnames to IP addresses.

.EXAMPLE
.\Get-HostNames.ps1

Hostname                                                    IpAddress
--------                                                    ---------
localhost                                                   127.0.0.1
fabrikam-local                                              127.0.0.1
www-local.fabrikam.com                                      127.0.0.1
#>

begin
{
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"

    function CreateHostsEntryObject(
        [string] $ipAddress,
        [string[]] $hostnames,
        <# [string] #> $comment) #HACK: never $null if type is specified
    {
        $hostsEntry = New-Object PSObject
        $hostsEntry | Add-Member NoteProperty -Name "IpAddress" `
            -Value $ipAddress

        [System.Collections.ArrayList] $hostnamesList =
            New-Object System.Collections.ArrayList

        $hostsEntry | Add-Member NoteProperty -Name "Hostnames" `
            -Value $hostnamesList

        If ($hostnames -ne $null)
        {
            $hostnames | foreach {
                $hostsEntry.Hostnames.Add($_) | Out-Null
            }
        }

        $hostsEntry | Add-Member NoteProperty -Name "Comment" -Value $comment

        return $hostsEntry
    }

    function ParseHostsEntry(
        [string] $line)
    {
        $hostsEntry = CreateHostsEntryObject

        Write-Debug "Parsing hosts entry: $line"

        If ($line.Contains("#") -eq $true)
        {
            If ($line -eq "#")
            {
                $hostsEntry.Comment = [string]::Empty
            }
            Else
            {
                $hostsEntry.Comment = $line.Substring($line.IndexOf("#") + 1)
            }

            $line = $line.Substring(0, $line.IndexOf("#"))
        }

        $line = $line.Trim()

        If ($line.Length -gt 0)
        {
            $hostsEntry.IpAddress = ($line -Split "\s+")[0]

            Write-Debug "Parsed address: $($hostsEntry.IpAddress)"

            [string[]] $parsedHostnames = $line.Substring(
                $hostsEntry.IpAddress.Length + 1).Trim() -Split "\s+"

            Write-Debug ("Parsed hostnames ($($parsedHostnames.Length)):" `
                + " $parsedHostnames")

            $parsedHostnames | foreach {
                $hostsEntry.Hostnames.Add($_) | Out-Null
            }
        }

        return $hostsEntry
    }

    function ParseHostsFile
    {
        $hostsEntries = New-Object System.Collections.ArrayList

        [string] $hostsFile = $env:WINDIR + "\System32\drivers\etc\hosts"

        If ((Test-Path $hostsFile) -eq $false)
        {
            Write-Verbose "Hosts file does not exist."
        }
        Else
        {
            [string[]] $hostsContent = Get-Content $hostsFile

            $hostsContent | foreach {
                $hostsEntry = ParseHostsEntry $_

                $hostsEntries.Add($hostsEntry) | Out-Null
            }
        }

        # HACK: Return an array (containing the ArrayList) to avoid issue with
        # PowerShell returning $null (when hosts file does not exist)
        return ,$hostsEntries
    }

    [Collections.ArrayList] $hostsEntries = ParseHostsFile
}

process
{
    $hostsEntries | foreach {
        $hostsEntry = $_

        $hostsEntry.Hostnames | foreach {
            $properties = @{
                Hostname = $_
                IpAddress = $hostsEntry.IpAddress
            }

            New-Object PSObject -Property $properties
        }
    }
}
```

### Add-Hostnames.ps1

```
<#
.SYNOPSIS
Adds one or more hostnames to the hosts file.

.DESCRIPTION
The hosts file is used to map hostnames to IP addresses.

.PARAMETER IPAddress
The IP address to map the hostname(s) to.

.PARAMETER Hostnames
One or more hostnames to map to the specified IP address.

.PARAMETER Comment
Optional comment that is written above the new hosts entry.

.EXAMPLE
.\Add-Hostnames.ps1 127.0.0.1 foobar

Description
-----------
Adds the following line to the hosts file (assuming "foobar" does not already
exist in the hosts file):

127.0.0.1    foobar

A warning is displayed if "foobar" already exists in the hosts file and is
mapped to the specified IP address. An error occurs if "foobar" is already
mapped to a different IP address.

.EXAMPLE
.\Add-Hostnames.ps1 127.0.0.1 foo, bar "This is a comment"

Description
-----------
Adds the following lines to the hosts file (assuming "foo" and "bar" do not
already exist in the hosts file):

# This is a comment
127.0.0.1    foo bar

A warning is displayed if either "foo" or "bar" already exists in the hosts
file and is mapped to the specified IP address. An error occurs if "foo" or
"bar" is already mapped to a different IP address.

.NOTES
This script must be run with administrator privileges.
#>
param(
    [parameter(Mandatory = $true)]
    [string] $IPAddress,
    [parameter(Mandatory = $true, ValueFromPipeline = $true)]
    [string[]] $Hostnames,
    [string] $Comment
)

begin
{
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"

    function CreateHostsEntryObject(
        [string] $ipAddress,
        [string[]] $hostnames,
        <# [string] #> $comment) #HACK: never $null if type is specified
    {
        $hostsEntry = New-Object PSObject
        $hostsEntry | Add-Member NoteProperty -Name "IpAddress" `
            -Value $ipAddress

        [System.Collections.ArrayList] $hostnamesList =
            New-Object System.Collections.ArrayList

        $hostsEntry | Add-Member NoteProperty -Name "Hostnames" `
            -Value $hostnamesList

        If ($hostnames -ne $null)
        {
            $hostnames | foreach {
                $hostsEntry.Hostnames.Add($_) | Out-Null
            }
        }

        $hostsEntry | Add-Member NoteProperty -Name "Comment" -Value $comment

        return $hostsEntry
    }

    function ParseHostsEntry(
        [string] $line)
    {
        $hostsEntry = CreateHostsEntryObject

        Write-Debug "Parsing hosts entry: $line"

        If ($line.Contains("#") -eq $true)
        {
            If ($line -eq "#")
            {
                $hostsEntry.Comment = [string]::Empty
            }
            Else
            {
                $hostsEntry.Comment = $line.Substring($line.IndexOf("#") + 1)
            }

            $line = $line.Substring(0, $line.IndexOf("#"))
        }

        $line = $line.Trim()

        If ($line.Length -gt 0)
        {
            $hostsEntry.IpAddress = ($line -Split "\s+")[0]

            Write-Debug "Parsed address: $($hostsEntry.IpAddress)"

            [string[]] $parsedHostnames = $line.Substring(
                $hostsEntry.IpAddress.Length + 1).Trim() -Split "\s+"

            Write-Debug ("Parsed hostnames ($($parsedHostnames.Length)):" `
                + " $parsedHostnames")

            $parsedHostnames | foreach {
                $hostsEntry.Hostnames.Add($_) | Out-Null
            }
        }

        return $hostsEntry
    }

    function ParseHostsFile
    {
        $hostsEntries = New-Object System.Collections.ArrayList

        [string] $hostsFile = $env:WINDIR + "\System32\drivers\etc\hosts"

        If ((Test-Path $hostsFile) -eq $false)
        {
            Write-Verbose "Hosts file does not exist."
        }
        Else
        {
            [string[]] $hostsContent = Get-Content $hostsFile

            $hostsContent | foreach {
                $hostsEntry = ParseHostsEntry $_

                $hostsEntries.Add($hostsEntry) | Out-Null
            }
        }

        # HACK: Return an array (containing the ArrayList) to avoid issue with
        # PowerShell returning $null (when hosts file does not exist)
        return ,$hostsEntries
    }

    function UpdateHostsFile(
        $hostsEntries = $(Throw "Value cannot be null: hostsEntries"))
    {
        Write-Verbose "Updatings hosts file..."

        [string] $hostsFile = $env:WINDIR + "\System32\drivers\etc\hosts"

        $buffer = New-Object System.Text.StringBuilder

        $hostsEntries | foreach {

            If ([string]::IsNullOrEmpty($_.IpAddress) -eq $false)
            {
                $buffer.Append($_.IpAddress) | Out-Null
                $buffer.Append("`t") | Out-Null
            }

            If ($_.Hostnames -ne $null)
            {
                [bool] $firstHostname = $true

                $_.Hostnames | foreach {
                    If ($firstHostname -eq $false)
                    {
                        $buffer.Append(" ") | Out-Null
                    }
                    Else
                    {
                        $firstHostname = $false
                    }

                    $buffer.Append($_) | Out-Null
                }
            }

            If ($_.Comment -ne $null)
            {
                If ([string]::IsNullOrEmpty($_.IpAddress) -eq $false)
                {
                    $buffer.Append(" ") | Out-Null
                }

                $buffer.Append("#") | Out-Null
                $buffer.Append($_.Comment) | Out-Null
            }

            $buffer.Append([System.Environment]::NewLine) | Out-Null
        }

        [string] $hostsContent = $buffer.ToString()

        $hostsContent = $hostsContent.Trim()

        Set-Content -Path $hostsFile -Value $hostsContent -Force -Encoding ASCII

        Write-Verbose "Successfully updated hosts file."
    }

    [bool] $isInputFromPipeline =
        ($PSBoundParameters.ContainsKey("Hostnames") -eq $false)

    [int] $pendingUpdates = 0

    [Collections.ArrayList] $hostsEntries = ParseHostsFile
}

process
{
    If ($isInputFromPipeline -eq $true)
    {
        $items = $_
    }
    Else
    {
        $items = $Hostnames
    }

    $newHostsEntry = CreateHostsEntryObject $IpAddress
    $hostsEntries.Add($newHostsEntry) | Out-Null

    $items | foreach {
        [string] $hostname = $_

        [bool] $isHostnameInHostsEntries = $false

        for ([int] $i = 0; $i -lt $hostsEntries.Count; $i++)
        {
            $hostsEntry = $hostsEntries[$i]

            Write-Debug "Hosts entry: $hostsEntry"

            If ($hostsEntry.Hostnames.Count -eq 0)
            {
                continue
            }

            for ([int] $j = 0; $j -lt $hostsEntry.Hostnames.Count; $j++)
            {
                [string] $parsedHostname = $hostsEntry.Hostnames[$j]

                Write-Debug ("Comparing specified hostname" `
                    + " ($hostname) to existing hostname" `
                    + " ($parsedHostname)...")

                If ([string]::Compare($hostname, $parsedHostname, $true) -eq 0)
                {
                    $isHostnameInHostsEntries = $true

                    If ($ipAddress -ne $hostsEntry.IpAddress)
                    {
                        Throw "The hosts file already contains the" `
                            + " specified hostname ($parsedHostname) and it is" `
                            + " mapped to a different address" `
                            + " ($($hostsEntry.IpAddress))."
                    }

                    Write-Verbose ("The hosts file already contains the" `
                        + " specified hostname ($($hostsEntry.IpAddress) $parsedHostname).")
                }
            }
        }

        If ($isHostnameInHostsEntries -eq $false)
        {
            Write-Debug ("Adding hostname ($hostname) to hosts entry...")

            $newHostsEntry.Hostnames.Add($hostname) | Out-Null
            $pendingUpdates++
        }
    }
}

end
{
    If ($pendingUpdates -eq 0)
    {
        Write-Verbose "No changes to the hosts file are necessary."

        return
    }

    Write-Verbose ("There are $pendingUpdates pending update(s) to the hosts" `
        + " file.")

    UpdateHostsFile $hostsEntries
}
```

### Remove-Hostnames.ps1

```
<#
.SYNOPSIS
Removes one or more hostnames from the hosts file.

.DESCRIPTION
The hosts file is used to map hostnames to IP addresses.

.PARAMETER Hostnames
One or more hostnames to remove from the hosts file.

.EXAMPLE
.\Remove-Hostnames.ps1 foobar

Description
-----------
Assume the following line was previously added to the hosts file:

127.0.0.1    foobar

After running "Remove-Hostnames.ps1 foobar" the hosts file no longer contains this
line.

.EXAMPLE
.\Remove-Hostnames.ps1 foo

Description
-----------
Assume the following line was previously added to the hosts file:

127.0.0.1    foobar foo bar

After running "Remove-Hostnames.ps1 foo" the line in the hosts file is updated
to remove the specified hostname ("foo"):

127.0.0.1    foobar bar

.EXAMPLE
.\Remove-Hostnames.ps1 foo, bar

Description
-----------
Assume the following line was previously added to the hosts file:

127.0.0.1    foobar foo bar

After running "Remove-Hostnames.ps1 foo, bar" the line in the hosts file is updated to
remove the specified hostnames ("foo" and "bar"):

127.0.0.1    foobar

.NOTES
This script must be run with administrator privileges.
#>
param(
    [parameter(Mandatory = $true, ValueFromPipeline = $true)]
    [string[]] $Hostnames
)

begin
{
    Set-StrictMode -Version Latest
    $ErrorActionPreference = "Stop"

    function CreateHostsEntryObject(
        [string] $ipAddress,
        [string[]] $hostnames,
        <# [string] #> $comment) #HACK: never $null if type is specified
    {
        $hostsEntry = New-Object PSObject
        $hostsEntry | Add-Member NoteProperty -Name "IpAddress" `
            -Value $ipAddress

        [System.Collections.ArrayList] $hostnamesList =
            New-Object System.Collections.ArrayList

        $hostsEntry | Add-Member NoteProperty -Name "Hostnames" `
            -Value $hostnamesList

        If ($hostnames -ne $null)
        {
            $hostnames | foreach {
                $hostsEntry.Hostnames.Add($_) | Out-Null
            }
        }

        $hostsEntry | Add-Member NoteProperty -Name "Comment" -Value $comment

        return $hostsEntry
    }

    function ParseHostsEntry(
        [string] $line)
    {
        $hostsEntry = CreateHostsEntryObject

        Write-Debug "Parsing hosts entry: $line"

        If ($line.Contains("#") -eq $true)
        {
            If ($line -eq "#")
            {
                $hostsEntry.Comment = [string]::Empty
            }
            Else
            {
                $hostsEntry.Comment = $line.Substring($line.IndexOf("#") + 1)
            }

            $line = $line.Substring(0, $line.IndexOf("#"))
        }

        $line = $line.Trim()

        If ($line.Length -gt 0)
        {
            $hostsEntry.IpAddress = ($line -Split "\s+")[0]

            Write-Debug "Parsed address: $($hostsEntry.IpAddress)"

            [string[]] $parsedHostnames = $line.Substring(
                $hostsEntry.IpAddress.Length + 1).Trim() -Split "\s+"

            Write-Debug ("Parsed hostnames ($($parsedHostnames.Length)):" `
                + " $parsedHostnames")

            $parsedHostnames | foreach {
                $hostsEntry.Hostnames.Add($_) | Out-Null
            }
        }

        return $hostsEntry
    }

    function ParseHostsFile
    {
        $hostsEntries = New-Object System.Collections.ArrayList

        [string] $hostsFile = $env:WINDIR + "\System32\drivers\etc\hosts"

        If ((Test-Path $hostsFile) -eq $false)
        {
            Write-Verbose "Hosts file does not exist."
        }
        Else
        {
            [string[]] $hostsContent = Get-Content $hostsFile

            $hostsContent | foreach {
                $hostsEntry = ParseHostsEntry $_

                $hostsEntries.Add($hostsEntry) | Out-Null
            }
        }

        # HACK: Return an array (containing the ArrayList) to avoid issue with
        # PowerShell returning $null (when hosts file does not exist)
        return ,$hostsEntries
    }

    function UpdateHostsFile(
        $hostsEntries = $(Throw "Value cannot be null: hostsEntries"))
    {
        Write-Verbose "Updatings hosts file..."

        [string] $hostsFile = $env:WINDIR + "\System32\drivers\etc\hosts"

        $buffer = New-Object System.Text.StringBuilder

        $hostsEntries | foreach {

            If ([string]::IsNullOrEmpty($_.IpAddress) -eq $false)
            {
                $buffer.Append($_.IpAddress) | Out-Null
                $buffer.Append("`t") | Out-Null
            }

            If ($_.Hostnames -ne $null)
            {
                [bool] $firstHostname = $true

                $_.Hostnames | foreach {
                    If ($firstHostname -eq $false)
                    {
                        $buffer.Append(" ") | Out-Null
                    }
                    Else
                    {
                        $firstHostname = $false
                    }

                    $buffer.Append($_) | Out-Null
                }
            }

            If ($_.Comment -ne $null)
            {
                If ([string]::IsNullOrEmpty($_.IpAddress) -eq $false)
                {
                    $buffer.Append(" ") | Out-Null
                }

                $buffer.Append("#") | Out-Null
                $buffer.Append($_.Comment) | Out-Null
            }

            $buffer.Append([System.Environment]::NewLine) | Out-Null
        }

        [string] $hostsContent = $buffer.ToString()

        $hostsContent = $hostsContent.Trim()

        Set-Content -Path $hostsFile -Value $hostsContent -Force -Encoding ASCII

        Write-Verbose "Successfully updated hosts file."
    }

    [bool] $isInputFromPipeline =
        ($PSBoundParameters.ContainsKey("Hostnames") -eq $false)

    [int] $pendingUpdates = 0

    [Collections.ArrayList] $hostsEntries = ParseHostsFile
}

process
{
    If ($isInputFromPipeline -eq $true)
    {
        $items = $_
    }
    Else
    {
        $items = $Hostnames
    }

    $items | foreach {
        [string] $hostname = $_

        for ([int] $i = 0; $i -lt $hostsEntries.Count; $i++)
        {
            $hostsEntry = $hostsEntries[$i]

            Write-Debug "Hosts entry: $hostsEntry"

            If ($hostsEntry.Hostnames.Count -eq 0)
            {
                continue
            }

            for ([int] $j = 0; $j -lt $hostsEntry.Hostnames.Count; $j++)
            {
                [string] $parsedHostname = $hostsEntry.Hostnames[$j]

                Write-Debug ("Comparing specified hostname" `
                    + " ($hostname) to existing hostname" `
                    + " ($parsedHostname)...")

                If ([string]::Compare($hostname, $parsedHostname, $true) -eq 0)
                {
                    Write-Debug "Removing hostname ($hostname) from host entry ($hostsEntry)..."

                    $hostsEntry.Hostnames.RemoveAt($j)
                    $j--

                    $pendingUpdates++
                }
            }

            If ($hostsEntry.Hostnames.Count -eq 0)
            {
                Write-Debug ("Removing host entry (because it no longer specifies" `
                    + " any hostnames)...")

                $hostsEntries.RemoveAt($i)
                $i--
            }
        }
    }
}

end
{
    If ($pendingUpdates -eq 0)
    {
        Write-Verbose "No changes to the hosts file are necessary."

        return
    }

    Write-Verbose ("There are $pendingUpdates pending update(s) to the hosts" `
        + " file.")

    UpdateHostsFile $hostsEntries
}
```

