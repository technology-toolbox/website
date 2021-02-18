---
title: "Use PowerShell to alleviate the pain of code coverage analysis in Visual Studio 2010 and .NET 3.5 solutions (e.g. SharePoint 2010)"
date: 2012-02-23T14:17:26-07:00
lastmod: 2012-02-23T14:18:01-07:00
excerpt: "This PowerShell script makes it much easier to perform code coverage analysis in Visual Studio 2010 and .NET Framework 3.5 solutions (e.g. SharePoint 2010)."
aliases: ["/blog/jjameson/archive/2012/02/23/use-powershell-to-alleviate-the-pain-of-code-coverage-analysis.aspx"]
draft: true
categories: ["Development", "SharePoint"]
tags: ["Core 
			Development", "MOSS 2007", "PowerShell", "SharePoint 
			2010", "Visual Studio"]
---

In
[my post from earlier today](/blog/jjameson/2012/02/22/code-coverage-analysis-with-visual-studio-2010-and-net-3), I noted how the code coverage analysis feature
in Visual Studio 2010 is so easy to configure there's really no excuse not to
use it -- provided your test projects target .NET Framework 4. However if, like
me, you need to target .NET 3.5 (e.g. when developing for SharePoint 2010),
then the instructions in the Visual Studio documentation for
[configuring
code coverage](http://msdn.microsoft.com/en-us/library/dd504821.aspx) simply don't work.

Instead of using the Code Coverage data diagnostic adapter within the Visual
Studio IDE, you need to instrument your .NET 3.5 assemblies "manually" using
[VSInstr](http://msdn.microsoft.com/en-us/library/ms182402.aspx),
and subsequently start/stop the code coverage profiler using
[VSPerfCmd](http://msdn.microsoft.com/en-us/library/ms182403.aspx).

To make this process relatively painless, I created a PowerShell script to
perform the following:

1. Instrument the assemblies that will be analyzed for code coverage
2. Re-sign the assemblies (since the process of instrumenting an assembly
   removes the strong name)
3. "Deploy" the instrumented assemblies (e.g. copy the instrumented assemblies
   to the "bin" folders for the assemblies containing the unit/integration
   tests)
4. Start the code coverage profiler
5. Run the unit/integration tests
6. Stop the code coverage profiler

### Step 1: Instrument the assemblies

The first step in using code coverage with Visual Studio 2010 and .NET 3.5
projects is to instrument the assemblies using
[VSInstr](http://msdn.microsoft.com/en-us/library/ms182402.aspx).

Let's start by defining a list of the assemblies that will be analyzed for
code coverage:

```
[string[]] $assembliesToInstrument =
    @(
        "CoreServices\bin\Debug\Fabrikam.Demo.CoreServices.dll",
        ("CoreServices\SharePoint\bin\Debug" `
            + "\Fabrikam.Demo.CoreServices.SharePoint.dll")
    )
```

For the sake of understanding the example in this post, imagine Fabrikam
has a Visual Studio solution containing two projects:

- CoreServices
- CoreServices.SharePoint

The Fabrikam.Demo.CoreServices assembly contains shared code used throughout
the solution (such as the **StringHelper** class). The Fabrikam.Demo.CoreServices.SharePoint
assembly contains common code used when developing SharePoint solutions (such
as the **SharePointSecurityHelper** class).

Next we need to instrument each assembly in the list:

```
$assembliesToInstrument |
        ForEach-Object {
            InstrumentAssembly $_            
        }
```

The `InstrumentAssembly`
function simply executes
[VSInstr](http://msdn.microsoft.com/en-us/library/ms182402.aspx)
on the specified assembly:

```
function InstrumentAssembly(
    [string] $assemblyPath = $(Throw "Value cannot be null: assemblyPath"))
{
    [string] $vsinstr = "${env:ProgramFiles(x86)}" `
        + "\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\x64\VSInstr.exe"

    & $vsinstr "$assemblyPath" /coverage
}
```

### Step 2: Re-sign the assemblies

When you run
[VSInstr](http://msdn.microsoft.com/en-us/library/ms182402.aspx)
against a signed assembly, the following warning is emitted:

{{< blockquote "font-italic" >}}

Warning VSP2001: ...\bin\Debug\Fabrikam.Demo.CoreServices.dll is a strongly named assembly. It will need to be re-signed before it can be executed.

{{< /blockquote >}}

This is why the **Code Coverage Detail** dialog in Visual Studio
allows you to specify a **Re-signing key file**.

To re-sign the instrumented assembly from the PowerShell script, we need
to use the Strong Name Tool (Sn.exe):

```
$assembliesToInstrument |
        ForEach-Object {
            InstrumentAssembly $_
            
            SignAssembly $_
        }
```

The `SignAssembly` function
simply executes
[Sn.exe](http://msdn.microsoft.com/en-us/library/k5b5tt23.aspx) on
the instrumented assembly:

```
function SignAssembly(
    [string] $assemblyPath = $(Throw "Value cannot be null: assemblyPath"))
{
    [string] $sn = "${env:ProgramFiles(x86)}" `
        + "\Microsoft SDKs\Windows\v7.0A\Bin\NETFX 4.0 Tools\sn.exe"
        
    & $sn -q -Ra "$assemblyPath" Fabrikam.Demo.snk
}
```

### Step 3: "Deploy" the instrumented assemblies

To ensure the instrumented assemblies are used when running the corresponding
tests, the modified assemblies need to be copied to the "bin" folders for the
test projects.

Let's start by defining the list of assemblies containing the unit/integration
tests. For this example, assume that each project has a corresponding test project:

- CoreServices.DeveloperTests
- CoreServices.Sharepoint.DeveloperTests

The corresponding array variable in PowerShell is:

```
[string[]] $testAssemblies =
    @(
        ("CoreServices\DeveloperTests\bin\Debug" `
            + "\Fabrikam.Demo.CoreServices.DeveloperTests.dll"),
        ("CoreServices\SharePoint\DeveloperTests\bin\Debug" `
            + "\Fabrikam.Demo.CoreServices.SharePoint.DeveloperTests.dll")
    )
```

To copy the instrumented assemblies into the "bin" folders for the test projects,
we first need to get the list of folders containing the test assemblies:

```
$testBinFolders = GetAssemblyFolders($testAssemblies)
```

The `GetAssemblyFolders`function is straightforward:

```
function GetAssemblyFolders(
    [string[]] $assemblyPaths = $(Throw "Value cannot be null: assemblyPaths"))
{
    [string[]] $folders = @()
    
    $assemblyPaths |
        ForEach-Object {
            [string] $folder = (Get-Item $_).DirectoryName

            $folders += $folder
        }

    return $folders
}
```

With the list of instrumented assemblies and the list of folders containing
the test assemblies, the next step is to copy the modified assemblies to the
destination folders:

```
$assembliesToInstrument |
        ForEach-Object {
            InstrumentAssembly $_
            
            SignAssembly $_

            CopyInstrumentedAssemblyToTestBinFolders $_ $testBinFolders
        }
```

The function simply uses the **Copy-Item** cmdlet to copy the
instrumented assembly to each destination "bin" older:

```
function CopyInstrumentedAssemblyToTestBinFolders(
    [string] $assemblyPath = $(Throw "Value cannot be null: assemblyPath"),    
    [string[]] $testBinFolders = $(Throw "Value cannot be null: testBinFolders"))
{
    $testBinFolders |
        ForEach-Object {
            Copy-Item $assemblyPath $_
        }
}
```

Also note that if the assemblies are deployed to the global assembly cache
(e.g. for a SharePoint feature receiver), then we need to update the assembly
in the GAC as well:

```
$assembliesToInstrument |
        ForEach-Object {
            InstrumentAssembly $_
            
            SignAssembly $_

            CopyInstrumentedAssemblyToTestBinFolders $_ $testBinFolders

            UpdateGacAssemblyIfNecessary $_
        }
```

The `UpdateGacAssemblyIfNecessary`
function uses **gacutil.exe** to check if the assembly is already
in the GAC (in which case it is replaced with the instrumented assembly). If
the assembly is not already in the GAC, then no action is performed.

```
function UpdateGacAssemblyIfNecessary(
    [string] $assemblyPath = $(Throw "Value cannot be null: assemblyPath"))
{
    [string] $baseName = (Get-Item $assemblyPath).BaseName

    [string] $gacutil = "${env:ProgramFiles(x86)}" `
        + "\Microsoft SDKs\Windows\v7.0A\Bin\gacutil.exe"

    [string] $numberOfItemsInGac = & $gacutil -l $baseName |
        Select-String "^Number of items =" |
            ForEach { $_.Line.Split("=")[1].Trim() }
            
    If ($numberOfItemsInGac -eq "0")
    {
        Write-Debug ("The assembly (" + $baseName + ") was not found in the GAC.")
    }
    ElseIf ($numberOfItemsInGac -eq "1")
    {
        & $gacutil /if $assemblyPath
    }
    Else
    {
        Throw "Unexpected number of items in the GAC: " + $numberOfItemsInGac
    }
}
```

> **Important**
>
> This script does not currently support side-by-side versions of the same assembly in the GAC. An exception is thrown if more than one matching assembly is found in the GAC.

> **Note**
>
> I've seen a number of resources that suggest using the **[Publish.GacInstall](http://msdn.microsoft.com/en-us/library/system.enterpriseservices.internal.publish.gacinstall.aspx)**method (in the **System.EnterpriseServices.Internal** namespace) to install an assembly in the GAC. However, I also recall seeing an MSDN blog post that indicated this was only intended to be used for adding COM interop assemblies and that **gacutil.exe** should be used instead. For Development environments, using **gacutil.exe** is acceptable, but for Test and Production environments, you should not rely on **gacutil.exe** for installing assemblies in the GAC.

### Step 4: Start the code coverage profiler

After a little refactoring in the script, I ended up with a function used
to start (and stop) the code coverage profiler:

```
function ToggleCodeCoverageProfiling(
    [bool] $enable)
{
    [string] $vsperfcmd = "${env:ProgramFiles(x86)}" `
        + "\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\x64\VSPerfCmd.exe"

    If ($enable -eq $true)
    {
        & $vsperfcmd /START:COVERAGE /OUTPUT:Fabrikam.Demo
    }
    Else
    {
        & $vsperfcmd /SHUTDOWN
    }
}
```

To start the code coverage profiler, simply call the function and pass
`$true`:

```
ToggleCodeCoverageProfiling $true
```

### Step 5: Run the unit/integration tests

With the code coverage profiler running, the next step is to run the unit/integration
tests. Note that in order to force the tests to run in a 64-bit process (in
order for the SharePoint tests to work), a test settings file must be specified:

```
[string] $testSettingsPath = "LocalTestRun.testrunconfig"

    ...

    $assembliesToInstrument |
        ForEach-Object {
            ...
        }
    
    ToggleCodeCoverageProfiling $true

    RunTests $testAssemblies $testSettingsPath
```

In order to consolidate the results from multiple test projects, I execute
**mstest.exe** only once and specify all of the test assemblies
using separate **/testcontainer** parameters (one for each test
assembly):

```
function RunTests(
    [string[]] $assemblyPaths = $(Throw "Value cannot be null: assemblyPaths"),
    [string] $testSettingsPath)
{
    [string] $mstest = "${env:ProgramFiles(x86)}" `
        + "\Microsoft Visual Studio 10.0\Common7\IDE\MSTest.exe"

    [string[]] $parameters = @("/nologo")

    $assemblyPaths |
        ForEach-Object {
            $parameters += ('/testcontainer:"' + $_ + '"')
        }
    
    If ([string]::IsNullOrEmpty($testSettingsPath) -eq $false)
    {
        $parameters += ('/testsettings:' + $testSettingsPath)
    }

    & $mstest $parameters
}
```

### Step 6: Stop the code coverage profiler

Once the unit/integration tests have completed, the final step is to stop
the code coverage profiler:

```
ToggleCodeCoverageProfiling $false
```

At this point, opening the code coverage file (Fabrikam.Demo.coverage) in
Visual Studio displays the results of the analysis.

Here is the script in its entirety.

### Run Developer Tests with Code Coverage.ps1

```
$ErrorActionPreference = "Stop"

function CopyInstrumentedAssemblyToTestBinFolders(
    [string] $assemblyPath = $(Throw "Value cannot be null: assemblyPath"),    
    [string[]] $testBinFolders = $(Throw "Value cannot be null: testBinFolders"))
{
    $testBinFolders |
        ForEach-Object {
            Write-Debug ("Copying assembly (" + $assemblyPath `
                + ") to folder (" + $_ + ")...")

            Copy-Item $assemblyPath $_
        }
}

function GetAssemblyFolders(
    [string[]] $assemblyPaths = $(Throw "Value cannot be null: assemblyPaths"))
{
    [string[]] $folders = @()
    
    $assemblyPaths |
        ForEach-Object {
            [string] $folder = (Get-Item $_).DirectoryName

            $folders += $folder
        }

    return $folders
}

function InstrumentAssembly(
    [string] $assemblyPath = $(Throw "Value cannot be null: assemblyPath"))
{
    [string] $vsinstr = "${env:ProgramFiles(x86)}" `
        + "\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\x64\VSInstr.exe"

    & $vsinstr "$assemblyPath" /coverage
}

function RunTests(
    [string[]] $assemblyPaths = $(Throw "Value cannot be null: assemblyPaths"),
    [string] $testSettingsPath)
{
    [string] $mstest = "${env:ProgramFiles(x86)}" `
        + "\Microsoft Visual Studio 10.0\Common7\IDE\MSTest.exe"

    [string[]] $parameters = @("/nologo")

    $assemblyPaths |
        ForEach-Object {
            $parameters += ('/testcontainer:"' + $_ + '"')
        }
    
    If ([string]::IsNullOrEmpty($testSettingsPath) -eq $false)
    {
        $parameters += ('/testsettings:' + $testSettingsPath)
    }

    Write-Debug "Running tests..."
    & $mstest $parameters
}

function SignAssembly(
    [string] $assemblyPath = $(Throw "Value cannot be null: assemblyPath"))
{
    Write-Debug ("Signing assembly (" + $assemblyPath + ")...")

    [string] $sn = "${env:ProgramFiles(x86)}" `
        + "\Microsoft SDKs\Windows\v7.0A\Bin\NETFX 4.0 Tools\sn.exe"
        
    & $sn -q -Ra "$assemblyPath" Fabrikam.Demo.snk
}

function ToggleCodeCoverageProfiling(
    [bool] $enable)
{
    [string] $vsperfcmd = "${env:ProgramFiles(x86)}" `
        + "\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\x64\VSPerfCmd.exe"

    If ($enable -eq $true)
    {
        Write-Debug "Starting code coverage profiler..."

        & $vsperfcmd /START:COVERAGE /OUTPUT:Fabrikam.Demo
    }
    Else
    {
        Write-Debug "Stopping code coverage profiler..."

        & $vsperfcmd /SHUTDOWN
    }
}

function UpdateGacAssemblyIfNecessary(
    [string] $assemblyPath = $(Throw "Value cannot be null: assemblyPath"))
{
    [string] $baseName = (Get-Item $assemblyPath).BaseName

    Write-Debug ("Checking if assembly (" + $baseName + ") is in the GAC...")

    [string] $gacutil = "${env:ProgramFiles(x86)}" `
        + "\Microsoft SDKs\Windows\v7.0A\Bin\gacutil.exe"

    [string] $numberOfItemsInGac = & $gacutil -l $baseName |
        Select-String "^Number of items =" |
            ForEach { $_.Line.Split("=")[1].Trim() }
            
    If ($numberOfItemsInGac -eq "0")
    {
        Write-Debug ("The assembly (" + $baseName + ") was not found in the GAC.")
    }
    ElseIf ($numberOfItemsInGac -eq "1")
    {
        Write-Debug ("Updating GAC assembly (" + $baseName + ")...")

        & $gacutil /if $assemblyPath
    }
    Else
    {
        Throw "Unexpected number of items in the GAC: " + $numberOfItemsInGac
    }
}

function Main
{
    [string] $testSettingsPath = "LocalTestRun.testrunconfig"

    [string[]] $assembliesToInstrument =
    @(
        "CoreServices\bin\Debug\Fabrikam.Demo.CoreServices.dll",
        ("CoreServices\SharePoint\bin\Debug" `
            + "\Fabrikam.Demo.CoreServices.SharePoint.dll")
    )
    
    [string[]] $testAssemblies =
    @(
        ("CoreServices\DeveloperTests\bin\Debug" `
            + "\Fabrikam.Demo.CoreServices.DeveloperTests.dll"),
        ("CoreServices\SharePoint\DeveloperTests\bin\Debug" `
            + "\Fabrikam.Demo.CoreServices.SharePoint.DeveloperTests.dll")
    )

    [string[]] $testBinFolders = GetAssemblyFolders($testAssemblies)

    $assembliesToInstrument |
        ForEach-Object {
            InstrumentAssembly $_
            
            SignAssembly $_

            CopyInstrumentedAssemblyToTestBinFolders $_ $testBinFolders

            UpdateGacAssemblyIfNecessary $_
        }
    
    ToggleCodeCoverageProfiling $true

    RunTests $testAssemblies $testSettingsPath

    ToggleCodeCoverageProfiling $false
}

Main
```

### Sample Visual Studio solution

I have attached a sample Visual Studio 2010 solution that contains a couple
of assemblies that target .NET Framework 3.5 (one "generic" assembly and another
that contains some SharePoint-specific code) as well as corresponding unit/integration
tests. You should be able to extract the files, create an "[http://fabrikam-local](http://fabrikam-local)"
Web application in SharePoint 2010 (or use the FABRIKAM\_DEMO\_URL environment
variable to point to one of your existing Web applications), and then run the
PowerShell script from the **Source** folder to perform code coverage
analysis:

```
& '.\Run Developer Tests with Code Coverage.ps1'
```

