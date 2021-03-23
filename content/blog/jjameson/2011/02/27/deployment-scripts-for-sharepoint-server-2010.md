---
title: Deployment Scripts for SharePoint Server 2010
date: 2011-02-27T08:19:00-07:00
excerpt:
  "A couple of years ago, I shared the scripts I created for deploying solutions
  based on Microsoft Office SharePoint Server (MOSS) 2007, or what I like to
  refer to as the \" DR.DADA approach to SharePoint .\" Well, I probably should
  have done this long..."
aliases:
  [
    "/blog/jjameson/archive/2011/02/26/deployment-scripts-for-sharepoint-server-2010.aspx",
    "/blog/jjameson/archive/2011/02/27/deployment-scripts-for-sharepoint-server-2010.aspx",
  ]
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "SharePoint 2010", "PowerShell"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/02/27/deployment-scripts-for-sharepoint-server-2010.aspx"
---

A couple of years ago, I shared the scripts I created for deploying solutions
based on Microsoft Office SharePoint Server (MOSS) 2007, or what I like to refer
to as the
"[DR.DADA approach to SharePoint](/blog/jjameson/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint)."

Well, I probably should have done this long before, but this week I finally got
around to upgrading my MOSS 2007 batch files (a.k.a. "deployment scripts") to
SharePoint Server 2010 and PowerShell.

This morning, I thought it would be helpful to share these scripts for those of
you working with the "latest and greatest" in the world of SharePoint.

Note that you can download the complete set of scripts I refer to in this
article via the attachment to a
[post I wrote earlier this week](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010).
Even if you aren't (yet) interested in claims authentication in SharePoint 2010,
I think you'll still find the previous post valuable for understanding how your
development team can quickly build (or rebuild) a custom Web application based
on SharePoint 2010.

Suppose we are migrating the public Internet site for Fabrikam Technologies
([http://www.fabrikam.com](http://www.fabrikam.com)) to SharePoint Server 2010.
We know that we'll need some customization and therefore we've created a custom
SharePoint solution (Fabrikam.Demo.Web.wsp) that includes the following
features:

- Fabrikam.Demo.Web\_HomeSiteConfiguration
- Fabrikam.Demo.Web\_WebAppConfiguration
- Fabrikam.Demo.Web\_WebParts

Let's start by assuming we've got a brand new SharePoint Server 2010 environment
setup with "nothing" on it. In other words, we haven't yet created a Web
application or deployed/activated any custom features. Therefore we first need a
script to quickly create a new Web application in SharePoint.

### Create Web Application.ps1

This script is used to create the Fabrikam Web application in SharePoint:

```PowerShell
$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function Main()
{
    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    Write-Host "Creating Web application ($webAppUrl)..."

    $hostHeader = $webAppUrl.Substring("http://".Length)

    $webAppName = "SharePoint - " + $hostHeader + "80"

    $membershipProviderName = "FabrikamSqlMembershipProvider"
    $roleProviderName = "FabrikamSqlRoleProvider"
    $contentDatabaseName = "WSS_Content_FabrikamDemo"
    $appPoolName = $webAppName

    $appPoolUserName = $env:USERDOMAIN + "\svc-web-fabrikam"

    if (($webAppUrl -eq "http://www-local.fabrikam.com") -or
        ($webAppUrl -eq "http://www-dev.fabrikam.com"))
    {
    	Write-Debug "Overriding variables for Development environment..."
        $appPoolUserName = $env:USERDOMAIN + "\svc-web-fabrikam-dev"
    }
    elseif ($webAppUrl -eq "http://www-test.fabrikam.com")
    {
    	Write-Debug "Overriding variables for Test environment..."
        $appPoolUserName = $env:USERDOMAIN + "\svc-web-fabrikam-test"
    }

    Write-Debug "hostHeader: $hostHeader"
    Write-Debug "webAppName: $webAppName"
    Write-Debug "appPoolName: $appPoolName"
    Write-Debug "appPoolUserName: $appPoolUserName"
    Write-Debug "contentDatabaseName: $contentDatabaseName"

    Write-Debug "Get service account for application pool ($appPoolUserName)..."
    $appPoolAccount = Get-SPManagedAccount -Identity $appPoolUserName `
        -Debug:$false -EA 0

    If ($appPoolAccount -eq $null)
    {
        Write-Host "Registering managed account ($appPoolUserName)..."

        Write-Debug "Get credential ($appPoolUserName)..."
        $appPoolCredential = Get-Credential $appPoolUserName

        $appPoolAccount = New-SPManagedAccount -Credential $appPoolCredential `
            -Debug:$false
    }

    $windowsAuthProvider = New-SPAuthenticationProvider -Debug:$false
    $formsAuthProvider = New-SPAuthenticationProvider `
        -ASPNETMembershipProvider $membershipProviderName `
        -ASPNETRoleProviderName $roleProviderName `
        -Debug:$false

    $authProviders = $windowsAuthProvider, $formsAuthProvider

    $webApp = New-SPWebApplication -Name $webAppName -AllowAnonymousAccess `
        -ApplicationPool $appPoolName -AuthenticationMethod "NTLM" `
        -ApplicationPoolAccount $appPoolAccount -Url $webAppUrl -Port 80 `
        -AuthenticationProvider $authProviders -HostHeader $hostHeader `
        -DatabaseName $contentDatabaseName `
        -Debug:$false

    Write-Host -Fore Green "Successfully created Web application ($webAppUrl)."
}

Main
```

If you aren't familiar with the "`-EA 0`" abbreviated syntax, just realize that
it's a short way of saying "`-ErrorAction SilentlyContinue`", or in other words
you are telling PowerShell, "there's a chance this command may generate an
error, but I'm okay with that -- just ignore it." For example, if you try to add
the SharePoint PowerShell snap-in but the snap-in has already been added, then
you get an error. However, in order to run the script from Windows PowerShell
ISE (Integrated Shell Environment) or from a plain ol' PowerShell prompt (i.e.
not the **SharePoint 2010 Management Shell** shortcut), then we need to ensure
the SharePoint snap-in is loaded.

The rest of the script should be pretty obvious. However, it might be worth
pointing out that I explicitly add the "`-Debug:$false`" parameter to the
various SharePoint cmdlets because I often set `$DebugPreference = "Continue"`
in order to get debug messages from my scripts, but I don't want to see the
SharePoint debug messages.

It's also worth mentioning that the URL for the Fabrikam Web site is expected to
vary by environment (e.g. DEV, TEST, and PROD) and therefore the
**FABRIKAM\_DEMO\_URL** environment variable can be used to specify the URL of
the SharePoint Web application. For example, in the Development integration
environment, the URL of the site is
[http://www-dev.fabrikam.com](http://www-dev.fabrikam.com). This becomes
important later on when deploying and retracting solutions.

Also keep in mind that this script creates a Web application with claims
authentication enabled. Therefore you might need to tweak it slightly if you
just want "Classic Mode Authentication."

### Create Site Collections.ps1

At this point, we have a brand new Web application but it doesn't contain any
site collections. While the Fabrikam Internet site will likely end up having
numerous site collections, let's start out by simply creating the top-level site
(i.e. "/") using the **Publishing Portal** site definition.

```PowerShell
$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function CreateSiteCollection(
    [string] $siteUrl = $(Throw "Value cannot be null: siteUrl"),
    [string] $ownerAlias = $(Throw "Value cannot be null: ownerAlias"),
    [string] $siteName = $(Throw "Value cannot be null: siteName"),
    [string] $siteTemplate = $(Throw "Value cannot be null: siteTemplate"),
    [string] $siteDescription)
{
    Write-Host "Creating site collection ($siteUrl)..."

    Write-Debug "ownerAlias: $ownerAlias"
    Write-Debug "siteName: $siteName"
    Write-Debug "siteTemplate: $siteTemplate"
    Write-Debug "siteDescription: $siteDescription"

    New-SPSite $siteUrl -OwnerAlias $ownerAlias -Name $siteName `
        -Description $siteDescription -Template $siteTemplate -Debug:$false > $null

    Write-Host -Fore Green "Successfully created site collection ($siteUrl)."
}

function Main
{
    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    Write-Debug "webAppUrl: $webAppUrl"

    $ownerAlias = $env:USERDOMAIN + "\" + $env:USERNAME

    # Create default site collection (i.e. the "root Web")
    $siteUrl = $webAppUrl + "/"
    $siteName = "Fabrikam"
    $siteTemplate = "BLANKINTERNETCONTAINER#0"
    $siteDescription = "Public Internet site for Fabrikam Technologies"
    CreateSiteCollection $siteUrl $ownerAlias $siteName $siteTemplate `
        $siteDescription
}

Main
```

There's not much worth noting about this script, except perhaps to be aware that
it is hard-coded to set the primary site collection administrator to the user
running the script.

### Enable Anonymous Access.ps1

For an Internet-facing site, I can't think of any scenario where we wouldn't
want to allow anonymous access to at least some part of the site (even for an
extranet site, at a minimum, we would probably want to support a custom login
page as well as some generic content, such as terms and conditions for using the
site).

For the Fabrikam site, most of the content will be available to anonymous users.
Therefore, we use a script to avoid having to repeatedly configure anonymous
access through **Site Actions** -&gt; **Site Permissions** (a.k.a.
/\_layouts/user.aspx). By "repeatedly" I am referring to performing this
configuration change in each environment (DEV, TEST, and PROD) or whenever a
developer decides to rebuild the Web application in his or her local SharePoint
environment.

```PowerShell
$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function EnableAnonymousAccess(
    [Microsoft.SharePoint.SPWeb] $web = $(Throw "Value cannot be null: web"))
{
    Write-Host "Enabling anonymous access on site ($($web.Url))..."

    $anonymousPermissionMask =
        [Microsoft.SharePoint.SPBasePermissions]::Open `
        -bor [Microsoft.SharePoint.SPBasePermissions]::ViewFormPages `
        -bor [Microsoft.SharePoint.SPBasePermissions]::ViewListItems `
        -bor [Microsoft.SharePoint.SPBasePermissions]::ViewPages `
        -bor [Microsoft.SharePoint.SPBasePermissions]::ViewVersions

    If ($web.AnonymousPermMask64 -eq $anonymousPermissionMask)
    {
        Write-Host `
            "Anonymous access is already enabled on the site ($($web.Url))."

        return;
    }

    If ($web.HasUniqueRoleAssignments -eq $false)
    {
        $web.BreakRoleInheritance($true);
    }

    $web.AnonymousPermMask64 = $anonymousPermissionMask;
    $web.Update();

    Write-Host -Fore Green `
        "Successfully enabled anonymous access on site ($($web.Url))."
}

function Main()
{
    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    $webUrl = $webAppUrl + "/"
    $web = Get-SPWeb $webUrl -Debug:$false
    EnableAnonymousAccess $web
    $web.Dispose()
}

Main
```

### Configure Object Cache User Accounts.ps1

If you haven't yet discovered errors in the event log after creating Publishing
sites in SharePoint Server 2010, then you probably don't even bother to look at
your event logs. In that case, shame on you ;-)

If you _have_ seen the errors I'm referring to, then you're probably familiar
with the following TechNet article:

{{< reference title="Configure object cache user accounts"
linkHref="http://technet.microsoft.com/en-us/library/ff758656.aspx" >}}

Here's a script to get rid of those pesky errors. It assumes the Portal Super
User is **{DOMAIN}\svc-sp-psu** (or perhaps some variant depending on
environment, such as **{DOMAIN}\svc-sp-psu-dev**) and the Portal Super Reader is
**{DOMAIN}\svc-sp-psr** (or, again, some variant of this).

It takes care of adding the appropriate user policies on the Web application
(**Full Control** to **{DOMAIN}\svc-sp-psu,** and **Full Read** to
**{DOMAIN}\svc-sp-psr**), as well as setting the corresponding properties on the
Web application -- as described in the above TechNet article. It also ensures
the specified service accounts are indeed valid (via the `GetUserDisplayName`
function).

```PowerShell
$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function ConfigureObjectCacheUserAccounts(
    [string] $webAppUrl = $(Throw "Value cannot be null: webAppUrl"),
    [string] $portalSuperUserAccount =
        $(Throw "Value cannot be null: portalSuperUserAccount"),
    [string] $portalSuperReaderAccount =
        $(Throw "Value cannot be null: portalSuperReaderAccount"))
{
    Write-Host ("Configuring object cache user accounts for Web application" `
        + " ($webAppUrl)...")

    Write-Debug "portalSuperUserAccount: $portalSuperUserAccount"
    Write-Debug "portalSuperReaderAccount: $portalSuperReaderAccount"

    $webApp = Get-SPWebApplication -Identity $webAppUrl -Debug:$false

    SetWebAppUserPolicy $webApp $portalSuperUserAccount "Full Control"
    SetWebAppProperty $webApp "portalsuperuseraccount" $portalSuperUserAccount

    SetWebAppUserPolicy $webApp $portalSuperReaderAccount "Full Read"
    SetWebAppProperty $webApp "portalsuperreaderaccount" $portalSuperReaderAccount

    Write-Host -Fore Green ("Successfully configured object cache user accounts" `
        + " for Web application ($webAppUrl).")
}

function GetUserDisplayName(
    [string] $userName = $(Throw "Value cannot be null: userName"))
{
    Write-Debug "Getting display name for user ($userName)..."

    $userNameParts = $userName.Split("\")
    $samAccountName = $userNameParts[1]

    $filter = "(&(objectCategory=User)(samAccountName=$samAccountName))"

    $searcher = New-Object System.DirectoryServices.DirectorySearcher
    $searcher.Filter = $filter

    $path = $searcher.FindOne()

    If ($path -eq $null)
    {
        Throw "User not found ($userName)."
    }

    $user = $path.GetDirectoryEntry()

    Write-Debug "Found display name for user ($($user.DisplayName))."
    return $user.DisplayName
}

function SetWebAppProperty(
    [Microsoft.SharePoint.Administration.SPWebApplication] $webApp =
         $(Throw "Value cannot be null: webApp"),
    [string] $propertyName = $(Throw "Value cannot be null: propertyName"),
    [string] $propertyValue)
{
    Write-Debug ("Setting property ($propertyName) on Web application" `
        + " ($($webApp.Url)...")

    If ($webApp.Properties[$propertyName] -eq $propertyValue)
    {
        Write-Debug ("The Web application property ($propertyName) is already set" `
            + " to the expected value ($propertyValue).")

        return;
    }

    $webApp.Properties[$propertyName] = $propertyValue
    $webApp.Update()

    Write-Debug ("Successfully set property ($propertyName) on Web" `
        + " application ($($webApp.Url)) to '$propertyValue'.")

}

function SetWebAppUserPolicy(
    [Microsoft.SharePoint.Administration.SPWebApplication] $webApp =
        $(Throw "Value cannot be null: webApp"),
    [string] $userName = $(Throw "Value cannot be null: userName"),
    [string] $permissions = $(Throw "Value cannot be null: permissions"))
{
    Write-Debug ("Setting policy ($permissions) for user" `
        + " ($userName) on Web application ($($webApp.Url))...")

    [Microsoft.SharePoint.Administration.SPPolicyRole] $policyRole =
        $webApp.PolicyRoles | where {$_.Name -eq $permissions}

    if ($policyRole -eq $null)
    {
        Throw "Invalid permissions ($permissions)."
    }

    $userDisplayName = GetUserDisplayName $userName

    [Microsoft.SharePoint.Administration.SPPolicyCollection] $policies =
        $webApp.Policies

    [Microsoft.SharePoint.Administration.SPPolicy] $policy = $policies.Add(
        $userName,
        $userDisplayName)

    $policy.PolicyRoleBindings.Add($policyRole)
    $webApp.Update()

    Write-Debug ("Successfully added policy ($permissions) for user" `
        + " ($userName) to Web application ($($webApp.Url))...")
}

function Main()
{
    $portalSuperUserAccount = $env:USERDOMAIN + "\svc-sp-psu"
    $portalSuperReaderAccount = $env:USERDOMAIN + "\svc-sp-psr"

    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    Write-Debug "webAppUrl: $webAppUrl"

    if (($webAppUrl -eq "http://www-local.fabrikam.com") -or
        ($webAppUrl -eq "http://www-dev.fabrikam.com"))
    {
        Write-Debug "Overriding variables for Development environment..."
        $portalSuperUserAccount = $env:USERDOMAIN + "\svc-sp-psu-dev"
        $portalSuperReaderAccount = $env:USERDOMAIN + "\svc-sp-psr-dev"
    }
    elseif ($webAppUrl -eq "http://www-test.fabrikam.com")
    {
        Write-Debug "Overriding variables for Test environment..."
        $portalSuperUserAccount = $env:USERDOMAIN + "\svc-sp-psu-test"
        $portalSuperReaderAccount = $env:USERDOMAIN + "\svc-sp-psr-test"
    }

    ConfigureObjectCacheUserAccounts $webAppUrl $portalSuperUserAccount `
        $portalSuperReaderAccount

}

Main
```

### Add Event Log Sources.ps1

The Fabrikam solution includes a custom **SPLogger** class for writing trace
messages and events. In order to write to the Windows event log with a custom
source (e.g. "Fabrikam Demo Site"), you first need to create the event log
source (assuming your solution is running with a least-privileged service
account, and I certainly _hope_ it is). Otherwise, a nasty error will occur when
attempting to log an event. [If you solution is running with administrative
privileges, then the custom event log source will be dynamically created as
necessary -- but please don't do this. It's just plain wrong.]

The following script ensures the custom event source is registered.

{{< div-block "note important" >}}

> **Important**
>
> This script must be run on each SharePoint server in the farm.

{{< /div-block >}}

```PowerShell
$ErrorActionPreference = "Stop"

Function AddEventLogSource(
    [string] $source = $(Throw "Value cannot be null: source"))
{
    $source = $source.Trim()

    If ([string]::IsNullOrEmpty($source) -eq $true)
    {
        Throw "The name of the event source is required."
    }

    $sourceExists = [System.Diagnostics.EventLog]::SourceExists($source)
    If ($sourceExists -eq $true)
    {
        Write-Host "The event source ($source) already exists."
    }
    Else
    {
        Write-Host "Creating event source ($source)..."

        [System.Diagnostics.EventLog]::CreateEventSource($source, "Application")

        Write-Host -Fore Green "Successfully created event source ($source)."
    }
}

function Main
{
    AddEventLogSource "Fabrikam Demo Site"
}

Main
```

Although this script currently adds only one custom event log source, I named it
plural just in case additional sources are added in the future (so I wouldn't
have to update the script name in the Installation Guide for the Fabrikam site).

At this point, we have a "vanilla" Web site created and configured in SharePoint
Server 2010. Now we need to run the "ADA" portion of the "DR.DADA" process in
order to _Add_ our custom solution (Fabrikam.Demo.Web.wsp), _Deploy_ the
solution to the Fabrikam Web application, and, finally, _Activate_ our features.

### Add Solutions.ps1

Let's start with a script to add the solution (note that additional solutions
may be added in the future, so the script name is plural).

If you've been developing with SharePoint 2010 for any signficant period of
time, then you might have encountered issues due to "stale" assemblies being
used for event receivers (unless perhaps you simply do all of your "DR.DADA"
operations through Visual Studio and never through PowerShell). This is because
your assemblies are actually loaded into the PowerShell process during various
deployment operations (for example, when you invoke the `Add-SPSolution`
cmdlet). I discovered this issue the "hard way" -- in other words, by attaching
WinDbg to my PowerShell command prompt.

When I searched the Internet for _**PowerShell reload assembly**_ (looking for
an easy way to unload a specific assembly), I discovered the following blog
post:

{{< reference title="PowerShell Does Not Reload Upgraded Assemblies"
linkHref="http://www.sharepointblues.com/2010/09/06/powershell-does-not-reload-upgraded-assemblies" >}}

I found Lauri's approach to be invaluable for isolating the AppDomain that loads
your assemblies, as shown in the following script.

```PowerShell
param(
    [switch] $runInThisAppDomain,
    [switch] $debug)

# When a solution is added to SharePoint, the corresponding assembly is loaded
# into the PowerShell AppDomain (along with any referenced assemblies). This can
# be seen by attaching to the PowerShell process with WinDbg and viewing modules
# as they are loaded.
#
# In order to avoid issues during the deployment, force the script to run in a new
# instance of PowerShell (and thus a new AppDomain). This is accomplished using a
# technique originally noted by Lauri Perltonen
# (http://www.sharepointblues.com/2010/09/06/powershell-does-not-reload-upgraded-assemblies).
If (-not $runInThisAppDomain)
{
    Write-Host -Fore Yellow "Invoking script in a new app domain"
    PowerShell.exe -Command $MyInvocation.Line -RunInThisAppDomain
    return
}

# If "-Debug" option is specified, enable debug messages in new PowerShell instance
If ($debug -eq $true)
{
    $DebugPreference = "Continue"
}

$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function AddSolution(
    [string] $solutionPath = $(Throw "Value cannot be null: solutionPath"))
{
    Write-Host "Adding solution ($solutionPath)..."

    Resolve-Path $solutionPath | Add-SPSolution -Debug:$false > $null

    Write-Host -Fore Green "Successfully added solution ($solutionPath)."
}

function Main()
{
    Write-Host "Adding solutions..."

    $buildConfiguration = $env:FABRIKAM_BUILD_CONFIGURATION
    If ($buildConfiguration -eq $null)
    {
        $buildConfiguration = "Release"
    }

	Write-Debug "buildConfiguration: $buildConfiguration"

    # For desktop builds, the WSP is created in the
    # ..\..\Web\bin\{Debug|Release} folder.
    # However, with Team Foundation Build, the WSP is created in the
    # $(BinariesRoot)\{Debug|Release} folder.
    $solutionFile = "..\..\Web\bin\$buildConfiguration\Fabrikam.Demo.Web.wsp"
    If ((Test-Path $solutionFile) -eq $false)
    {
        $solutionFile = "..\..\$buildConfiguration\Fabrikam.Demo.Web.wsp"
    }

    AddSolution $solutionFile
}

Main
```

{{< div-block "note update" >}}

> **Update (2011-03-02)**
>
> I modified the original script above to ensure it works with TFS builds as
> well as "desktop" builds (i.e. built from within Visual Studio).

{{< /div-block >}}

The optional `-Debug` parameter for this script makes it easy to debug the new
PowerShell instance (without requiring you to temporarily specify
`$DebugPreference = "Continue"` in your PowerShell profile).

Note how I allow the build configuration (Debug or Release) to be specified
outside of the script depending on the environment (i.e. using the
FABRIKAM\_BUILD\_CONFIGURATION environment variable). For example, in the
Production environment, we always want to deploy Release builds -- not Debug
builds. Whereas in development environments, we always want to install the Debug
builds. (Whether we install Debug or Release builds to the Test environment
depends on where we are at in the release cycle.)

### Deploy Solutions.ps1

After adding the solutions to SharePoint, we next need to deploy them to the Web
application. When a solution is deployed, SharePoint extracts the contents of
the CAB file -- er, I mean WSP file -- including the assembly and any files
deployed to the "14 hive", and copies these to the appropriate locations (such
as the Global Assembly Cache or a folder under C:\Program Files\Common
Files\Microsoft Shared\Web Server Extensions\14).

Note that in a SharePoint farm comprised of multiple servers, this deployment
must be done on each server in the farm. No, you don't execute the PowerShell
script on each SharePoint server in the farm. Rather, when you run the script on
_one_ of the servers in the farm, SharePoint automatically creates a timer job
on _each_ server in the farm to deploy the solution on that server.

```PowerShell
param(
    [switch] $force,
    [switch] $runInThisAppDomain,
    [switch] $debug)

# When a solution is deployed to SharePoint, the corresponding assembly is loaded
# into the PowerShell AppDomain (along with any referenced assemblies). This can
# be seen by attaching to the PowerShell process with WinDbg and viewing modules
# as they are loaded.
#
# In order to avoid issues during the deployment, force the script to run in a new
# instance of PowerShell (and thus a new AppDomain). This is accomplished using a
# technique originally noted by Lauri Perltonen
# (http://www.sharepointblues.com/2010/09/06/powershell-does-not-reload-upgraded-assemblies).
If (-not $runInThisAppDomain)
{
    Write-Host -Fore Yellow "Invoking script in a new app domain"
    PowerShell.exe -Command $MyInvocation.Line -RunInThisAppDomain
    return
}

# If "-Debug" option is specified, enable debug messages in new PowerShell instance
If ($debug -eq $true)
{
    $DebugPreference = "Continue"
}

$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function DeploySolution(
    [string] $solutionName = $(Throw "Value cannot be null: solutionName"),
    [string] $webAppUrl = $(Throw "Value cannot be null: webAppUrl"),
    [bool] $force,
    [bool] $local)
{
    Write-Host ("Deploying solution ($solutionName) to Web application" `
        + " ($webAppUrl)...")

    If ($force -eq $true)
    {
        Write-Debug "The solution deployment will be forced."
    }

    If ($local -eq $true)
    {
        Write-Debug ("The solution will be deployed locally (bypassing" `
            + " SharePoint timer job).")
    }

    Install-SPSolution $solutionName -GACDeployment `
        -WebApplication $webAppUrl -Force:([bool]::Parse($force)) `
        -Local:([bool]::Parse($local)) -Confirm:$false -Debug:$false

    # If the deployment was performed using a SharePoint timer job, then wait
    # for the timer job to finish
    If (-not $local)
    {
    	. '.\Wait for Solution Deployment Jobs to Finish.ps1' $solutionName
    }

    Write-Host -Fore Green ("Successfully deployed solution ($solutionName)" `
        + " to Web application ($webAppUrl).")
}

function Main(
    [bool] $force)
{
    Write-Host "Deploying solutions..."

    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    [bool] $local = $false

    If (($webAppUrl -eq "http://www-local.fabrikam.com") `
    	-or ($webAppUrl -eq "http://www-dev.fabrikam.com"))
    {
        $local = $true
    }

    DeploySolution "Fabrikam.Demo.Web.wsp" $webAppUrl $force $local
}

Main $force
```

Also note that the script supports an optional parameter to force the solution
to be deployed -- in the (hopefully) rare event you need to use that.

The most interesting part about **Deploy Solutions.ps1** is the fact that I try
to avoid using a SharePoint Timer job to deploy the solution if at all possible.
In other words, on my local development VM or in the Development integration
environment, there's no need to schedule the deployment through the SharePoint
Timer infrastructure since it's just a single server environment. This is
another great reason to follow
[a standard naming convention for your environments](/blog/jjameson/2009/06/09/environment-naming-conventions).

If the solution is not deployed with the "`-Local`" option, then we need to wait
for the solution deployment job to finish before we can activate the features.
Back in the MOSS 2007 days, I used `stsadm.exe -o execadmsvcjobs` to wait for
the timer job to finish before continuing. However, since the StsAdm.exe utility
is considered "pass&eacute;" in SharePoint 2010, I now use a PowerShell script
instead.

### Wait for Solution Deployment Jobs to Finish.ps1

Here's a script that allows you to wait for either specific solution deployment
jobs to finish (if one or more solution names are specified as parameters to the
script) or to wait for _all_ solution deployment jobs to finish (if no
parameters are specified when running the script).

```PowerShell
$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function WaitForSharePointTimerJobToFinish(
	[Microsoft.SharePoint.Administration.SPJobDefinition] $job)
{
    If ($job -eq $null)
    {
        return
    }

    $jobName = $job.Name

    Write-Host -NoNewLine ("Waiting for SharePoint timer job ($jobName) to" `
        + " finish...")

    While ((Get-SPTimerJob $jobName -Debug:$false) -ne $null)
    {
        Write-Host -NoNewLine "."
        Start-Sleep -Seconds 5
    }

	Write-Host

    Write-Host "The SharePoint timer job ($jobName) has finished."
}

function WaitForSolutionDeploymentJobsToFinish(
	[string] $solutionName)
{
    Write-Debug "solutionName: $solutionName"

    If ([string]::IsNullOrEmpty($solutionName) -eq $true)
    {
        Write-Debug "Waiting for all solution deployment jobs to finish..."
        $jobNameFilter = "*solution-deployment*"
    }
    Else
    {
        Write-Debug ("Waiting for solution deployment " `
            + " ($solutionName) to finish...")

        $jobNameFilter = "*solution-deployment*$solutionName*"
    }

    Write-Debug "jobNameFilter: $jobNameFilter"

    $jobs = Get-SPTimerJob -Debug:$false  | Where { $_.Name -like $jobNameFilter }

    If ($jobs -eq $null)
    {
        Write-Debug "No solution deployment jobs found"
        return
    }

    If ($jobs -is [Array])
    {
        Foreach ($job in $jobs)
        {
            WaitForSharePointTimerJobToFinish $job
        }
    }
    Else
    {
        WaitForSharePointTimerJobToFinish $jobs
    }
}

function Main()
{
    If ($args.Count -eq 0)
    {
        WaitForSolutionDeploymentJobsToFinish
    }
    Else
    {
        Foreach ($solutionName in $args)
        {
            WaitForSolutionDeploymentJobsToFinish $solutionName
        }
    }
}

Main $args
```

### Activate Features.ps1

With our custom WSP deployed, we are now ready to activate the features. This is
easy enough to do using the `Enable-SPFeature` cmdlet, but the following script
makes this a little more robust. For example, it first determines the scope of
each activated feature and then checks to see if the feature is already
activated at the corresponding scope. This avoids annoying errors like "The
feature ... is already activated..." that terminate the script.

Also note that, like **Deploy Solutions.ps1**, the following script supports an
optional parameter to force the features to be activated.

```PowerShell
param(
    [switch] $force,
    [switch] $runInThisAppDomain,
    [switch] $debug)

# When a feature is activated in SharePoint, the corresponding assembly is loaded
# into the PowerShell AppDomain (along with any referenced assemblies). This can
# be seen by attaching to the PowerShell process with WinDbg and viewing modules
# as they are loaded.
#
# In order to avoid issues during the deployment, force the script to run in a new
# instance of PowerShell (and thus a new AppDomain). This is accomplished using a
# technique originally noted by Lauri Perltonen
# (http://www.sharepointblues.com/2010/09/06/powershell-does-not-reload-upgraded-assemblies).
If (-not $runInThisAppDomain)
{
    Write-Host -Fore Yellow "Invoking script in a new app domain"
    PowerShell.exe -Command $MyInvocation.Line -RunInThisAppDomain
    return
}

# If "-Debug" option is specified, enable debug messages in new PowerShell instance
If ($debug -eq $true)
{
    $DebugPreference = "Continue"
}

$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function ActivateFeature(
    [string] $featureName = $(Throw "Value cannot be null: featureName"),
    [string] $siteUrl = "",
    [bool] $force = $false)
{
    Write-Debug "Activating feature ($featureName)..."
    Write-Debug "featureName: $featureName"

    If ($force -eq $true)
    {
        Write-Debug "The feature activation will be forced."
    }

    $feature = Get-SPFeature $featureName -Debug:$false

    If ($feature.Scope -eq [Microsoft.SharePoint.SPFeatureScope]::Farm)
    {
        $feature = Get-SPFeature $featureName -Farm -Debug:$false -EA 0

        If ($feature -ne $null -and ($force -eq $false))
        {
            Write-Host "The feature ($featureName) is already activated on the farm."
            return;
        }

        Write-Host "Activating feature ($featureName) on farm..."

        Enable-SPFeature $featureName -Force:([bool]::Parse($force)) `
            -Confirm:$false -Debug:$false

        Write-Host -Fore Green ("Successfully activated farm feature" `
            + " ($featureName).")

        return
    }
    ElseIf ($feature.Scope -eq `
        [Microsoft.SharePoint.SPFeatureScope]::WebApplication)
    {
        $feature = Get-SPFeature $featureName -WebApplication $siteUrl `
            -Debug:$false -EA 0
    }
    ElseIf ($feature.Scope -eq [Microsoft.SharePoint.SPFeatureScope]::Site)
    {
        $feature = Get-SPFeature $featureName -Site $siteUrl -Debug:$false -EA 0
    }
    ElseIf ($feature.Scope -eq [Microsoft.SharePoint.SPFeatureScope]::Web)
    {
        $feature = Get-SPFeature $featureName -Web $siteUrl -Debug:$false -EA 0
    }

    If ($feature -ne $null -and ($force -eq $false))
    {
        Write-Host ("The feature ($featureName) is already activated on the site" `
            + " ($siteUrl)...")

        return;
    }

    Write-Host "Activating feature ($featureName) on site ($siteUrl)..."

    Enable-SPFeature $featureName -Url $siteUrl `
        -Force:([bool]::Parse($force)) -Confirm:$false -Debug:$false

    Write-Host -Fore Green ("Successfully activated feature ($featureName) on" `
        + " site ($siteUrl).")
}

function Main()
{
    Write-Host "Activating features..."

    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    $siteUrl = $webAppUrl + "/"

    ActivateFeature "Fabrikam.Demo.Web_WebAppConfiguration" $siteUrl $force
    ActivateFeature "Fabrikam.Demo.Web_WebParts" $siteUrl $force
    ActivateFeature "Fabrikam.Demo.Web_HomeSiteConfiguration" $siteUrl $force
}

Main
```

At this point, the Fabrikam site is fully configured and ready for testing.

Now, imagine that we've fixed some bugs or modified our custom SharePoint
solution (for example, to add a custom master page, or some new page layouts).
Consequently, we need to "DRD" the old solution and "ADA" the new version.

Let's start by deactivating the features...

### Deactivate Features.ps1

If you've carefully examined the **Activate Features.ps1** script, then there's
really no point in scrutinizing the following script ;-)

```PowerShell
param(
    [switch] $force,
    [switch] $runInThisAppDomain,
    [switch] $debug)

# When a feature is deactivated in SharePoint, the corresponding assembly is loaded
# into the PowerShell AppDomain (along with any referenced assemblies). This can
# be seen by attaching to the PowerShell process with WinDbg and viewing modules
# as they are loaded.
#
# In order to avoid issues during the deployment, force the script to run in a new
# instance of PowerShell (and thus a new AppDomain). This is accomplished using a
# technique originally noted by Lauri Perltonen
# (http://www.sharepointblues.com/2010/09/06/powershell-does-not-reload-upgraded-assemblies).
If (-not $runInThisAppDomain)
{
    Write-Host -Fore Yellow "Invoking script in a new app domain"
    PowerShell.exe -Command $MyInvocation.Line -RunInThisAppDomain
    return
}

# If "-Debug" option is specified, enable debug messages in new PowerShell instance
If ($debug -eq $true)
{
    $DebugPreference = "Continue"
}

$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function DeactivateFeature(
    [string] $featureName = $(Throw "Value cannot be null: featureName"),
    [string] $siteUrl = "",
    [bool] $force = $false)
{
    Write-Debug "Deactivating feature ($featureName)..."
    Write-Debug "siteUrl: $siteUrl"

    If ($force -eq $true)
    {
        Write-Debug "The feature deactivation will be forced."
    }

    $feature = Get-SPFeature $featureName -Debug:$false -EA 0

    If ($feature -eq $null)
    {
        Write-Warning "The specified feature ($featureName) was not found."
        return
    }

    If ($feature.Scope -eq [Microsoft.SharePoint.SPFeatureScope]::Farm)
    {
        $feature = Get-SPFeature $featureName -Farm -Debug:$false -EA 0

        If ($feature -eq $null)
        {
            Write-Host "The feature ($featureName) is not activated on the farm."
            return
        }

        Write-Host "Deactivating farm feature ($featureName)..."

        Disable-SPFeature $featureName -Force:([bool]::Parse($force)) `
            -Confirm:$false -Debug:$false

        Write-Host -Fore Green ("Successfully deactivated farm feature" `
            + " ($featureName).")

        return
    }
    ElseIf ($feature.Scope -eq `
        [Microsoft.SharePoint.SPFeatureScope]::WebApplication)
    {
        $feature = Get-SPFeature $featureName -WebApplication $siteUrl `
            -Debug:$false -EA 0
    }
    ElseIf ($feature.Scope -eq [Microsoft.SharePoint.SPFeatureScope]::Site)
    {
        $feature = Get-SPFeature $featureName -Site $siteUrl -Debug:$false -EA 0
    }
    ElseIf ($feature.Scope -eq [Microsoft.SharePoint.SPFeatureScope]::Web)
    {
        $feature = Get-SPFeature $featureName -Web $siteUrl -Debug:$false -EA 0
    }

    If ($feature -eq $null)
    {
        Write-Host ("The feature ($featureName) is not activated on the site" `
            + " ($siteUrl)...")

        return
    }

    Write-Host "Deactivating feature ($featureName) on site ($siteUrl)..."

    Disable-SPFeature $featureName -Url $siteUrl -Force:([bool]::Parse($force)) `
        -Confirm:$false -Debug:$false

    Write-Host -Fore Green ("Successfully deactivated feature ($featureName) on" `
        + " site ($siteUrl).")
}

function Main(
    [bool] $force)
{
    Write-Host "Deactivating features..."

    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    $siteUrl = $webAppUrl + "/"

    DeactivateFeature "Fabrikam.Demo.Web_HomeSiteConfiguration" $siteUrl $force
    DeactivateFeature "Fabrikam.Demo.Web_WebParts" $siteUrl $force
	DeactivateFeature "Fabrikam.Demo.Web_WebAppConfiguration" $siteUrl $force
}

Main $force
```

### Retract Solutions.ps1

After the features are deactivated, we are ready to retract the solution from
the Web application. Like the **Deploy Solutions.ps1** script, I try to avoid
SharePoint timer jobs -- if possible -- so that developers can be as productive
as possible.

```PowerShell
param(
    [switch] $runInThisAppDomain,
    [switch] $debug)

# When a solution is retracted from SharePoint, the corresponding assembly is loaded
# into the PowerShell AppDomain (along with any referenced assemblies). This can
# be seen by attaching to the PowerShell process with WinDbg and viewing modules
# as they are loaded.
#
# In order to avoid issues during the deployment, force the script to run in a new
# instance of PowerShell (and thus a new AppDomain). This is accomplished using a
# technique originally noted by Lauri Perltonen
# (http://www.sharepointblues.com/2010/09/06/powershell-does-not-reload-upgraded-assemblies).
If (-not $runInThisAppDomain)
{
    Write-Host -Fore Yellow "Invoking script in a new app domain"
    PowerShell.exe -Command $MyInvocation.Line -RunInThisAppDomain
    return
}

# If "-Debug" option is specified, enable debug messages in new PowerShell instance
If ($debug -eq $true)
{
    $DebugPreference = "Continue"
}

$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function RetractSolution(
    [string] $solutionName = $(Throw "Value cannot be null: solutionName"),
    [string] $webAppUrl = $(Throw "Value cannot be null: webAppUrl"),
    [bool] $local)
{
    Write-Host ("Retracting solution ($solutionName) from Web application" `
        + " ($webAppUrl)...")

    If ($local -eq $true)
    {
        Write-Debug ("The solution will be retracted locally (bypassing" `
            + " SharePoint timer job).")
    }

    $webApp = Get-SPWebApplication $webAppUrl -Debug:$false

    $solution = Get-SPSolution $solutionName -Debug:$false -EA 0

    If ($solution -eq $null)
    {
        Write-Warning "The specified solution ($solutionName) was not found."
        return
    }

    $deployedWebApp = $solution.DeployedWebApplications |
        Where { $_.Url -eq $webApp.Url }

    If ($deployedWebApp -eq $null)
    {
        Write-Host ("The solution ($solutionName) is not deployed to the" `
            + " specified Web application ($webAppUrl).")

        return;
    }

    Uninstall-SPSolution $solutionName -WebApplication $webAppUrl `
    	-Local:([bool]::Parse($local)) -Confirm:$false -Debug:$false

    # If the retraction was performed using a SharePoint timer job, then wait
    # for the timer job to finish
    If ($local -eq $false)
    {
    	. '.\Wait for Solution Deployment Jobs to Finish.ps1' $solutionName
    }

    Write-Host -Fore Green ("Successfully retracted solution ($solutionName)" `
        + " from Web application ($webAppUrl).")
}

function Main()
{
    Write-Host "Retracting solutions..."

    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    $local = $false

    If (($webAppUrl -eq "http://www-local.fabrikam.com") `
    	-or ($webAppUrl -eq "http://www-dev.fabrikam.com"))
    {
        $local = $true
    }

    RetractSolution "Fabrikam.Demo.Web.wsp" $webAppUrl $local
}

Main
```

### Delete Solutions.ps1

Lastly, it's time to delete the old solution from the SharePoint farm...

```PowerShell
# When a solution is deleted from SharePoint, the corresponding assembly is
# *not* loaded into the PowerShell AppDomain. Consequently, there is no reason
# to start a new PowerShell instance when running this script (unlike the
# "Add", "Deploy", "Activate", "Deactivate", "Retract", and "Upgrade" scripts).

$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function DeleteSolution(
    [string] $solutionName = $(Throw "Value cannot be null: solutionName"))
{
    Write-Host "Deleting solution ($solutionName)..."

    $solution = Get-SPSolution $solutionName -EA 0 -Debug:$false

    If ($solution -eq $null)
    {
        Write-Warning "The specified solution ($solutionName) was not found."
        return
    }

    Remove-SPSolution $solutionName -Confirm:$false -Debug:$false

    Write-Host -Fore Green "Successfully deleted solution ($solutionName)."
}

function Main()
{
    Write-Host "Deleting solutions..."

    DeleteSolution "Fabrikam.Demo.Web.wsp"
}

Main
```

At this point, we are ready to "ADA" the updated Fabrikam.Demo.Web.wsp solution.

### Redeploy Features.ps1

If, like me, you get tired of cycling through the command history (F7) to
repeatedly execute the "DR.DADA" process, then you can use the following script
to save a few dozen keystrokes.

```PowerShell
$ErrorActionPreference = "Stop"

function Main()
{
    & '.\Deactivate Features.ps1'

    & '.\Retract Solutions.ps1'

    & '.\Delete Solutions.ps1'

    & '.\Add Solutions.ps1'

    & '.\Deploy Solutions.ps1'

    & '.\Activate Features.ps1'
}

Main
```

This script essentially performs the same activities as the **Default**
deployment configuration for a SharePoint project in Visual Studio.
Consequently, I don't expect this to be used all that much during the
development process. I've still found this useful, however, for some scenarios.
For example, in the sample SharePoint solution I provided in
[my previous post](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010),
you'll find that I changed the **Activate On Default** property of the
"WebAppConfiguration" feature to **False**. For the reasons why I did this,
refer to the following post (that was originally written for MOSS 2007 but still
applies to SharePoint 2010):

{{< reference title="SharePoint Features Activated by Default"
linkHref="/blog/jjameson/2010/03/31/sharepoint-features-activated-by-default"
linkText="https://www.technologytoolbox.com/blog/jjameson/2010/03/31/sharepoint-features-activated-by-default" >}}

Consequently, I used the **Redeploy Features.ps1** script to ensure the
WebAppConfiguration feature is activated (thus ensuring my custom Sign In page
for claims authentication was configured on the Web application).

### Upgrade Solutions.ps1

While the "DR.DADA" process doesn't typically take very long, there are a
limited number of changes that can be made to a SharePoint solution in which a
simple "Upgrade Solution" will suffice. Here's a script that performs the
equivalent of the old {{< kbd "stsadm.exe -o upgradesolution" >}} command.

```PowerShell
param(
    [switch] $force,
    [switch] $runInThisAppDomain,
    [switch] $debug)

# When a solution is upgraded in SharePoint, the corresponding assembly is loaded
# into the PowerShell AppDomain (along with any referenced assemblies). This can
# be seen by attaching to the PowerShell process with WinDbg and viewing modules
# as they are loaded.
#
# In order to avoid issues during the deployment, force the script to run in a new
# instance of PowerShell (and thus a new AppDomain). This is accomplished using a
# technique originally noted by Lauri Perltonen
# (http://www.sharepointblues.com/2010/09/06/powershell-does-not-reload-upgraded-assemblies).
If (-not $runInThisAppDomain)
{
    Write-Host -Fore Yellow "Invoking script in a new app domain"
    PowerShell.exe -Command $MyInvocation.Line -RunInThisAppDomain
    return
}

# If "-Debug" option is specified, enable debug messages in new PowerShell instance
If ($debug -eq $true)
{
    $DebugPreference = "Continue"
}

$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function UpgradeSolution(
    [string] $solutionPath = $(Throw "Value cannot be null: solutionPath"),
    [bool] $force,
    [bool] $local)
{
    Write-Host "Upgrading solution ($solutionPath)..."

    If ($force -eq $true)
    {
        Write-Debug "The solution upgrade will be forced."
    }

    If ($local -eq $true)
    {
        Write-Debug ("The solution will be upgraded locally (bypassing" `
            + " SharePoint timer job).")
    }

    $literalPath = Resolve-Path $solutionPath
    $solutionFile = Get-Item $literalPath

    Update-SPSolution $solutionFile.Name -LiteralPath $literalPath -GACDeployment `
        -Force:([bool]::Parse($force)) -Local:([bool]::Parse($local)) `
        -Debug:$false > $null

    # If the deployment was performed using a SharePoint timer job, then wait
    # for the timer job to finish
    If ($local -eq $false)
    {
    	. '.\Wait for Solution Deployment Jobs to Finish.ps1' $solutionFile.Name
    }

    Write-Host -Fore Green "Successfully upgraded solution ($solutionPath)."
}

function Main(
    [bool] $force)
{
    Write-Host "Upgrading solutions..."

    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    $local = $false

    If (($webAppUrl -eq "http://www-local.fabrikam.com") `
    	-or ($webAppUrl -eq "http://www-dev.fabrikam.com"))
    {
        $local = $true
    }

    $buildConfiguration = $env:FABRIKAM_BUILD_CONFIGURATION
    If ($buildConfiguration -eq $null)
    {
        $buildConfiguration = "Release"
    }

	Write-Debug "buildConfiguration: $buildConfiguration"

    # For desktop builds, the WSP is created in the
    # ..\..\Web\bin\{Debug|Release} folder.
    # However, with Team Foundation Build, the WSP is created in the
    # $(BinariesRoot)\{Debug|Release} folder.
    $solutionFile = "..\..\Web\bin\$buildConfiguration\Fabrikam.Demo.Web.wsp"
    If ((Test-Path $solutionFile) -eq $false)
    {
        $solutionFile = "..\..\$buildConfiguration\Fabrikam.Demo.Web.wsp"
    }

    UpgradeSolution $solutionFile $force $local
}

Main $force
```

{{< div-block "note update" >}}

> **Update (2011-03-02)**
>
> I modified the original script above to ensure it works with TFS builds as
> well as "desktop" builds (i.e. built from within Visual Studio).

{{< /div-block >}}

### Delete Web Application.ps1

On my local development VM, I often find it helpful to "nuke" the Fabrikam Web
application and start over from scratch. To achieve this, I "DRD" the Fabrikam
solution/features and then run the following script and start over with the
**Create Web Application.ps1** script.

```PowerShell
$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function Main()
{
    $webAppUrl = $env:FABRIKAM_DEMO_URL
    If ($webAppUrl -eq $null)
    {
        $webAppUrl = "http://www.fabrikam.com"
    }

    Write-Debug "webAppUrl: $webAppUrl"

    Remove-SPWebApplication $webAppUrl -DeleteIISSite -RemoveContentDatabases `
        -Debug:$false
}

Main
```

In fact, on several occasions I've found it to be _very_ helpful to rollback my
SharePoint 2010 Hyper-V VM to a snapshot that I took shortly after installing
SharePoint Server 2010 and creating the farm (but before creating any Web
applications or configuring any service applications). Whenever I do that, I
simply need to force a "Get Latest" from TFS (since my TFS workspace isn't aware
that I've rolled back my VHD to an earlier point in time), compile and package
the solution, and then run the scripts described above in the following order:

{{< console-block-start >}}

& '.\Create Web Application.ps1'\
& '.\Create Site Collections.ps1'\
& '.\Enable Anonymous Access.ps1'\
& '.\Configure Object Cache User Accounts.ps1'\
& '.\Add Event Log Sources.ps1'\
& '.\Add Solutions.ps1'\
& '.\Deploy Solutions.ps1'\
& '.\Activate Features.ps1'

{{< console-block-end >}}

Thanks to the extremely robust scripting capabilities in SharePoint 2010, I'm
able to rebuild my development environment in a matter of minutes.

{{< div-block "note warning" >}}

> **Warning**
>
> If, like me, you decide to use Hyper-V snapshots in your SharePoint
> development environment, then make darn sure you've checked in any pending
> changes to TFS (or at least shelved your changes) before you apply a snapshot.
> [Note that when applying an earlier snapshot, I don't typically take a new
> snapshot before reverting to the earlier point in time. In other words, I
> treat my SharePoint development VM as "volatile" -- or "disposable" (if you
> prefer that term instead).]

{{< /div-block >}}
