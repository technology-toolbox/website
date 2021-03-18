---
title: Packaging a code sample using PowerShell
date: 2012-02-28T02:32:40-07:00
excerpt:
  "Here is the PowerShell script I developed to avoid accidentally including
  \"junk\" in code samples I create for my blog."
aliases:
  [
    "/blog/jjameson/archive/2012/02/27/packaging-a-code-sample-using-powershell.aspx",
    "/blog/jjameson/archive/2012/02/28/packaging-a-code-sample-using-powershell.aspx",
  ]
draft: true
categories: ["Development"]
tags: ["PowerShell", "TFS"]
---

In
[my post from earlier this morning](/blog/jjameson/2012/02/27/zip-a-folder-using-powershell),
I mentioned how I now use PowerShell to package code samples for my blog to
avoid accidentally including "junk" in the zip files (e.g. temporary object
folders created during the build).

Here is the script in hopes that it may help other developers out there.

Note that this script assumes the code sample is stored in Team Foundation
Server, so it first cleans the working folder, then performs a "get latest
version" on the specified folder, builds the solution, removes some extraneous
items (e.g. "obj" folders), and finally packages the folder into a zip file.

### PackageCodeSample.ps1

```
$ErrorActionPreference = "Stop"

Import-Module Pscx -EA 0

function GetSolutionFile(
    [string] $path)
{
    If ([string]::IsNullOrEmpty($path) -eq $true)
    {
        Throw "Path must be specified."
    }

    $solutionFile = Get-ChildItem $path -Filter *.sln -Recurse

    If ($solutionFile -eq $null)
    {
        Throw "Solution file not found."
    }
    ElseIf ($solutionFile -is [Array])
    {
        Throw "More than one solution file was found."
    }

    return $solutionFile
}

function RemoveExtraneousItems(
    [string] $path)
{
    If ([string]::IsNullOrEmpty($path) -eq $true)
    {
        Throw "Path must be specified."
    }

    Write-Host "Removing extraneous items from folder ($path)..."

    Get-ChildItem $path -Include obj -Recurse |
        Remove-Item -Recurse
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

    Write-Zip $directory.FullName -OutputPath $zipFileName -IncludeEmptyDirectories
}

function PackageCodeSample(
    [string] $codeSampleFolder)
{
    If ([string]::IsNullOrEmpty($codeSampleFolder) -eq $true)
    {
        Throw "Code sample folder must be specified."
    }

    Write-Host ("Packaging code sample ($codeSampleFolder)...")

    If (Test-Path $codeSampleFolder)
    {
        Remove-Item $codeSampleFolder -Force -Recurse
    }

    [string] $parentFolder = [Io.Path]::GetDirectoryName($codeSampleFolder)
    [string] $codeSampleFolderBaseName = [Io.Path]::GetFileName(
        $codeSampleFolder)

    Push-Location $parentFolder

    $tf = "${env:ProgramFiles(x86)}" `
            + "\Microsoft Visual Studio 10.0\Common7\IDE\TF.exe"

    & $tf get $codeSampleFolderBaseName /force /recursive

    Pop-Location

    $solutionFile = GetSolutionFile $codeSampleFolder

    Push-Location $solutionFile.DirectoryName

    $msbuild = "${env:windir}" `
            + "\Microsoft.NET\Framework\v3.5\MSBuild.exe"

    & $msbuild $solutionFile.Name

    Pop-Location

    [IO.DirectoryInfo] $directory = Get-Item $codeSampleFolder

    RemoveExtraneousItems $directory

    ZipFolder $directory

    Write-Host -Fore Green ("Successfully packaged code sample (" `
        + $directory.FullName + ").")

}
```

To package a code sample, first ensure the resulting zip file does not exist,
and then simply call the `PackageCodeSample` function, specifying the path to
the code to package:

```
Remove-Item "C:\NotBackedUp\Fabrikam\Demo\Dev\SharePoint2010CodeCoverage.zip"

PackageCodeSample "C:\NotBackedUp\Fabrikam\Demo\Dev\SharePoint2010CodeCoverage"
```

> **Note**
>
> This script uses the
> [PowerShell Community Extensions](http://pscx.codeplex.com/) to create the zip
> file. If you can't or don't want to install PSCX, refer to my previous post
> for a way to create a zip file using out-of-the-box PowerShell and some
> scriptable COM objects from the Windows Shell.
