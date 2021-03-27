---
title: Reporting Errors with TFS Migration/Upgrade
date: 2010-05-20T04:07:00-06:00
excerpt:
  When upgrading Team Foundation Server (TFS) -- or, really, any application
  that utilizes SQL Server Reporting Services -- you might choose to restore
  your data to a new environment (for example, to migrate to new hardware, or to
  validate the upgrade in...
aliases:
  [
    "/blog/jjameson/archive/2010/05/19/reporting-errors-with-tfs-migration-upgrade.aspx",
    "/blog/jjameson/archive/2010/05/20/reporting-errors-with-tfs-migration-upgrade.aspx",
  ]
draft: true
categories: ["Development"]
tags: ["SQL Server", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/05/20/reporting-errors-with-tfs-migration-upgrade.aspx"
---

When upgrading Team Foundation Server (TFS) -- or, really, any application that
utilizes SQL Server Reporting Services -- you might choose to restore your data
to a new environment (for example, to migrate to new hardware, or to validate
the upgrade in Development and Test environments before upgrading your
Production environment).

Depending on the exact steps you perform during the upgrade, you may experience
issues with Reporting Services. These issues may occur during the upgrade (e.g.
when TFS upgrades the reports for each team project), as well as after the
upgrade is complete (e.g. when creating a new team project).

During the upgrade, a warning occurs indicating the reports could not be
successfully upgraded. The log file contains warnings for each TFS project that
resemble the following:

{{< div-block "fst-italic" >}}

> [Warning] Failed to upgrade Reporting settings for project AdventureWorks:
> TF30225: Error uploading report 'Work Item with Tasks':
> System.Web.Services.Protocols.SoapException: An error occurred within the
> report server database. This may be due to a connection failure, timeout or
> low disk condition within the database. ---&gt;
> Microsoft.ReportingServices.Diagnostics.Utilities.ReportServerStorageException:
> An error occurred within the report server database. This may be due to a
> connection failure, timeout or low disk condition within the database. ---&gt;
> System.Data.SqlClient.SqlException: The EXECUTE permission was denied on the
> object 'xp\_sqlagent\_notify', database 'mssqlsystemresource', schema 'sys'.\
> at
> Microsoft.ReportingServices.WebServer.ReportingService2005Impl.SetCacheOptions(String
> Report, Boolean CacheReport, ExpirationDefinition Expiration)\
> at
> Microsoft.ReportingServices.WebServer.ReportingService2005.SetCacheOptions(String
> Report, Boolean CacheReport, ExpirationDefinition Expiration)

{{< /div-block >}}

After the upgrade, when attempting to create a new TFS project (which adds new
reports to Reporting Services), you may see the following:

{{< div-block "fst-italic" >}}

> **TF301777: Team Project Creation Failed**
> 
> New Team Project Wizard encountered the following error and could not
> continue.
> 
> **Error**
> 
> The Project Creation Wizard encountered an error while creating reports to the
> SQL Server Reporting Services on
> http://cyclops-dev/ReportServer/ReportService2005.asmx.
> 
> **Explanation**
> 
> The Project Creation Wizard encountered a problem while creating reports on
> the SQL Server Reporting Services on
> http://cyclops-dev/ReportServer/ReportService2005.asmx. The reason for the
> failure cannot be determined at this time. Because the operation failed, the
> wizard was not able to finish creating the SQL Server Reporting Services site.
> 
> **User Action**
> 
> Contact the administrator for the SQL Server Reporting Services on
> http://cyclops-dev/ReportServer/ReportService2005.asmx to confirm that the SQL
> Server Reporting Services server is running and you have sufficient privileges
> to create a project. Your user account on SQL Server Reporting Services must
> have Content Manager permission to create a new project. Also, you might find
> additional helpful information in the project creation log. The log shows each
> action taken by the wizard at the time of the failure and may include
> additional details about the error.

{{< /div-block >}}

The problem occurs if you restore the ReportServerDB and ReportServerTempDB
databases without first configuring SQL Server Reporting Services in the new
environment.

It turns out that when you configure Reporting Services, it not only creates the
two databases noted above, but also grants permissions to a number of stored
procedures in the master and msdb databases to the RSExecRole database role.

An easy way to confirm the problem is to browse to the Schedules tab in Report
Manager:

1. Browse to the Report Manager site (e.g.
   [http://cyclops-dev/Reports](http://cyclops-dev/Reports)).
2. On the Report Manager home page, in the masthead of the page, click **Site
   Settings**.
3. On the Site Settings page, click the **Schedules** tab.

An error similar to the following should be displayed:

{{< div-block "errorMessage" >}}

> EXECUTE permission was denied on the object 'xp\_sqlagent\_notify', database
> 'mssqlsystemresource', schema 'sys'

{{< /div-block >}}

Fortunately, once you've confirmed the root cause of the issue, resolving it is
simply a matter of granting the necessary permissions by running the following
SQL statements (courtesy of Tudor Trufinescu in a
[forum post on MSDN](http://social.msdn.microsoft.com/forums/en-US/sqlreportingservices/thread/444c3bab-985b-40a0-8362-2742df1a6577/)):

```SQL
USE master
GO
GRANT EXECUTE ON master.dbo.xp_sqlagent_notify TO RSExecRole
GO
GRANT EXECUTE ON master.dbo.xp_sqlagent_enum_jobs TO RSExecRole
GO
GRANT EXECUTE ON master.dbo.xp_sqlagent_is_starting TO RSExecRole
GO
USE msdb
GO
-- Permissions for SQL Agent SP's
GRANT EXECUTE ON msdb.dbo.sp_help_category TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_add_category TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_add_job TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_add_jobserver TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_add_jobstep TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_add_jobschedule TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_help_job TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_delete_job TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_help_jobschedule TO RSExecRole
GO
GRANT EXECUTE ON msdb.dbo.sp_verify_job_identifiers TO RSExecRole
GO
GRANT SELECT ON msdb.dbo.sysjobs TO RSExecRole
GO
GRANT SELECT ON msdb.dbo.syscategories TO RSExecRole
GO
```

However, even after fixing the permissions issues in SQL Server, you still need
to complete the upgrade of the reports for existing TFS projects (which failed
during the TFS upgrade process). Otherwise, the Web Parts that render reports
will continue to show errors similar to the following:

{{< div-block "fst-italic" >}}

> **Reporting Services Error**
> 
> The item '/TfsReports/DefaultCollection/AdventureWorks/Remaining Work' cannot
> be found. (rsItemNotFound)

{{< /div-block >}}

I initially attempted to fix this issue using the File.BatchNewTeamProject
command in Visual Studio, with the following XML input file:

```XML
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="ProjectCreationSettingsFileSchema.xsd">
  <TFSName>http://cyclops-dev:8080/tfs/DefaultCollection</TFSName>
  <LogFolder>C:\NotBackedUp\Temp</LogFolder>
  <ProjectName>AdventureWorks</ProjectName>
  <AddFeaturesToExistingProject>true</AddFeaturesToExistingProject>
  <ProjectReportsEnabled>true</ProjectReportsEnabled>
  <ProjectReportsForceUpload>true</ProjectReportsForceUpload>
  <ProjectSiteEnabled>false</ProjectSiteEnabled>
  <ProcessTemplateName>MSF for Agile Software Development - v4.2</ProcessTemplateName>
</Project>
```

This process is described in more detail in the following MSDN article:

{{< reference title="Adding Dashboards and Reports to Upgraded Team Projects"
linkHref="http://msdn.microsoft.com/en-us/library/ff462695(v=VS.100).aspx" >}}

While this successfully uploaded the missing reports (e.g.
/TfsReports/DefaultCollection/AdventureWorks/Remaining Work), it only led to a
different error.

{{< div-block "fst-italic" >}}

> **Reporting Services Error**
> 
> Default value or value provided for the report parameter 'IterationParam' is
> not a valid value. (rsInvalidReportParameter)

{{< /div-block >}}

I rebuilt the TFS warehouse and Analysis Services databases to try to resolve
this error, but that didn't help.

Consequently, I "rolled back" my TFS upgrade and performed the upgrade again
using the process described in
[my previous blog post](/blog/jjameson/2010/05/20/performing-a-do-over-with-tfs-2010-upgrade).
After granting permissions to the RSExecRole, the TFS upgrade completed without
any issues (i.e. the warnings reported during the initial TFS upgrade no longer
occurred).

After waiting for the cube to be processed, the reports on the TFS project sites
appeared as expected.

