---
title: PowerShell script to deploy/rebuild an ASP.NET Web application (a.k.a. Building TechnologyToolbox.com, part 8)
date: 2011-11-17T01:51:12-07:00
description:
  In this post, I describe the PowerShell script used to rebuild the Development
  and Test environments for TechnologyToolbox.com. From a high-level
  perspective, the script deletes the IIS website (if it exists), creates a new
  website (including the corresponding application pool), and then copies the
  files for the main site as well as the Subtext files for the /blog
  application...
aliases:
  [
    "/blog/jjameson/archive/2011/11/16/building-technologytoolbox-com-part-8.aspx",
    "/blog/jjameson/archive/2011/11/17/building-technologytoolbox-com-part-8.aspx",
  ]
categories: ["Development", "My System"]
tags: ["My System", "PowerShell", "Subtext", "Web Development"]
---

In this post, I describe the PowerShell script used to rebuild the Development
and Test environments for TechnologyToolbox.com. From a high-level perspective,
the script deletes the IIS website (if it exists), creates a new website
(including the corresponding application pool), and then copies the files for
the main site as well as the Subtext files for the /blog application.

You may recall the following illustration from the post I wrote
[introducing TechnologyToolbox.com](/blog/jjameson/2011/10/18/introducing-technologytoolbox-com).
This shows how the Caelum and Subtext Visual Studio solutions are merged
together during the deployment process.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Solution-Architecture-600x520.jpg"
alt="Solution architecture" height="520" width="600"
caption="Figure 1: Solution architecture" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Solution-Architecture-726x629.jpg)

### Step 1 - Set CAELUM_URL environment variable

In order to use a single script to rebuild both DEV and TEST, an environment
variable is used to determine which environment the script is currently running
in.

In DEV, the <var>CAELUM_URL</var> environment variable is set to
[http://www-dev.technologytoolbox.com](http://www-dev.technologytoolbox.com),
whereas in TEST the variable is set to
[http://www-test.technologytoolbox.com](http://www-test.technologytoolbox.com).

### Step 2 - Remove existing website and application pool

To cleanup the environment, the website and corresponding application pool are
deleted as part of the rebuild process. This ensures that no "stale" files or
configuration settings are left behind after the rebuild.

This is relatively easy to accomplish using the new IIS cmdlets (which are
available after importing the IIS Module into the PowerShell instance).

For a complete list of the IIS cmdlets, refer to the following Technet article:

{{< reference
title="Web Server (IIS) Administration Cmdlets in Windows PowerShell"
linkHref="http://technet.microsoft.com/en-us/library/ee790599.aspx" >}}

{{< div-block "note important" >}}

> **Important**
>
> If, like me, you are using Windows Server 2008 R2 (and IIS 7.5), then you
> cannot use the `Add-PSSSnapin WebAdministration` command specified in the
> above TechNet article. Instead you need to use
> `Import-Module WebAdministration` -- as shown below.

{{< /div-block >}}

```PowerShell
...
Import-Module WebAdministration

function RemoveWebsite(
    [string] $siteName = $(Throw "Value cannot be null: siteName"),
    [string] $sitePath = $(Throw "Value cannot be null: sitePath"))
{
    # HACK: Don't use "Get-Website -Name ..."
    # https://connect.microsoft.com/PowerShell/feedback/details/597787/get-website-always-returns-full-list-of-web-sites
    #
    #$site = Get-Website -Name $siteName

    $site = Get-Item ("IIS:\Sites\" + $siteName) -EA 0

    If ($site -ne $null -and $site.Name -eq $siteName)
    {
        Write-Host "Stopping website ($siteName)...`r`n"
        Stop-Website -Name $siteName
        Write-Host "Successfully stopped website ($siteName).`r`n"

        Write-Host "Removing website ($siteName)...`r`n"
        Remove-Website -Name $siteName
        Write-Host "Successfully removed website ($siteName).`r`n"
    }

    If (Test-Path $sitePath)
    {
        Write-Host "Removing website folder ($sitePath)...`r`n"
        Remove-Item $sitePath -Recurse
        Write-Host "Successfully removed website folder ($sitePath).`r`n"
    }

    $appPool = Get-Item ("IIS:\AppPools\" + $siteName) -EA 0

    If ($appPool -ne $null)
    {
        Write-Host "Removing application pool ($siteName)...`r`n"
        Remove-WebAppPool -Name $siteName
        Write-Host "Successfully removed application pool ($siteName).`r`n"
    }
}
```

{{< div-block-start "note" >}}

> **Note**
>
> There is currently a bug in the `Get-Website` cmdlet, so be very careful if
> you choose to use it:
>
> {{< reference title="Get-Website always returns full list of web sites" linkHref="https://connect.microsoft.com/PowerShell/feedback/details/597787/get-website-always-returns-full-list-of-web-sites" >}}

{{< div-block-end >}}

If you are wondering why I specify "``r`n`" at the end of each `Write-Host`
command, I'll explain that in
[a separate post](/blog/jjameson/2011/11/19/tips-tricks-for-running-powershell-scripts-as-scheduled-tasks).

### Step 3 - Create and configure the application pool

Like the Production environment, the website runs in a dedicated application
pool in DEV and TEST. Note that in the Test environment, the app pool must be
configured to run as **NetworkService** in order to access the database on a
different server (as I noted in
[my previous post](/blog/jjameson/2011/11/14/building-technologytoolbox-com-part-7)).
The default app pool identity can be used in DEV since the database runs on the
same server as the website in that environment.

```PowerShell
...
    $siteUrl = [System.Uri] $env:CAELUM_URL
...
    $siteName = $($siteUrl.Host)
    $appPoolIdentity = "IIS APPPOOL\" + $siteName

    If ($($siteUrl.AbsoluteUri) -eq "http://www-test.technologytoolbox.com/")
    {
        $appPoolIdentity = "NT AUTHORITY\Network Service"
    }
...
    Write-Host "Creating application pool ($siteName)...`r`n"
    $appPool = New-WebAppPool -Name $siteName
    If ($appPoolIdentity -eq "NT AUTHORITY\Network Service")
    {
        Write-Host "Setting app pool identity ($appPoolIdentity)...`r`n"
        $appPool.processModel.identityType = "NetworkService"
        $appPool | Set-Item
        Write-Host "Successfully set app pool identity ($appPoolIdentity).`r`n"
    }
```

{{< div-block "note" >}}

> **Tip**
>
> Using the
> **[System.Uri](http://msdn.microsoft.com/en-us/library/system.uri.aspx)**
> class from the .NET Framework is a convenient way to parse URLs in PowerShell
> (for example to extract the hostname, as shown above).

{{< /div-block >}}

### Step 4 - Create website folder under Inetpub\wwwroot

Prior to creating the IIS website, the corresponding folder must be created:

```PowerShell
$sitePath = "C:\inetpub\wwwroot\" + $siteName

Write-Host "Creating website folder ($sitePath)...`r`n"
New-Item -Type Directory -Path $sitePath > $null
Write-Host "Successfully created website folder ($sitePath).`r`n"
```

### Step 5 - Create IIS website

Next, the `New-Website` cmdlet is used to create the website in IIS:

```PowerShell
Write-Host "Creating website ($sitePath)...`r`n"
New-Website -Name $siteName -HostHeader $siteName `
    -PhysicalPath $sitePath -ApplicationPool $siteName > $null

Write-Host "Successfully created website ($sitePath).`r`n"
```

### Step 6 - Copy Subtext website content

The Subtext solution is deployed by copying files from the Release server
(DAZZLER). Note that variables are used to specify which version (e.g. "\_latest"
or "2.5.2.9") to install as well as the build configuration (i.e. "Debug" or
"Release").

```PowerShell
Write-Host "Copying Subtext website content...`r`n"
Copy-Item "\\dazzler\Builds\Subtext\$subtextVersion\$buildConfiguration\_PublishedWebsites\Subtext.Web" "$sitePath\blog" -Recurse
Write-Host "Successfully copied Subtext website content.`r`n"
```

For more details on the build process, refer to one of my posts from a couple of
years ago:

{{< reference title="Build and Deployment Overview"
linkHref="/blog/jjameson/2009/09/26/build-and-deployment-overview"
linkText="https://www.technologytoolbox.com/blog/jjameson/2009/09/26/build-and-deployment-overview" >}}

### Step 7 - Configure permissions on Subtext App_Data folder

Subtext requires read/write access to the App_Data folder, so I use the
**[icacls](http://technet.microsoft.com/en-us/library/cc753525%28WS.10%29.aspx)**
utility to configure the NTFS permissions:

```PowerShell
Write-Host "Granting read/write access on Subtext App_Data folder...`r`n"
icacls "$sitePath\blog\App_Data" /grant ($appPoolIdentity + ":(OI)(CI)(M)") | Out-Default
Write-Host "Successfully granted read/write access on Subtext App_Data folder.`r`n"
```

### Step 8 - Convert blog folder to application

The /blog folder must be converted to an application (due to some of the
configuration settings specified in the Subtext Web.config file). In IIS
Manager, this is accompished by right-clicking the **blog** folder and then
clicking **Convert to Application**. In PowerShell, this is accomplished using
the `New-WebApplication` command:

```PowerShell
Write-Host "Creating new application for blog site...`r`n"
New-WebApplication -Site $siteName -Name blog -PhysicalPath "$sitePath\blog" -ApplicationPool $siteName > $null
Write-Host "Successfully created new application for blog site.`r`n"
```

### Step 9 - Copy Caelum website content

Similar to step 6, the Caelum solution is deployed by copying files from the
Release server. As with the deployment of the Subtext solution, variables are
used to specify the version to deploy as well as the build configuration.

```PowerShell
Write-Host "Copying Caelum website content...`r`n"
Copy-Item \\dazzler\Builds\Caelum\$caelumVersion\$buildConfiguration\_PublishedWebsites\Website\* $sitePath -Recurse -Force
Write-Host "Successfully copied Caelum website content.`r`n"
```

### Step 10 - Overwrite Web.config files (as necessary)

While I like the _idea_ behind
[Web.config transform files](http://msdn.microsoft.com/en-us/library/dd465326.aspx),
I don't recommend them for most scenarios. Instead of limiting the
transformations to build configurations (e.g. Web.Debug.config and
Web.Release.config), I wish you could apply a transform based on some other
criteria (e.g. which environment you are deploying to).

For example, as I've noted in the past, I typically recommend deploying Debug
builds to the Test environment in the early part of your release cycle. Then,
when you are nearing a release, switch to deploying Release builds to that
environment.

I also don't like the idea of deploying something to Production that hasn't
previously been validated in the Test environment -- which is why I shudder when
I see customers create build configurations specific to each environment.

For these reasons, I create configuration files as necessary for each
environment (e.g. Web.TEST.config) and store these in source control. It takes a
little more discipline to keep these files in-sync, but using a tool like
[DiffMerge](/blog/jjameson/2009/03/24/diffmerge-a-better-differencing-tool)
makes this rather easy.

```PowerShell
If ($($siteUrl.AbsoluteUri) -eq "http://www-test.technologytoolbox.com/")
{
    Write-Host "Overwriting Web.config files...`r`n"
    Copy-Item "$sitePath\Web.TEST.config" "$sitePath\Web.config" -Force
    Copy-Item "$sitePath\blog\Web.TEST.config" "$sitePath\blog\Web.config" -Force
    Write-Host "Successfully overwrote Web.config files.`r`n"
}
```

### Putting it all together

Here is the complete script that incorporates all of the steps above.

#### Rebuild Website.ps1

```PowerShell
param(
    [string] $caelumVersion,
    [string] $subtextVersion,
    [string] $buildConfiguration)

$ErrorActionPreference = "Stop"

Import-Module WebAdministration

function RemoveWebsite(
    [string] $siteName = $(Throw "Value cannot be null: siteName"),
    [string] $sitePath = $(Throw "Value cannot be null: sitePath"))
{
    # HACK: Don't use "Get-Website -Name ..."
    # https://connect.microsoft.com/PowerShell/feedback/details/597787/get-website-always-returns-full-list-of-web-sites
    #
    #$site = Get-Website -Name $siteName

    $site = Get-Item ("IIS:\Sites\" + $siteName) -EA 0

    If ($site -ne $null -and $site.Name -eq $siteName)
    {
        Write-Host "Stopping website ($siteName)...`r`n"
        Stop-Website -Name $siteName
        Write-Host "Successfully stopped website ($siteName).`r`n"

        Write-Host "Removing website ($siteName)...`r`n"
        Remove-Website -Name $siteName
        Write-Host "Successfully removed website ($siteName).`r`n"
    }

    If (Test-Path $sitePath)
    {
        Write-Host "Removing website folder ($sitePath)...`r`n"
        Remove-Item $sitePath -Recurse
        Write-Host "Successfully removed website folder ($sitePath).`r`n"
    }

    $appPool = Get-Item ("IIS:\AppPools\" + $siteName) -EA 0

    If ($appPool -ne $null)
    {
        Write-Host "Removing application pool ($siteName)...`r`n"
        Remove-WebAppPool -Name $siteName
        Write-Host "Successfully removed application pool ($siteName).`r`n"
    }
}

function Main(
    [string] $caelumVersion,
    [string] $subtextVersion,
    [string] $buildConfiguration)
{
    If ([string]::IsNullOrEmpty($env:CAELUM_URL) -eq $true)
    {
        Throw "CAELUM_URL environment variable must be set."
    }

    $siteUrl = [System.Uri] $env:CAELUM_URL

    Write-Debug "Absolute URL: $($siteUrl.AbsoluteUri)"

    If (($($siteUrl.AbsoluteUri) -eq "http://www-local.technologytoolbox.com/") `
    	-or ($($siteUrl.AbsoluteUri) -eq "http://www-dev.technologytoolbox.com/"))
    {
        If ([string]::IsNullOrEmpty($caelumVersion) -eq $true)
        {
            Write-Host "Defaulting to latest build for Caelum...`r`n"
            $caelumVersion = "_latest"
        }

        If ([string]::IsNullOrEmpty($subtextVersion) -eq $true)
        {
            Write-Host "Defaulting to latest build for Subtext...`r`n"
            $subtextVersion = "_latest"
        }

        If ([string]::IsNullOrEmpty($buildConfiguration) -eq $true)
        {
            Write-Host "Defaulting to Debug build configuration...`r`n"
            $buildConfiguration = "Debug"
        }
    }

    If ([string]::IsNullOrEmpty($caelumVersion) -eq $true)
    {
        Throw "Caelum build version must be specified for this environment."
    }

    If ([string]::IsNullOrEmpty($subtextVersion) -eq $true)
    {
        Throw "Subtext build version must be specified for this environment."
    }

    If ([string]::IsNullOrEmpty($buildConfiguration) -eq $true)
    {
        Write-Host "Defaulting to Release build configuration...`r`n"
        $buildConfiguration = "Release"
    }

    $siteName = $($siteUrl.Host)

    $sitePath = "C:\inetpub\wwwroot\" + $siteName
    $appPoolIdentity = "IIS APPPOOL\" + $siteName

    If ($($siteUrl.AbsoluteUri) -eq "http://www-test.technologytoolbox.com/")
    {
        $appPoolIdentity = "NT AUTHORITY\Network Service"
    }

    Write-Debug "App pool identity: $appPoolIdentity"

    RemoveWebsite $siteName $sitePath

    Write-Host "Creating application pool ($siteName)...`r`n"
    $appPool = New-WebAppPool -Name $siteName
    If ($appPoolIdentity -eq "NT AUTHORITY\Network Service")
    {
        Write-Host "Setting app pool identity ($appPoolIdentity)...`r`n"
        $appPool.processModel.identityType = "NetworkService"
        $appPool | Set-Item
        Write-Host "Successfully set app pool identity ($appPoolIdentity).`r`n"
    }

    Write-Host "Successfully created application pool ($siteName).`r`n"

    Write-Host "Creating website folder ($sitePath)...`r`n"
    New-Item -Type Directory -Path $sitePath > $null
    Write-Host "Successfully created website folder ($sitePath).`r`n"

    Write-Host "Creating website ($sitePath)...`r`n"
    New-Website -Name $siteName -HostHeader $siteName `
        -PhysicalPath $sitePath -ApplicationPool $siteName > $null

    Write-Host "Successfully created website ($sitePath).`r`n"

    Write-Host "Copying Subtext website content...`r`n"
    Copy-Item "\\dazzler\Builds\Subtext\$subtextVersion\$buildConfiguration\_PublishedWebsites\Subtext.Web" "$sitePath\blog" -Recurse
    Write-Host "Successfully copied Subtext website content.`r`n"

    Write-Host "Granting read/write access on Subtext App_Data folder...`r`n"
    icacls "$sitePath\blog\App_Data" /grant ($appPoolIdentity + ":(OI)(CI)(M)") | Out-Default
    Write-Host "Successfully granted read/write access on Subtext App_Data folder.`r`n"

    Write-Host "Creating new application for blog site...`r`n"
    New-WebApplication -Site $siteName -Name blog -PhysicalPath "$sitePath\blog" -ApplicationPool $siteName > $null
    Write-Host "Successfully created new application for blog site.`r`n"

    Write-Host "Copying Caelum website content...`r`n"
    Copy-Item \\dazzler\Builds\Caelum\$caelumVersion\$buildConfiguration\_PublishedWebsites\Website\* $sitePath -Recurse -Force
    Write-Host "Successfully copied Caelum website content.`r`n"

    If ($($siteUrl.AbsoluteUri) -eq "http://www-test.technologytoolbox.com/")
    {
        Write-Host "Overwriting Web.config files...`r`n"
        Copy-Item "$sitePath\Web.TEST.config" "$sitePath\Web.config" -Force
        Copy-Item "$sitePath\blog\Web.TEST.config" "$sitePath\blog\Web.config" -Force
        Write-Host "Successfully overwrote Web.config files.`r`n"
    }

    Write-Host -Fore Green "Successfully rebuilt Web application.`r`n"
}

Main $caelumVersion $subtextVersion $buildConfiguration
```

### Automated deployments to DEV

As shown in the following figure, I recommend performing automated deployments
to DEV on a daily basis (or more frequently depending on where you are in the
release cycle).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/3-Automated-build-and-deployment-600x343.png"
alt="Automated build and deployment" height="343" width="600"
caption="Figure 2: Automated build and deployment" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/3-Automated-build-and-deployment-926x530.png)

While you could use the new "Lab Management" features in Visual Studio 2010 to
perform the deployments, for years I've simply been using scheduled tasks to
automatically deploy the latest build to DEV.

To invoke the PowerShell script and capture all of the commands and
corresponding output, I created a simple batch file.

#### Rebuild Website.cmd

```Batch
PowerShell.exe -Command ".\'Rebuild Website.ps1'; Exit $LASTEXITCODE" > "Rebuild Website.log" 2>&1

EXIT %ERRORLEVEL%
```

### "Manual" deployments to TEST

While the DEV environment is automatically rebuilt (at least) once per day,
deployments to the Test environment occur much less often -- for example, when a
beta version of the solution is ready.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/4-Installing-the-Beta-1-version-to-TEST-600x343.png"
alt="\"Manual\" deployment to TEST" height="343" width="600"
caption="Figure 4: \"Manual\" deployment to TEST" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/4-Installing-the-Beta-1-version-to-TEST-940x538.png)

The reason I put the quotes around "manual" is because most of the deployment is
scripted. The only thing that needs to be done is to open a PowerShell window
(using the **Run as administrator** option) and execute the script above.:

```Text
PS C:\> {{< kbd "cd 'C:\NotBackedUp\TechnologyToolbox\Caelum\Main\Source\Deployment Files\Scripts'" >}}
PS C:\NotBackedUp\...\Scripts> {{< kbd "& '.\Rebuild Website.ps1' 1.0.57.0 2.5.2.9" >}}
```

Note that when running the script in TEST, the Caelum and Subtext versions must
be specified (unlike DEV, which assumes "\_latest" if no versions are
specified).
