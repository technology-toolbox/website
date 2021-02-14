---
title: "Performing a \"Do Over\" with TFS 2010 Upgrade"
date: 2010-05-19T21:23:00+08:00
excerpt: "Regardless of whether you call it a \"mulligan\", a \"do over\", or whatever, the fact is you may encounter errors during your upgrade to Team Foundation Server (TFS) 2010 -- hopefully in your Development or Test environment first, not when upgrading your..."
draft: true
categories: ["Development"]
tags: ["TFS"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/20/performing-a-do-over-with-tfs-2010-upgrade.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/20/performing-a-do-over-with-tfs-2010-upgrade.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Regardless of whether you call it a "mulligan", a "do over", or whatever, the fact is you may encounter errors during your upgrade to Team Foundation Server (TFS) 2010 -- hopefully in your Development or Test environment first, not when upgrading your Production environment -- and consequently you want to start over and perform the upgrade again.

While the TFS installation guide (and even the TFS uprade tool itself) makes it very clear to backup your data before upgrading, it doesn't provide any instructions for how to recover from a failed upgrade.

If you need to perform the upgrade again (and you don't have the luxury of applying a snapshot to "rollback" your Development or Test environment), then you can use the steps described below.

First, use the Team Foundation Server Configuration Tool to unconfigure components on the server:

```
cd "%ProgramFiles\Microsoft Team Foundation Server 2010\Tools"
```

```
TfsConfig.exe setup /uninstall:All
```

Next, remove the SharePoint content database (e.g. WSS\_Content\_TFS) containing the upgraded TFS project sites:

1. On the SharePoint Central Administration home page, in the **Application
   Management** section, click **Manage content databases**.
2. On the **Manage Content Databases** page, in the list of content
   databases, click the name of the content database to remove (e.g. **WSS\_Content\_TFS**).
3. On the **Manage Content Database Settings** page, click the
   **Remove content database **checkbox. When prompted to confirm
   removing the content database, click **OK**.

Delete the databases that will be restored from backup (e.g. ReportServerDB, ReportServerTempDB, TfsWarehouse, and WSS\_Content\_TFS) as well as the new TFS 2010 databases:

- Tfs\_Configuration
- Tfs\_DefaultCollection
- Tfs\_Warehouse
- Tfs\_Analysis (Analysis Services)

Next, restore the following databases:

- ReportServerDB
- ReportServerTempDB
- TfsActivityLogging
- TfsBuild
- TfsIntegration
- TfsVersionControl
- TfsWarehouse (Database Engine)
- TfsWarehouse (Analysis Services)
- TfsWorkItemTracking
- TfsWorkItemTrackingAttachments
- WSS\_Content\_TFS

Then attach the SharePoint content database (e.g. WSS\_Content\_TFS) containing the TFS project sites to the SharePoint Web application.

To attach the content database in SharePoint Server 2010 by using Windows Powershell:

1. On the **Start** menu, click **All Programs**,
   click **Microsoft SharePoint 2010 Products**, right-click
   **SharePoint 2010 Management Shell**, and then click **Run
   as administrator**. If prompted by **User Account Control
   **to allow the program to make changes to this computer, click
   **Yes**.

2. At the Windows PowerShell command prompt, type the following command:
   
   ```
   Mount-SPContentDatabase -Name <DatabaseName> -DatabaseServer <ServerName> 
   	-WebApplication <URL> [-Updateuserexperience]
   ```

Where:

    - <var>&lt;DatabaseName&gt;</var> is the name of the database you want 
    to upgrade.
    - <var>&lt;ServerName&gt;</var> is server on which the database is stored.
    - <var>&lt;URL&gt;</var> is the URL for the Web application that will 
    host the sites.
    - <var>-Updateuserexperience</var> specifies to update the sites with 
    the new SharePoint user experience (part of Visual Upgrade). If you omit 
    this parameter, the sites retain the old user experience after upgrade.

For example:

    ```
    Mount-SPContentDatabase -Name WSS_Content_TFS -DatabaseServer CYCLOPS-DEV 
    	-WebApplication http://cyclops-dev -Updateuserexperience
    ```

Next, run the Team Foundation Server Administration Console (TfsMgmt.exe) and click **Configure Installed Features**.

This will launch the Team Foundation Server Configuration Center and allow you to complete the upgrade process again.

