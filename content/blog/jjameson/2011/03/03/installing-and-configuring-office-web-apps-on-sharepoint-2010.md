---
title: "Installing and Configuring Office Web Apps (on SharePoint 2010)"
date: 2011-03-03T00:16:00-07:00
excerpt: "In the current sprint of the project I'm working on, we are deploying Office Web Apps to support an enterprise collaboration platform based on SharePoint Server 2010. 
 While creating the installation guide for this sprint, I used the following TechNet..."
draft: true
categories: ["Infrastructure", "SharePoint"]
tags: ["Infrastructure", "SharePoint 
		2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/03/installing-and-configuring-office-web-apps-on-sharepoint-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/03/installing-and-configuring-office-web-apps-on-sharepoint-2010.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

In the current sprint of the project I'm working on, we are deploying Office  Web Apps to support an enterprise collaboration platform based on SharePoint Server  2010.

While creating the installation guide for this sprint, I used the following TechNet  article as a reference for the section on installing and configuring Office Web  Apps:

<cite>Deploy Office Web Apps (Installed on SharePoint 2010 Products)</cite>
[http://technet.microsoft.com/en-us/library/ff431687.aspx](http://technet.microsoft.com/en-us/library/ff431687.aspx)

While the above article includes *most* of the steps you need to perform  when deploying Office Web Apps on SharePoint 2010, it currently seems to be lacking  a few important pieces (at least based upon my experience):

- Configure Excel Services Application trusted location (if you are using
  both HTTP and HTTPS)
- Configure the Office Web Apps cache
- Grant access to the Web application content database for Office Web Apps

These steps are performed after completing the deployment steps in the above  TechNet article.

For the remainder of this post, imagine that we are installing Office Web Apps  on the extranet site for Fabrikam Technologies (http://extranet.fabrikam.com) and  we want to provide the ability for Fabrikam employees and its partners to collaborate  on documents created in Microsoft Word, Excel, and PowerPoint.

Assume we have created a new Web application, [configured it for claims-based authentication](/blog/jjameson/2011/02/19/configuring-claims-based-authentication-in-sharepoint-server-2010), and deployed Office Web Apps  by following the steps in the aforementioned TechNet article.

The relevant service accounts for the Fabrikam extranet site are listed in the  following table.

<caption>Table 1 - Service Accounts</caption>| User Logon Name | Full Name | Description |
| --- | --- | --- |
| EXTRANET\svc-sharepoint | Service account for SharePoint | SharePoint farm account used to create and access the SharePoint configuration 
	database. |
| EXTRANET\svc-spserviceapp | Service account for SharePoint service applications  | Used as the application pool identity for SharePoint service applications
	 |
| EXTRANET\svc-web-fabrikam | Service account for Fabrikam Web site | Used for the application pool for the Fabrikam extranet Web application |
In order to resolve a few issues with the deployment and ensure it conforms to  recommended best practices, we need to perform some additional configuration steps.

### Configure Excel Services Application trusted location

When the Excel Services Application is created, a default trusted location is  automatically configured (**http://**) for all content on the SharePoint  farm. This default trusted location enables any file to be loaded from the SharePoint  farm into Excel Services. However, this default trusted location does not support  HTTPS (https://) and therefore results in the following error when attempting to  access an Excel workbook using a secured connection:

> This workbook cannot be opened because it is not stored in an Excel Services
> Application trusted location.

Use the following procedure to change the default trusted location to support  HTTPS.

> **Important**
>
> Skip this section for environments that are not configured with SSL certificates (e.g. development environments).

#### To configure the Excel Services Application trusted location for HTTPS instead

of HTTP:

1. On the Central Administration home page, in the **Application Management** section, click **Manage service applications**.
2. On the **Service Applications** tab, click **Excel Services
   Application** (where the **Type** column is **Excel
   Services Application Web Service Application**).
3. On the **Manage Excel Services Application** page, click
   **Trusted File Locations**.
4. On the **Excel Services Application Trusted File Locations** page, click the default trusted file location (**http://**)
   to edit the corresponding settings.
5. On the **Excel Services Application Edit Trusted File Location** page, in the **Location** section, change the **Address** from **http://** to **https://**
   and then click **OK**.

> **Note**
>
> Since users of the Fabrikam extranet site are automatically redirected from http:// to https:// during sign in (via the [Claims Login Form Web Part](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010)), it is not expected that Excel Services will be used over HTTP (only HTTPS). If it is necessary to support both HTTP and HTTPS, then a separate trusted file location will need to be configured.

### Configure the Office Web Apps cache

By default, when you install Office Web Apps, the cache available to render documents  is 100 GB and the cache expiration period is 30 days. The cached content for Office  Web Apps is stored in a SharePoint content database.

It is recommended to isolate the content database used for the Office Web Apps  cache, so that cached files do not contribute to size of the "main" content database(s)  for the Web application. Also note that anytime you create a new SharePoint content  database, it is recommended to expand the initial database files (at least in a  production environment).

> **Important**
>
> You must start a new instance of the SharePoint 2010 Management Shell
> after installing the Office Web Apps in order to use the new PowerShell
> cmdlets (e.g.
> [Set-SPOfficeWebAppsCache](http://technet.microsoft.com/en-us/library/ff608181.aspx)).
>
> Also note that you may need to wait a few minutes (after installing Office
> Web Apps or rebuilding the Web application) before performing the following
> procedure (for the SharePoint timer job to configure the cache on the site
> collection before moving it to a separate content database).

The following procedures are used to reduce the Office Web Apps cache size to  30 GB, move the cache to a new content database, and expand the corresponding database  files.

#### To configure the Office Web Apps cache and create a separate content database

for caching:

1. On the **Start** menu, click **All Programs**,
   click **Microsoft SharePoint 2010 Products**, right-click
   **SharePoint 2010 Management Shell**, and then click **Run
   as administrator**. If prompted by User Account Control to allow the
   program to make changes to the computer, click Yes.

2. From the Windows PowerShell command prompt, run the following script:
   
   ```
   $ErrorActionPreference = "Stop"
   
   Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0
   
   function ConfigureOfficeWebAppsCache(
       [string] $webAppUrl,
       [int] $cacheSizeInGigabytes,
       [int] $expirationPeriodInDays,
       [string] $cacheDatabaseName)
   {
       Write-Host ("Configuring Office Web Apps cache for Web application" `
           + " ($webAppUrl)...")
   
       Write-Debug "cacheSizeInGigabytes: $cacheSizeInGigabytes"
       Write-Debug "expirationPeriodInDays: $expirationPeriodInDays"
       Write-Debug "cacheDatabaseName: $cacheDatabaseName"
   
       $cacheSizeInBytes = $cacheSizeInGigabytes * 1024 * 1024 * 1024
       $webApp = Get-SPWebApplication --Identity $webAppUrl -Debug:$false
       $webApp | Set-SPOfficeWebAppsCache `
           -ExpirationPeriodInDays $expirationPeriodInDays `
           -MaxSizeInBytes $cacheSizeInBytes -Debug:$false
   
       $cacheDatabase = Get-SPContentDatabase $cacheDatabaseName -Debug:$false -EA 0
   
       if ($cacheDatabase -eq $null)
       {
           $cacheDatabase = New-SPContentDatabase -Name $cacheDatabaseName `
               -WebApplication $webApp -Debug:$false
       }
   
       Get-SPOfficeWebAppsCache -WebApplication $webapp -Debug:$false |
           Move-SPSite -DestinationDatabase $cacheDatabase -Confirm:$false `
           -Debug:$false
   
       Write-Host "Restricting the cache database to a single site collection..."
   
       $database = Get-SPContentDatabase $cacheDatabase
   
       $database.MaximumSiteCount = 1
       $database.WarningSiteCount = 0
       $database.Update()
   
       Write-Host -Fore Green ("Successfully configured Office Web Apps cache for" `
           + " Web application ($webAppUrl).")
   }
   
   function Main()
   {
       $webAppUrl = "http://extranet.fabrikam.com"
       $cacheSizeInGigabytes = 20
       $expirationPeriodInDays = 30
       $cacheDatabaseName = "OfficeWebAppsCache"
   
       ConfigureOfficeWebAppsCache $webAppUrl $cacheSizeInGigabytes `
           $expirationPeriodInDays $cacheDatabaseName
   }
   
   Main
   ```

3. Wait for the script to complete and verify that no errors occurred during
   the process.

4. Reset Internet Information Services (IIS) in order for the change to take
   effect:
   
   ```
   iisreset
   ```

#### To increase the size of the database files for the Office Web Apps cache:

1. Start SQL Server Management Studio and connect to the appropriate server.
2. In the **Object Explorer**, expand the **Databases** folder.
3. Right-click the **OfficeWebAppsCache** database and then click
   **Properties**.
4. In the **Database Properties** dialog, in the **Select
   a page** area on the left, click **Files**.
5. Using the settings specified in the following table, specify the new values
   for **Initial Size** and **Autogrowth**.

<caption>Table 2 - Initial data and log file sizes</caption>    | Database | Logical Name | File Type | Filegroup | Initial Size [MB] | Autogrowth |
    | --- | --- | --- | --- | --- | --- |
    | OfficeWebAppsCache | OfficeWebAppsCache | Data | PRIMARY | 10,000 | By 500 MB, unrestricted gerowth |
    |  | OfficeWebAppsCache | Log | N/A | 400 | By 10 percent, restricted growth: 4,000 MB |
6. Click **OK**.
The following SQL statements can be used as an alternative to setting the sizes  through the Database Properties dialog:

```
USE [master]
GO
ALTER DATABASE [OfficeWebAppsCache]
MODIFY FILE(
    NAME = N'OfficeWebAppsCache'
    , SIZE = 10240000KB
    , FILEGROWTH = 512000KB)
GO
ALTER DATABASE [OfficeWebAppsCache]
MODIFY FILE(
    NAME = N'OfficeWebAppsCache_log'
    , SIZE = 409600KB
    , MAXSIZE = 4096000KB)
GO
```

### Grant access to the Web application content database for Office Web Apps

Since the service account used to run the Office Web Apps service applications  is different from the account used to run the application pool for the Web application,  it is necessary to explicitly grant access to all of the content databases used  by the Web application.

#### To grant the Office Web Apps service account access to the content databases:

1. On the **Start** menu, click **All Programs**,
   click **Microsoft SharePoint 2010 Products**, right-click
   **SharePoint 2010 Management Shell**, and then click **Run
   as administrator**. If prompted by User Account Control to allow the
   program to make changes to the computer, click Yes.

2. From the Windows PowerShell command prompt, type the following commands:
   
   ```
   $webApp = Get-SPWebApplication "http://extranet.fabrikam.com"
   
   $webApp.GrantAccessToProcessIdentity("EXTRANET\svc-spserviceapp")
   ```
