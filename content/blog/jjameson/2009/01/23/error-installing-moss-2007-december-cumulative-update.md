---
title: "Error Installing MOSS 2007 December Cumulative Update"
date: 2009-01-23T11:43:00-07:00
excerpt: "Earlier this week I built a new Microsoft Office SharePoint Server (MOSS) 2007 development VM using a fresh install of Windows Server 2008, SQL Server 2008, and Visual Studio 2008. The process wasn't quite as smooth as I had hoped. 
 One of the issues..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/01/23/error-installing-moss-2007-december-cumulative-update.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/01/23/error-installing-moss-2007-december-cumulative-update.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Earlier this week I built a new Microsoft Office SharePoint Server (MOSS) 2007  development VM using a fresh install of Windows Server 2008, SQL Server 2008, and  Visual Studio 2008. The process wasn't quite as smooth as I had hoped.

One of the issues I encountered occurs when you attempt to go straight from a  slipstreamed install of MOSS 2007 Service Pack 1 (SP1) to the [December Cumulative Update](http://support.microsoft.com/kb/960011)  (CU), bypassing the July Infrastructure Update (IU).

Since the December CU includes the July IU, I thought this would be okay. Unfortunately  an `SPUpgradeException` occurs during the "build-to-build" upgrade:

{{< console-block-start >}}

C:\NotBackedUp\Temp&gt;{{< kbd "psconfig -cmd upgrade -inplace b2b -wait" >}}

{{< sample-block >}}

Copyright (C) Microsoft Corporation 2005. All rights reserved.ion 12.0.6300.5000\
\
Performing configuration task 1 of 4\
Initializing SharePoint Products and Technologies upgrade...\
\
Successfully initialized SharePoint Products and Technologies upgrade.\
\
Performing configuration task 2 of 4\
Initiating the upgrade sequence...\
\
Successfully initiated the upgrade sequence.\
\
Performing configuration task 3 of 4\
Upgrading SharePoint Products and Technologies...\
\
Failed to upgrade SharePoint Products and Technologies.\
\
An exception of type Microsoft.SharePoint.Upgrade.SPUpgradeException was thrown.
Additional exception information: Upgrade completed with errors. Review the
upgrade.log file located in C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Logs\Upgrade.log. The number of errors and warnings is
listed at the end of the upgrade log file.\
\
Total number of configuration settings run: 3\
Total number of successful configuration settings: 2\
Total number of unsuccessful configuration settings: 1\
Successfully stopped the configuration of SharePoint Products and Technologies.\
Configuration of SharePoint Products and Technologies failed. Configuration
must be performed before you use SharePoint Products and Technologies. For further
details, see the diagnostic log located at C:\Program Files\Common Files\Microsoft
Shared\Web Server Extensions\12\LOGS\PSCDiagnostics\_1\_19\_2009\_6\_33\_30\_5\_252811372.log
and the application event log.\

{{< /sample-block >}}

{{< console-block-end >}}

Upon cracking open the upgrade log file, I found the following:

```
[SPWebTemplateSequence] [DEBUG] [1/19/2009 6:35:31 AM]: Template OSRV#0: Activating site-collection-scoped features...
[SPWebTemplateSequence] [DEBUG] [1/19/2009 6:35:31 AM]: Template OSRV#0: Activating feature 2b1e4cbf-b5ba-48a4-926a-37100ad77dee in site collection with URL "http://ssp-public-local/ssp/admin", force=False.
[SPWebTemplateSequence] [ERROR] [1/19/2009 6:35:31 AM]: Template OSRV#0: Exception thrown in activating SPSite scoped features for SPSite with URL "http://ssp-public-local/ssp/admin" (Id=9774ded8-a54e-49e7-b010-7f741446e28f). Skipping this SPSite for template upgrade.  Exception: System.InvalidOperationException: Feature '2b1e4cbf-b5ba-48a4-926a-37100ad77dee' is not installed in this farm, and can not be added to this scope.
   at Microsoft.SharePoint.SPFeatureCollection.AddInternal(Guid featureId, SPFeaturePropertyCollection properties, Boolean force, Boolean fMarkOnly)
   at Microsoft.SharePoint.SPFeatureCollection.Add(Guid featureId, Boolean force)
   at Microsoft.SharePoint.Upgrade.SPWebTemplateSequence.ActivateSiteFeatures(List`1 lstsiteidToUpgrade, List`1& lstsiteidExceptions, List`1& lstwebinfoExceptions)
```

Notice that it is complaining about a feature not being installed on the farm.  This feature ('2b1e4cbf-b5ba-48a4-926a-37100ad77dee') is actually the new "S2SearchAdmin"  interface included in the July IU that substantially improves the Search Administration  pages. Assuming you didn't install the July IU separately (which on this brand new  VM, I had not), you need to explicitly install the new features first:

{{< console-block-start >}}

C:\NotBackedUp\Temp&gt;{{< kbd "psconfig -cmd installfeatures" >}}

{{< sample-block >}}

Copyright (C) Microsoft Corporation 2005. All rights reserved.ion 12.0.6300.5000\
\
Performing configuration task 1 of 5\
Initializing SharePoint Products and Technologies upgrade...\
\
Successfully initialized SharePoint Products and Technologies upgrade.\
\
Performing configuration task 2 of 5\
Initiating the upgrade sequence...\
\
Successfully initiated the upgrade sequence.\
\
Performing configuration task 3 of 5\
Registering SharePoint features...\
\
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\ExpirationWorkflow\Feature.xml.\
\
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\ReviewWorkflows\Feature.xml.\
\
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\S2BaseSiteStapling\feature.xml.\
\
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\S2SearchAdmin\feature.xml.\
\
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\S2SiteAdmin\feature.xml.\
\
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\SearchServerWizardFeature\Feature.xml.\
\
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\SignaturesWorkflow\Feature.xml.\
\
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\TranslationWorkflow\Feature.xml.\
\
Feature installation has determined that there are no new features that require
installation.\
\
Successfully registered SharePoint features.\
\
Performing configuration task 4 of 5\
Upgrading SharePoint Products and Technologies...\
\
Successfully upgraded SharePoint Products and Technologies.\
\
Performing configuration task 5 of 5\
Finalizing the SharePoint Products and Technologies configuration...\
\
Successfully completed the SharePoint Products and Technologies configuration.\
\
Total number of configuration settings run: 5\
Total number of successful configuration settings: 5\
Total number of unsuccessful configuration settings: 0\
Successfully stopped the configuration of SharePoint Products and Technologies.\
Configuration of the SharePoint Products and Technologies has succeeded.\

{{< /sample-block >}}

{{< console-block-end >}}

Once the new features are installed, the upgrade goes without a hitch:

{{< console-block-start >}}

C:\NotBackedUp\Temp&gt;{{< kbd "psconfig -cmd upgrade -inplace b2b -wait" >}}

{{< sample-block >}}

Copyright (C) Microsoft Corporation 2005. All rights reserved.ion 12.0.6300.5000\
\
Performing configuration task 1 of 4\
Initializing SharePoint Products and Technologies upgrade...\
\
Successfully initialized SharePoint Products and Technologies upgrade.\
\
Performing configuration task 2 of 4\
Initiating the upgrade sequence...\
\
Successfully initiated the upgrade sequence.\
\
Performing configuration task 3 of 4\
Upgrading SharePoint Products and Technologies...\
\
Successfully upgraded SharePoint Products and Technologies.\
\
Performing configuration task 4 of 4\
Finalizing the SharePoint Products and Technologies configuration...\
\
Successfully completed the SharePoint Products and Technologies configuration.\
\
Total number of configuration settings run: 4\
Total number of successful configuration settings: 4\
Total number of unsuccessful configuration settings: 0\
Successfully stopped the configuration of SharePoint Products and Technologies.\
Configuration of the SharePoint Products and Technologies has succeeded.

{{< /sample-block >}}

{{< console-block-end >}}

It turns out there are several other gotchas when working with SharePoint on  Windows Server 2008. However, given the length of this post, I think I'll blog about  those separately ;-)

