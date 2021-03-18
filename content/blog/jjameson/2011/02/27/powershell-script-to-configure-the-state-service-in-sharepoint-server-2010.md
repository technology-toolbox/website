---
title: PowerShell Script to Configure the State Service in SharePoint Server 2010
date: 2011-02-27T14:24:00-07:00
excerpt:
  In my first post today , I provided a number of scripts for deploying custom
  solutions and features in SharePoint Server 2010. However, those certainly
  weren't all of the PowerShell scripts that I currently use when working with
  SharePoint 2010. Here...
aliases:
  [
    "/blog/jjameson/archive/2011/02/27/powershell-script-to-configure-the-state-service-in-sharepoint-server-2010.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["SharePoint 2010", "PowerShell"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/02/27/powershell-script-to-configure-the-state-service-in-sharepoint-server-2010.aspx"
---

In
[my first post today](/blog/jjameson/2011/02/27/deployment-scripts-for-sharepoint-server-2010),
I provided a number of scripts for deploying custom solutions and features in
SharePoint Server 2010. However, those certainly weren't *all* of the PowerShell
scripts that I currently use when working with SharePoint 2010. Here's another
one that I deliberately excluded from the previous post.

Note that in any "real world" deployment of SharePoint Server 2010, you're going
to want to configure the State Service (or else you won't be able to use any
features that depend on InfoPath Forms Services -- including the out-of-the-box
approval workflow).

While you are certainly welcome to use Central Administration and/or the Farm
Configuration Wizard to configure the **State Service** service application,
personally I don't recommend it. Why not? Simple...you end up with one of those
nasty database names with a GUID in it.

It's not that I put heroic effort into avoiding database names with GUIDs in
them. After all, if you follow the instructions in one of the installation
guides I've written for various SharePoint projects, then you'll find that
PSConfig.exe creates the SharePoint\_AdminContent\_{GUID} database for Central
Administration. Personally, that one doesn't bother me (much) -- probably
because the amount of SQL traffic to that database is negligible. However, I
can't say the same for the numerous other SharePoint databases.

So, rather than having Central Administration create a StateService\_{GUID}
database, why not use a little bit of PowerShell instead? Plus, this saves you a
few clicks whenever you build out a new environment -- or rebuild an existing
(typically development) environment.

Here's a simple script to do just that.

### Configure State Service.ps1

```
$ErrorActionPreference = "Stop"

Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

# The State Service is required in order to use the out-of-the-box workflows in
# SharePoint Server 2010 (e.g. Approval - SharePoint 2010) or any other features
# that leverage InfoPath Forms Services.
#
# When using the Farm Configuration Wizard to configure the State Service, the
# resulting database is named StateService_{GUID}. In order to avoid lengthy
# database names containing GUIDs, the State Service is configured using PowerShell.

function ConfigureStateService(
    [string] $stateServiceName = $(Throw "Value cannot be null: stateServiceName"),
    [string] $stateServiceDatabaseName =
        $(Throw "Value cannot be null: stateServiceDatabaseName"))
{
    Write-Host "Configuring the State Service..."

    Write-Debug "stateServiceName: $stateServiceName"
    Write-Debug "stateServiceDatabaseName: $stateServiceDatabaseName"

    $serviceApp = Get-SPStateServiceApplication -Identity "$stateServiceName" `
        -Debug:$false -EA 0

    If ($serviceApp -ne $null)
    {
        Write-Host "The State Service has already been configured."
        return
    }

    $database = New-SPStateServiceDatabase -Name $stateServiceDatabaseName `
        -Debug:$false

    $serviceApp = New-SPStateServiceApplication -Name $stateServiceName `
        -Database $database -Debug:$false

    New-SPStateServiceApplicationProxy -ServiceApplication $serviceApp `
        -Name $stateServiceName -DefaultProxyGroup -Debug:$false > $null

	Write-Host -Fore Green "Successfully configured the State Service."
}

function Main()
{
    $stateServiceName = "State Service"
    $stateServiceDatabaseName = "StateService"

    ConfigureStateService $stateServiceName $stateServiceDatabaseName
}

Main
```

Obviously the real work here is performed by the actual SharePoint cmdlets, like
`New-SPStateServiceDatabase` and `New-SPStateServiceApplication`, and while you
could certainly type those PowerShell commands each time you need to configure
the State Service, I find this script much more convenient.

Thanks to [Todd Carter](http://www.todd-carter.com/) for providing the core
PowerShell commands used in the script above. While apparently the
"[wizard likes his GUIDs](http://todd-carter.com/post/2010/04/26/The-Wizard-Likes-His-GUIDs.aspx)"...Jeremy
most certainly does not (at least not when it comes to database names).

Seriously, why doesn't SharePoint use a database name like "StateService" by
default -- but if it determines that the desired database name is already in
use, then -- and only then -- it appends a GUID in order to avoid a naming
collision? I'm guessing 99.999% of the time there is only going to be one
StateService database per SharePoint farm. [If you think I'm wrong, please tell
me, because apparently I'm missing something.]
