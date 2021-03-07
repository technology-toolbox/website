---
title: "Upgrading TFS 2005/2008 Project Sites to TFS 2010, Part 2 - Team Wiki"
date: 2010-05-17T05:41:00-06:00
excerpt: "In part 1 of this series , I showed how you can add new Team Foundation Server (TFS) 2010 dashboard functionality to project sites originally created in TFS 2008 (or TFS 2005). 
 Another feature that you might want to add to upgraded project sites is..."
aliases: ["/blog/jjameson/archive/2010/05/16/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-2-team-wiki.aspx", "/blog/jjameson/archive/2010/05/17/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-2-team-wiki.aspx"]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "WSS v3", "TFS", "SharePoint 2010", "PowerShell"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/17/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-2-team-wiki.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/17/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-2-team-wiki.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In
[part 1 of this series](/blog/jjameson/2010/05/14/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-1-agile-dashboard-features),
I showed how you can add new Team Foundation Server (TFS) 2010 dashboard
functionality to project sites originally created in TFS 2008 (or TFS 2005).

Another feature that you might want to add to upgraded project sites is a "Team
Wiki" library. Wikis provide a great way to quickly share information about
various aspects of your project on the site.

Note that wiki libraries are not a feature of TFS, but are inherently available
in Windows SharePoint Services (and therefore Microsoft Office SharePoint Server
(MOSS) 2007). Consequently, you might very well be leveraging wikis already on
your TFS project sites.

If you haven't already added a wiki library, you can quickly create one using
out-of-the-box SharePoint functionality.

However, what if you want to add a wiki library to a number of project sites all
at once?

If you are using SharePoint Server 2010 (or SharePoint Foundation), then you can
easily use PowerShell to add a wiki library:

```
# Adds a "Wiki" library to a SharePoint site

function AddWikiLibrary(
    [Microsoft.SharePoint.SPweb] $web,
    $libraryName,
    $libraryDescription)
{
    Write-Debug "Adding wiki library ($libraryName) to site ($($web.Url))..."

    $listId = $web.Lists.Add(
        $name,
        $description,
        [Microsoft.SharePoint.SPListTemplateType]::WebPageLibrary)

    $wikiLibrary = $web.Lists[$listId]

    $null = [Microsoft.SharePoint.Utilities.SPUtility]::AddDefaultWikiContent(
        $wikiLibrary)
}

$name = "Team Wiki"

$description = "Share knowledge for a Team Project by adding or editing" `
    + " content in this wiki"

$DebugPreference = "SilentlyContinue"
$web = Get-SPWeb "http://cyclops/sites/AdventureWorks"

$DebugPreference = "Continue"
AddWikiLibrary $web $name $description
```

Note that in SharePoint, you create a new list (or library) using the **
[SPListCollection.Add](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.splistcollection.add.aspx)**
method. Also note that the template corresponding to **Wiki Page Library** is
actually **
[SPListTemplateType.WebPageLibrary](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.splisttemplatetype.aspx)**,
and you can use the **
[SPUtility.AddDefaultWikiContent](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.utilities.sputility.adddefaultwikicontent.aspx)**
method to create the two default pages that are automatically added when you
create a wiki library through the SharePoint site.

Lastly, as I demonstrated in the earlier post, by using PowerShell you can
easily add a wiki library to a number of project sites at once:

```
$sitesToUpgrade =
    @(
        "http://cyclops/sites/AdventureWorks",
        "http://cyclops/sites/Demo",
        "http://cyclops/sites/Toolbox"
    )

$sitesToUpgrade |
    ForEach-Object {
        $DebugPreference = "SilentlyContinue"
        $web = Get-SPWeb $_

        $DebugPreference = "Continue"
        AddWikiLibrary $web $name $description
        $web.Dispose()
    }
```

