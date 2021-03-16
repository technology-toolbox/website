---
title: "Zip a folder using PowerShell"
date: 2012-02-27T23:50:24-07:00
excerpt: "There are a couple of options for creating a zip file for a specific folder. You can either use a third-party solution (like the PowerShell Community Extensions) or do it all with \"out-of-the-box\" functionality."
aliases: ["/blog/jjameson/archive/2012/02/27/zip-a-folder-using-powershell.aspx"]
draft: true
categories: ["Development", "Infrastructure"]
tags: ["PowerShell"]
---

After creating the code sample for my previous post, I realized the original zip
file I provided contained quite a bit of "junk" (e.g. temporary object folders
created during the build, a copy of one of the SharePoint assemblies in the
"bin" folder, etc.). I've since updated the attachment on that post to reduce
the size from 850 KB to a mere 134 KB.

However, there's a strong possibility this will happen again in the future with
some other post, if I don't take the time to either document the process of
"trimming the fat" before creating a zip file for a code sample or automate the
process via PowerShell.

Since I estimated the script would take only slightly longer to create than the
documentation -- and also save considerable time over the long run -- I decided
to go with the PowerShell option.

I already assembled some PowerShell for a similar scenario last year (in order
to quickly transfer the SharePoint solution that I was working on between
environments). From the research I did back then, I recall there being a couple
of approaches to creating a zip file in PowerShell. One is to use the
[PowerShell Community Extensions](http://pscx.codeplex.com/) (or some other
third-party solution), and the other is to combine out-of-the-box PowerShell
with some scriptable COM objects from the Windows Shell (specifically the
[**Shell**](http://msdn.microsoft.com/en-us/library/windows/desktop/bb774094.aspx)
and
[**Folder**](http://msdn.microsoft.com/en-us/library/windows/desktop/bb787868.aspx)
objects).

Imagine you have a folder (e.g. C:\NotBackedUp\Fabrikam) that you want to
compress into a zip file (e.g. C:\NotBackedUp\Fabrikam.zip).

Assuming you have installed the PowerShell Community Extensions, you could
simply execute the following in Windows PowerShell:

```
PS C:\Users\jjameson> {{< kbd "Import-Module Pscx" >}}
PS C:\Users\jjameson> {{< kbd "cd C:\NotBackedUp" >}}
{{< sample-output "C:\NotBackedUp" >}}
PS C:\NotBackedUp> {{< kbd "Write-Zip Fabrikam -OutputPath Fabrikam.zip -IncludeEmptyDirectories" >}}

Mode           LastWriteTime       Length Name
----           -------------       ------ ----
-a---     2/28/2012  5:00 AM      7698443 Fabrikam.zip
```

However, what if you don't have the PowerShell Community Extensions installed
(and, for whatever reason, you can't or don't want to install them)? In that
case, it takes a little more work.

If you Google "PowerShell zip files" you'll quickly discover a number of
resources that show how to create a zip file using the **Set-Content** cmdlet,
followed by the use of the
**[CopyHere](http://msdn.microsoft.com/en-us/library/windows/desktop/ms723207.aspx)**
method on the **Folder** shell object. For example,
[David Aiken's blog post](http://blogs.msdn.com/b/daiken/archive/2007/02/12/compress-files-with-windows-powershell-then-package-a-windows-vista-sidebar-gadget.aspx)
shows the following:

```
function Add-Zip
{
	param([string]$zipfilename)

	if(-not (test-path($zipfilename)))
	{
		set-content $zipfilename ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18))
		(dir $zipfilename).IsReadOnly = $false
	}

	$shellApplication = new-object -com shell.application
	$zipPackage = $shellApplication.NameSpace($zipfilename)

	foreach($file in $input)
	{
            $zipPackage.CopyHere($file.FullName)
            Start-sleep -milliseconds 500
	}
}
```

However, there are a few problems with this approach:

- If you use the function as David illustrates in his post (e.g. "`dir Fabrikam\*.* -Recurse | Add-Zip Fabrikam.zip`") then the folder hierarchy is
  not preserved within the zip file -- which is almost certainly not what you
  want (but seems to have been okay for David's scenario).

- If you try to operate on the folder instead (e.g. "`dir Fabrikam | Add-Zip Fabrikam.zip`") then an error occurs:
  {{< blockquote "font-italic text-danger" >}}
  
  You cannot call a method on a null-valued expression.\
  At line:13 char:33\
  
  + $zipPackage.CopyHere &lt;&lt;&lt;&lt; ($file.FullName)\
    ...
  
  {{< /blockquote >}}

- Relying exclusively on a 500 ms delay (to wait for the asynchronous
  **CopyHere** operation to complete) seems a little "dicey" to me. In other
  words, how do you know the zip operation completed successfully?

A different approach is to place the call to **Start-Sleep** inside a loop that
checks the number of items in the zip file against the expected number (as shown
in
[another blog post](http://mysticdotnet.blogspot.com/2010/04/compression-with-powershell.html)).
This is the approach I used last year and it seemed to work just fine -- most of
the time.

As I mentioned before, the **CopyHere** method runs asynchronously. When the
**CopyHere** operation is running, a dialog is displayed with a **Cancel**
button -- and if you click this by mistake (or press {{< kbd "Enter" >}} when
the dialog box has the focus) then, well, let's just say that you aren't on the
"Happy Path" anymore.

To make this process more robust, I decided to use a different approach --
specifically, counting all of the files and folders in the zip file and
comparing it to the expected number. The approach shown in the
[other blog post](http://mysticdotnet.blogspot.com/2010/04/compression-with-powershell.html)
I referred to before only counts the items in the "root" of the zip file (which,
honestly, does seem to work reliably -- even when you click the **Cancel**
button during the **CopyHere** operation). However, I wanted a higher degree of
confidence that wasn't based on the assumption that cancelling the **CopyHere**
operation is treated as a "transaction."

First, we need a function to create a zip file for a specific folder (a.k.a.
directory):

```
function ZipFolder(
    [IO.DirectoryInfo] $directory)
{
    ...
    [IO.DirectoryInfo] $parentDir = $directory.Parent

    [string] $zipFileName

    If ($parentDir.FullName.EndsWith("\") -eq $true)
    {
        # e.g. $parentDir = "C:\"
        $zipFileName = $parentDir.FullName + $directory.Name + ".zip"
    }
    Else
    {
        $zipFileName = $parentDir.FullName + "\" + $directory.Name + ".zip"
    }

    ...

    Set-Content $zipFileName ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18))

    $shellApp = New-Object -ComObject Shell.Application
    $zipFile = $shellApp.NameSpace($zipFileName)

    ...

    [int] $expectedCount = (Get-ChildItem $directory -Force -Recurse).Count
    $expectedCount += 1 # account for the top-level folder

    $zipFile.CopyHere($directory.FullName)

    # wait for CopyHere operation to complete
    WaitForZipOperationToFinish $zipFile $expectedCount

    ...}
```

The `WaitForZipOperationToFinish` function is where the "magic" happens:

```
function WaitForZipOperationToFinish(
    [__ComObject] $zipFile,
    [int] $expectedNumberOfItemsInZipFile)
{
    ...

    Write-Host -NoNewLine "Waiting for zip operation to finish..."
    Start-Sleep -Milliseconds 100 # ensure zip operation had time to start

    [int] $waitTime = 0
    [int] $maxWaitTime = 60 * 1000 # [milliseconds]
    while($waitTime -lt $maxWaitTime)
    {
        [int] $waitInterval = GetWaitInterval($waitTime)

        Write-Host -NoNewLine "."
        Start-Sleep -Milliseconds $waitInterval
        $waitTime += $waitInterval

        ...
        [bool] $isFileLocked = IsFileLocked($zipFile.Self.Path)

        If ($isFileLocked -eq $true)
        {
            Write-Debug "Zip file is locked by another process."
            Continue
        }
        Else
        {
            Break
        }
    }

    Write-Host

    If ($waitTime -ge $maxWaitTime)
    {
        Throw "Timeout exceeded waiting for zip operation"
    }

    [int] $count = CountZipItems($zipFile)

    If ($count -eq $expectedNumberOfItemsInZipFile)
    {
        Write-Debug "The zip operation completed succesfully."
    }
    ElseIf ($count -eq 0)
    {
        Throw ("Zip file is empty. This can occur if the operation is" `
            + " cancelled by the user.")
    }
    ElseIf ($count -gt $expectedCount)
    {
        Throw "Zip file contains more than the expected number of items."
    }
}
```

I use a variable "wait interval" to account for scenarios ranging from very
small folders to relatively large folders (but still assuming the zip operation
should complete in less than 60 seconds):

```
function GetWaitInterval(
    [int] $waitTime)
{
    If ($waitTime -lt 1000)
    {
        return 100
    }
    ElseIf ($waitTime -lt 5000)
    {
        return 1000
    }
    Else
    {
        return 5000
    }
}
```

To determine if the **CopyHere** operation is running, I check to see if the zip
file can be locked exclusively:

```
function IsFileLocked(
    [string] $path)
{
    ...

    [bool] $isFileLocked = $true

    $file = $null

    Try
    {
        $file = [IO.File]::Open(
            $path,
            [IO.FileMode]::Open,
            [IO.FileAccess]::Read,
            [IO.FileShare]::None)

        $isFileLocked = $false
    }
    Catch [IO.IOException]
    {
        If ($_.Exception.Message.EndsWith(
            "it is being used by another process.") -eq $false)
        {
            Throw $_.Exception
        }
    }
    Finally
    {
        If ($file -ne $null)
        {
            $file.Close()
        }
    }

    return $isFileLocked
}
```

Once the zip file is no longer locked by the zip operation, it is time to count
the total number of files and folders in a zip file:

```
function CountZipItems(
    [__ComObject] $zipFile)
{
    ...
    [int] $count = CountZipItemsRecursive($zipFile)
    ...
    return $count
}

function CountZipItemsRecursive(
    [__ComObject] $parent)
{
    ...
    [int] $count = 0

    $parent.Items() |
        ForEach-Object {
            $count += 1

            If ($_.IsFolder -eq $true)
            {
                $count += CountZipItemsRecursive($_.GetFolder)
            }
        }

    return $count
}
```

The final step is to use these functions to create the zip file:

```
PS C:\NotBackedUp> {{< kbd "$directory = Get-Item \"C:\NotBackedUp\Fabrikam\"" >}}
PS C:\NotBackedUp> {{< kbd "ZipFolder $directory" >}}
{{< sample-output "Creating zip file for folder (C:\NotBackedUp\Fabrikam)..." >}}

{{< sample-output "Waiting for zip operation to finish.............." >}}
{{< sample-output "Counting items in zip file (C:\NotBackedUp\Fabrikam.zip)..." >}}
{{< sample-output "840 items in zip file (C:\NotBackedUp\Fabrikam.zip)." >}}
{{< sample-output "Successfully created zip file for folder (C:\NotBackedUp\Fabrikam)." >}}
```

Here is the PowerShell script in its entirety.

### ZipFolder.ps1

```
function CountZipItems(
    [__ComObject] $zipFile)
{
    If ($zipFile -eq $null)
    {
        Throw "Value cannot be null: zipFile"
    }

    Write-Host ("Counting items in zip file (" + $zipFile.Self.Path + ")...")

    [int] $count = CountZipItemsRecursive($zipFile)

    Write-Host ($count.ToString() + " items in zip file (" `
        + $zipFile.Self.Path + ").")

    return $count
}

function CountZipItemsRecursive(
    [__ComObject] $parent)
{
    If ($parent -eq $null)
    {
        Throw "Value cannot be null: parent"
    }

    [int] $count = 0

    $parent.Items() |
        ForEach-Object {
            $count += 1

            If ($_.IsFolder -eq $true)
            {
                $count += CountZipItemsRecursive($_.GetFolder)
            }
        }

    return $count
}

function IsFileLocked(
    [string] $path)
{
    If ([string]::IsNullOrEmpty($path) -eq $true)
    {
        Throw "The path must be specified."
    }

    [bool] $fileExists = Test-Path $path

    If ($fileExists -eq $false)
    {
        Throw "File does not exist (" + $path + ")"
    }

    [bool] $isFileLocked = $true

    $file = $null

    Try
    {
        $file = [IO.File]::Open(
            $path,
            [IO.FileMode]::Open,
            [IO.FileAccess]::Read,
            [IO.FileShare]::None)

        $isFileLocked = $false
    }
    Catch [IO.IOException]
    {
        If ($_.Exception.Message.EndsWith(
            "it is being used by another process.") -eq $false)
        {
            Throw $_.Exception
        }
    }
    Finally
    {
        If ($file -ne $null)
        {
            $file.Close()
        }
    }

    return $isFileLocked
}

function GetWaitInterval(
    [int] $waitTime)
{
    If ($waitTime -lt 1000)
    {
        return 100
    }
    ElseIf ($waitTime -lt 5000)
    {
        return 1000
    }
    Else
    {
        return 5000
    }
}

function WaitForZipOperationToFinish(
    [__ComObject] $zipFile,
    [int] $expectedNumberOfItemsInZipFile)
{
    If ($zipFile -eq $null)
    {
        Throw "Value cannot be null: zipFile"
    }
    ElseIf ($expectedNumberOfItemsInZipFile -lt 1)
    {
        Throw "The expected number of items in the zip file must be specified."
    }

    Write-Host -NoNewLine "Waiting for zip operation to finish..."
    Start-Sleep -Milliseconds 100 # ensure zip operation had time to start

    [int] $waitTime = 0
    [int] $maxWaitTime = 60 * 1000 # [milliseconds]
    while($waitTime -lt $maxWaitTime)
    {
        [int] $waitInterval = GetWaitInterval($waitTime)

        Write-Host -NoNewLine "."
        Start-Sleep -Milliseconds $waitInterval
        $waitTime += $waitInterval

        Write-Debug ("Wait time: " + $waitTime / 1000 + " seconds")

        [bool] $isFileLocked = IsFileLocked($zipFile.Self.Path)

        If ($isFileLocked -eq $true)
        {
            Write-Debug "Zip file is locked by another process."
            Continue
        }
        Else
        {
            Break
        }
    }

    Write-Host

    If ($waitTime -ge $maxWaitTime)
    {
        Throw "Timeout exceeded waiting for zip operation"
    }

    [int] $count = CountZipItems($zipFile)

    If ($count -eq $expectedNumberOfItemsInZipFile)
    {
        Write-Debug "The zip operation completed succesfully."
    }
    ElseIf ($count -eq 0)
    {
        Throw ("Zip file is empty. This can occur if the operation is" `
            + " cancelled by the user.")
    }
    ElseIf ($count -gt $expectedCount)
    {
        Throw "Zip file contains more than the expected number of items."
    }
}

function ZipFolder(
    [IO.DirectoryInfo] $directory)
{
    If ($directory -eq $null)
    {
        Throw "Value cannot be null: directory"
    }

    Write-Host ("Creating zip file for folder (" + $directory.FullName + ")...")

    [IO.DirectoryInfo] $parentDir = $directory.Parent

    [string] $zipFileName

    If ($parentDir.FullName.EndsWith("\") -eq $true)
    {
        # e.g. $parentDir = "C:\"
        $zipFileName = $parentDir.FullName + $directory.Name + ".zip"
    }
    Else
    {
        $zipFileName = $parentDir.FullName + "\" + $directory.Name + ".zip"
    }

    If (Test-Path $zipFileName)
    {
        Throw "Zip file already exists ($zipFileName)."
    }

    Set-Content $zipFileName ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18))

    $shellApp = New-Object -ComObject Shell.Application
    $zipFile = $shellApp.NameSpace($zipFileName)

    If ($zipFile -eq $null)
    {
        Throw "Failed to get zip file object."
    }

    [int] $expectedCount = (Get-ChildItem $directory -Force -Recurse).Count
    $expectedCount += 1 # account for the top-level folder

    $zipFile.CopyHere($directory.FullName)

    # wait for CopyHere operation to complete
    WaitForZipOperationToFinish $zipFile $expectedCount

    Write-Host -Fore Green ("Successfully created zip file for folder (" `
        + $directory.FullName + ").")
}

Remove-Item "C:\NotBackedUp\Fabrikam.zip"

[IO.DirectoryInfo] $directory = Get-Item "C:\NotBackedUp\Fabrikam"
ZipFolder $directory
```

