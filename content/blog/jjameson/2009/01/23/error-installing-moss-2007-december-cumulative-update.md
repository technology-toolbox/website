---
title: "Error Installing MOSS 2007 December Cumulative Update"
date: 2009-01-23T11:43:00+08:00
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

C:\NotBackedUp\Temp&gt;<kbd>psconfig -cmd upgrade -inplace b2b -wait</kbd>

<samp>Copyright (C) Microsoft Corporation 2005. All rights reserved.ion 12.0.6300.5000<br>
<br>
Performing configuration task 1 of 4<br>
Initializing SharePoint Products and Technologies upgrade...<br>
<br>
Successfully initialized SharePoint Products and Technologies upgrade.<br>
<br>
Performing configuration task 2 of 4<br>
Initiating the upgrade sequence...<br>
<br>
Successfully initiated the upgrade sequence.<br>
<br>
Performing configuration task 3 of 4<br>
Upgrading SharePoint Products and Technologies...<br>
<br>
Failed to upgrade SharePoint Products and Technologies.<br>
<br>
An exception of type Microsoft.SharePoint.Upgrade.SPUpgradeException was thrown.
Additional exception information: Upgrade completed with errors. Review the
upgrade.log file located in C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Logs\Upgrade.log. The number of errors and warnings is
listed at the end of the upgrade log file.<br>
<br>
Total number of configuration settings run: 3<br>
Total number of successful configuration settings: 2<br>
Total number of unsuccessful configuration settings: 1<br>
Successfully stopped the configuration of SharePoint Products and Technologies.<br>
Configuration of SharePoint Products and Technologies failed. Configuration
must be performed before you use SharePoint Products and Technologies. For further
details, see the diagnostic log located at C:\Program Files\Common Files\Microsoft
Shared\Web Server Extensions\12\LOGS\PSCDiagnostics_1_19_2009_6_33_30_5_252811372.log
and the application event log.<br>
</samp>

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

C:\NotBackedUp\Temp&gt;<kbd>psconfig -cmd installfeatures</kbd>

<samp>Copyright (C) Microsoft Corporation 2005. All rights reserved.ion 12.0.6300.5000<br>
<br>
Performing configuration task 1 of 5<br>
Initializing SharePoint Products and Technologies upgrade...<br>
<br>
Successfully initialized SharePoint Products and Technologies upgrade.<br>
<br>
Performing configuration task 2 of 5<br>
Initiating the upgrade sequence...<br>
<br>
Successfully initiated the upgrade sequence.<br>
<br>
Performing configuration task 3 of 5<br>
Registering SharePoint features...<br>
<br>
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\ExpirationWorkflow\Feature.xml.<br>
<br>
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\ReviewWorkflows\Feature.xml.<br>
<br>
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\S2BaseSiteStapling\feature.xml.<br>
<br>
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\S2SearchAdmin\feature.xml.<br>
<br>
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\S2SiteAdmin\feature.xml.<br>
<br>
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\SearchServerWizardFeature\Feature.xml.<br>
<br>
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\SignaturesWorkflow\Feature.xml.<br>
<br>
Successfully installed feature C:\Program Files\Common Files\Microsoft Shared\Web
Server Extensions\12\Template\Features\TranslationWorkflow\Feature.xml.<br>
<br>
Feature installation has determined that there are no new features that require
installation.<br>
<br>
Successfully registered SharePoint features.<br>
<br>
Performing configuration task 4 of 5<br>
Upgrading SharePoint Products and Technologies...<br>
<br>
Successfully upgraded SharePoint Products and Technologies.<br>
<br>
Performing configuration task 5 of 5<br>
Finalizing the SharePoint Products and Technologies configuration...<br>
<br>
Successfully completed the SharePoint Products and Technologies configuration.<br>
<br>
Total number of configuration settings run: 5<br>
Total number of successful configuration settings: 5<br>
Total number of unsuccessful configuration settings: 0<br>
Successfully stopped the configuration of SharePoint Products and Technologies.<br>
Configuration of the SharePoint Products and Technologies has succeeded.<br>
</samp>

Once the new features are installed, the upgrade goes without a hitch:

C:\NotBackedUp\Temp&gt;<kbd>psconfig -cmd upgrade -inplace b2b -wait</kbd>

<samp>Copyright (C) Microsoft Corporation 2005. All rights reserved.ion 12.0.6300.5000<br>
<br>
Performing configuration task 1 of 4<br>
Initializing SharePoint Products and Technologies upgrade...<br>
<br>
Successfully initialized SharePoint Products and Technologies upgrade.<br>
<br>
Performing configuration task 2 of 4<br>
Initiating the upgrade sequence...<br>
<br>
Successfully initiated the upgrade sequence.<br>
<br>
Performing configuration task 3 of 4<br>
Upgrading SharePoint Products and Technologies...<br>
<br>
Successfully upgraded SharePoint Products and Technologies.<br>
<br>
Performing configuration task 4 of 4<br>
Finalizing the SharePoint Products and Technologies configuration...<br>
<br>
Successfully completed the SharePoint Products and Technologies configuration.<br>
<br>
Total number of configuration settings run: 4<br>
Total number of successful configuration settings: 4<br>
Total number of unsuccessful configuration settings: 0<br>
Successfully stopped the configuration of SharePoint Products and Technologies.<br>
Configuration of the SharePoint Products and Technologies has succeeded.</samp>

It turns out there are several other gotchas when working with SharePoint on  Windows Server 2008. However, given the length of this post, I think I'll blog about  those separately ;-)

