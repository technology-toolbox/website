---
title: "Use PowerShell to \"Reset to Site Definition\" in SharePoint Server 2010"
date: 2010-05-18T19:29:00-06:00
excerpt: "In one of my posts last month , I provided the following steps to \"reghost\" all of the pages in a Team Foundation Server (TFS) project site: 
 
 Browse to the Site Settings page for the site (e.g. http://cyclops/sites/Demo/_layouts/settings.aspx )...."
aliases: ["/blog/jjameson/archive/2010/05/18/use-powershell-to-quot-reset-to-site-definition-quot-in-sharepoint-server-2010.aspx"]
draft: true
categories: ["Development", "SharePoint"]
tags: ["TFS", "
                SharePoint 2010", "PowerShell"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/18/use-powershell-to-quot-reset-to-site-definition-quot-in-sharepoint-server-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/18/use-powershell-to-quot-reset-to-site-definition-quot-in-sharepoint-server-2010.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In [one of my posts last month](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010), I provided the following steps to "reghost"         all of the pages in a Team Foundation Server (TFS) project site:

1. Browse to the Site Settings page for the site (e.g. [http://cyclops/sites/Demo/\_layouts/settings.aspx](http://cyclops/sites/Demo/_layouts/settings.aspx)).
2. On the **Site Settings** page, in the **Site Actions**
   section, click **Reset to site definition**.
3. On the **Reset Page to Site Definition Version** page, click the option
   to **Reset all pages in this site to site definition version**, and
   then click **Reset**.

Today, I was about to perform this process manually on several sites, but then I         decided to spend a few minutes exploring the **Reset Page to Site Definition Version**         page (e.g. [http://cyclops/sites/Demo/\_layouts/reghost.aspx](http://cyclops/sites/Demo/_layouts/reghost.aspx)).

That's when I discovered the **[SPWeb.RevertAllDocumentContentStreams](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spweb.revertalldocumentcontentstreams.aspx)** method. [I'm actually a little         embarrassed that I wasn't aware of this little gem before. A few years ago, while         working on the Agilent Technologies project, I'd used some custom C# to reghost         specific pages (to resolve deployment issues), but I clearly recall some general         "wonkiness" with it. If I'd known about the **RevertAllDocumentContentStreams**         method a few years ago, I probably would have just run that as part of each and         every deployment ;-) ]

Anyway, once you know about the method, it's very easy to call it from PowerShell.

Here's a little script to reset all pages in a list of sites to the site definition         version:

```
$sitesToReset =
    @(
        "http://cyclops/sites/AdventureWorks",
        "http://cyclops/sites/Demo",
        "http://cyclops/sites/Toolbox"
    )

$sitesToReset |
    ForEach-Object {
        $DebugPreference = "SilentlyContinue"
        $web = Get-SPWeb $_

        $DebugPreference = "Continue"
        
        Write-Debug "Reghosting all pages in site ($($web.Url))..."
        $web.RevertAllDocumentContentStreams()
        $web.Dispose()
    }
```

