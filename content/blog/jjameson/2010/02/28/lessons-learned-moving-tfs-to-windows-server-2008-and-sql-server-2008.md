---
title: Lessons Learned Moving TFS to Windows Server 2008 and SQL Server 2008
date: 2010-02-28T12:40:00-07:00
excerpt:
  I've been a bad blogger this month. Almost a month ago, I wrote a post about
  using Web standards with Microsoft Office SharePoint Server (MOSS) 2007 , but
  I noted that there would be more to come on that subject in the near future.
  Well, almost a full...
aliases:
  [
    "/blog/jjameson/archive/2010/02/27/lessons-learned-moving-tfs-to-windows-server-2008-and-sql-server-2008.aspx",
    "/blog/jjameson/archive/2010/02/28/lessons-learned-moving-tfs-to-windows-server-2008-and-sql-server-2008.aspx",
  ]
draft: true
categories: ["SharePoint", "Infrastructure", "Development"]
tags: ["WSS v3", "SQL Server", "WSS v2", "Windows Server", "Infrastructure", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/02/28/lessons-learned-moving-tfs-to-windows-server-2008-and-sql-server-2008.aspx"
---

I've been a bad blogger this month.

Almost a month ago, I wrote a post about
[using Web standards with Microsoft Office SharePoint Server (MOSS) 2007](/blog/jjameson/2010/01/30/web-standards-design-with-moss-2007-part-1),
but I noted that there would be more to come on that subject in the near future.
Well, almost a full month has passed and I still haven't posted "part 2" yet.
Sorry about that.

As I sat down to write the next post in that series, I decided to take it in a
different direction in order to make it more valuable to SharePoint developers.
Let's just say that I've been working hard on the series and I hope to have the
subsequent posts published by next weekend.

In the meantime, I wanted to share some lessons learned while performing an
upgrade in
[the Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter).

A couple of weeks ago, I started receiving notifications (from Systems Center
Operations Manager) regarding disk errors on my primary SQL Server computer
(BEAST). The situation deteriorated and I soon found that one of the four drives
in my RAID 1+0 array was offline. After a short sigh of relief that I hadn't
lost any of my data, I added the drive back to the array and initiated a rebuild
of the mirror. I also ordered a new drive to replace my backup drive and thus
have a spare in case the problematic drive failed again.

When I received the new hard drive, I decided it was probably a good time to
rebuild BEAST. As I noted in an earlier post, the server -- at least until a
week ago -- was running Windows Server 2003 and SQL Server 2005. Note that BEAST
also serves as the "data tier" for my instance of Visual Studio Team Foundation
Server 2008. The "application tier" runs on a VM (CYCLOPS) on a different
server.

Prior to initiating the rebuild, I found the following article on MSDN:

{{< reference title="TFS 2005: How to: Move Your Data Tier to another computer"
linkHref="http://support.microsoft.com/kb/955601" >}}

Note that the article is for TFS 2005, but based on my knowledge of the
differences between TFS 2005 and TFS 2008, it seemed reasonable that the steps
would be very similar, if not completely identical.

Also note that what I was trying to achieve with the rebuild was certainly
"within the spirit" of this KB article, but wasn't strictly inline with what it
prescribes. In other words, my goal was to take my TFS instance down while I
rebuilt the backend data tier (using the same hardware but with the latest
version of Windows Server and SQL Server). After restoring the backend SQL
Server databases, I expected to be able to start up my TFS instance again
without issue. I quickly discovered that expectation would not be met (at least
not without a lot more work than I originally anticipated).

After ensuring that I had full backups of all SQL Server databases (as well as a
backup of my [BackedUp](/blog/jjameson/2007/03/22/backedup-and-notbackedup)
folder) on BEAST, I proceeded to rebuild the server with Windows Server 2008 R2.
Note that I didn't find anything that stated TFS 2008 is supported on Windows
Server 2008 R2 -- but then again, I didn't find anything that stated it wasn't
supported on R2, either. It's not the kind of risk that I would recommend to a
customer, but this is just my home lab after all.

I was less concerned with upgrading the version of SQL Server during the rebuild
process, because in my experience the SQL Server team has done a great job in
the area of restoring databases from one version to another.

Note that while my TFS instance was originally built with Team Foundation Server
2005, I upgraded this to TFS 2008 a couple of years ago and later applied TFS
SP1 as well. This is important to note because in TFS 2008, there is no separate
install for the data tier (hence my first deviation from KB 955601). I also
encountered an issue when trying to manually process the **TfsWarehouse**
database, but I'm getting ahead of myself, so I'll cover more about that in a
moment.

After installing SQL Server 2008 SP1 and restoring my TFS databases, I found
that most everything worked as expected in TFS except for the reports.

Note that I didn't attempt to restore the **TfsWarehouse** Analysis Services
(OLAP) database from a backup, but rather rebuilt it using the
[**SetupWarehouse** utility](http://msdn.microsoft.com/en-us/library/ms400783.aspx),
as prescribed in the aforementioned KB article:

{{< console-block-start >}}

SetupWarehouse.exe -o -s beast -d TfsWarehouse -c warehouseschema.xml -a
TECHTOOLBOX\svc-tfs -ra TECHTOOLBOX\svc-tfsreports -mturl http://cyclops:8080

{{< console-block-end >}}

This command completed successfully. However, when I attempted to process the
**TfsWarehouse** OLAP database, I encountered an error that stated the service
account that I had configured for Analysis Services (TECHTOOLBOX\svc-sql-as)
could not login to the **TfsWarehouse** relational database.

After digging around a little, I discovered that the data source configured by
TFS for populating the OLAP database (i.e. **TfsWarehouseDataSource**) is
configured with the following security setting:

- **Impersonation Info: ImpersonateServiceAccount**

After ensuring the **TECHTOOLBOX\svc-sql-as** service account was added to the
**TfsWarehouseDataReader** role in the **TfsWarehouse** relational database, I
continued getting the error when trying to process the OLAP database. I even
attempted to process the TFS data warehouse by invoking it from Warehouse
Controller Web service (i.e.
[http://localhost:8080/warehouse/v1.0/warehousecontroller.asmx](http://localhost:8080/warehouse/v1.0/warehousecontroller.asmx))
but no luck there either.

I found that if I changed the **Impersonation Info** setting on
**TfsWarehouseDataSource** to **ImpersonateAccount** (a.k.a. "**Use a specific
user name and password**") and entered the credentials for the TFS reporting
service (TECHTOOLBOX\svc-tfsreports) -- which is automatically configured as a
member of the **TfsWarehouseDataReader** role in the **TfsWarehouse** relational
database -- then the error no longer occurred.

However, I wasn't comfortable with leaving this change in place because it might
cause other issues in the future, and it almost certainly puts the TFS
installation into the dreaded "unsupported" category.

At that point, I decided to try a different approach.

Rather than simply rebuilding the data tier, I decided to rebuild the entire TFS
instance (i.e. data tier and application tier) by following the steps in the
following MSDN article:

{{< reference
title="How to: Move Your Team Foundation Server from One Hardware Configuration to Another"
linkHref="http://msdn.microsoft.com/en-us/library/ms404869(VS.80).aspx" >}}

Well, you can probably imagine what happened...

Yep...same error. Unable to process the OLAP database due to the following
error:

{{< log-excerpt >}}

Log Name: Application\
Source: TFS Warehouse\
Event ID: 3000\
Task Category: None\
Level: Error\
Keywords: Classic\
User: N/A\
Computer: cyclops.corp.technologytoolbox.com\
Description:\
TF53010: The following error has occurred in a Team Foundation component or
extension:\
Machine: CYCLOPS\
Application Domain: /LM/W3SVC/453528946/ROOT/Warehouse-2-129116740741730815\
Assembly: Microsoft.TeamFoundation.Warehouse, Version=9.0.0.0, Culture=neutral,
PublicKeyToken=b03f5f7f11d50a3a; v2.0.50727\
Process Details:\
Process Name: w3wp\
Process Id: 2660\
Thread Id: 3120\
Account name: TECHTOOLBOX\svc-tfs\
\
Detailed Message: Create OLAP failed\
Exception Message: Internal error: The operation terminated unsuccessfully.\
Internal error: The operation terminated unsuccessfully.\
...\
OLE DB error: OLE DB or ODBC error: Login failed for user
'TECHTOOLBOX\svc-sql-as'.; 42000.\
Errors in the high-level relational engine. A connection could not be made to
the data source with the DataSourceID of 'TfsWarehouseDataSource', Name of
'TfsWarehouseDataSource'.\
Errors in the OLAP storage engine: An error occurred while the dimension, with
the ID of 'File', Name of 'File' was being processed.\
Errors in the OLAP storage engine: An error occurred while the 'File Extension'
attribute of the 'File' dimension from the 'TfsWarehouse' database was being
processed.\
...\
(type OperationException)\
\
Exception Stack Trace: at
Microsoft.AnalysisServices.AnalysisServicesClient.SendExecuteAndReadResponse(ImpactDetailCollection
impacts, Boolean expectEmptyResults, Boolean throwIfError)\
at Microsoft.AnalysisServices.AnalysisServicesClient.Process(IMajorObject obj,
ProcessType type, Binding source, ErrorConfiguration errorConfig,
WriteBackTableCreation writebackOption, ImpactDetailCollection impact)\
at Microsoft.AnalysisServices.Server.Process(IMajorObject obj, ProcessType
processType, Binding source, ErrorConfiguration errorConfig,
WriteBackTableCreation writebackOption, XmlaWarningCollection warnings,
ImpactDetailCollection impactResult, Boolean analyzeImpactOnly)\
at Microsoft.AnalysisServices.Server.SendProcess(IMajorObject obj, ProcessType
processType, Binding source, ErrorConfiguration errorConfig,
WriteBackTableCreation writebackOption, XmlaWarningCollection warnings,
ImpactDetailCollection impactResult, Boolean analyzeImpactOnly)\
at Microsoft.AnalysisServices.ProcessableMajorObject.Process(ProcessType
processType, ErrorConfiguration errorConfiguration, XmlaWarningCollection
warnings)\
at Microsoft.AnalysisServices.ProcessableMajorObject.Process(ProcessType
processType)\
at
Microsoft.TeamFoundation.Warehouse.OlapCreator.ProcessOlapNoTransaction(Boolean
schemaUpdated, UpdateStatusStore updateStatus, Server server, SqlTransaction
transaction)\
at Microsoft.TeamFoundation.Warehouse.OlapCreator.CreateOlap(WarehouseConfig
whConf, String accessUser, String[] dataReaderAccounts, Boolean dropDB, Boolean
processCube)

{{< /log-excerpt >}}

...along with the corresponding error on the data tier:

{{< log-excerpt >}}

Log Name: Application\
Source: MSSQLServerOLAPService\
Event ID: 3\
Task Category: (289)\
Level: Error\
Keywords: Classic\
User: N/A\
Computer: beast.corp.technologytoolbox.com\
Description:\
Error during OLE DB operation. Error Code = 0xC1060000, External Code =
0x80040E4D: Login failed for user 'TECHTOOLBOX\svc-sql-as'. 42000.

{{< /log-excerpt >}}

At this point, I suspected a bug due to the fact that I was using Windows Server
2008 R2 on my backend SQL Server, because I had carefully followed each step in
the
[TFS install guide](http://www.microsoft.com/downloads/details.aspx?FamilyId=FF12844F-398C-4FE9-8B0D-9E84181D9923&displaylang=en)
for a dual-server deployment. I then rebuilt BEAST again, but this time used
Windows Server 2008 (instead of Windows Server 2008 R2).

Fast forward a couple of hours and -- yep, you guessed it -- same error. Crikey!

However, at this point, I firmly believed that I was in a "supported
configuration" and therefore decided to spend some more time troubleshooting the
problem. [I felt confident that reverting back to Windows Server 2003 and SQL
Server 2005 would also resolve the issue, but I viewed this as taking a step
backwards and would use it as a last resort.]

I discovered that just prior to the error in the event log shown above, the
following event was logged:

{{< log-excerpt >}}

Log Name: Application\
Source: MSSQLSERVER\
Event ID: 18456\
Task Category: Logon\
Level: Information\
Keywords: Classic,Audit Failure\
User: TECHTOOLBOX\svc-sql-as\
Computer: beast.corp.technologytoolbox.com\
Description:\
Login failed for user 'TECHTOOLBOX\svc-sql-as'. Reason: Token-based server
access validation failed with an infrastructure error. Check for previous
errors. [CLIENT: 192.168.0.101]

{{< /log-excerpt >}}

After researching this a little bit, I suspected that SQL Server was attempting
to use Kerberos to impersonate the Analysis Services service account. Following
[KB 917409](http://support.microsoft.com/kb/917409), I then registered a Service
Principal Name (SPN) for the Analysis Services service on BEAST and ensured the
proper settings in Active Directory to enable delegation.

However, I still couldn't get it to work. At that point, I decided to just
"punt" the issue and change the service account for Analysis Services to run as
**NetworkService** instead of a domain account. Consequently, I had to add **NT
AUTHORITY\NETWORK SERVICE** to the **TfsWarehouseDataReader** role in the
**TfsWarehouse** relational database.

At this point, I confirmed that the TFS data warehouse could be successfully
updated (using the Warehouse Controller).

You might be wondering why I chose to use a domain account in the first place.
It's my understanding that this is a best practice (and I believe it's required
when configuring a cluster) and it's the way I've always done it. It was
definitely the way BEAST was configured prior to this TFS rebuild. Note that the
TFS install guide says to:

{{< blockquote "font-italic" >}}

type the name of a domain account or **NT AUTHORITY\NETWORK SERVICE** in
**Account Name** for every service.

{{< /blockquote >}}

Thus I thought that I *should* be able to use a domain account for Analysis
Services.

As a sanity check, I also found the following in
[SQL Server 2008 books online](http://msdn.microsoft.com/en-us/library/ms174905.aspx):

{{< blockquote "font-italic" >}}

You can choose to run an instance of Microsoft SQL Server Analysis Services in
the security context of many different accounts. However, we recommend that you
use a domain or local user account as the logon account for Analysis Services.

{{< /blockquote >}}

Continuing the process of validating my new installation of TFS, I created a new
test project (based on the MSF Agile template) and confirmed that I could view
the reports from the Team Explorer window in Visual Studio. I also confirmed
that I could browse to the project team site (i.e. the SharePoint site).

Note that after all this work, I was really only at the beginning of the overall
process. More specifically, I had just completed the following step:

{{< blockquote "font-italic" >}}

1. Install Team Foundation Server in the new environment and make sure it is
   operational. For detailed instructions...

{{< /blockquote >}}

I continued following the MSDN article and restored my database backups (except
for the **ReportServer** and **ReportServerTempDB** databases -- but I'll get to
that in a moment).

While subsequently attempting to attach the SharePoint content database (for the
TFS project team sites), I encountered an error because my TFS server was
previously running Windows SharePoint Services v2 but was now running WSS v3.
When I installed TFS in my "new environment" I had to first integrate SP1 into
the installation of Team Foundation Server (i.e. a prerequisite for supporting
SQL Server 2008 and Windows Server 2008). In other words, installing SP1 onto an
existing TFS 2008 doesn't upgrade the environment to WSS v3. However, installing
TFS 2008 with SP1 integrated will install WSS v3.

In order to upgrade my SharePoint content database, I restored it to a VM
running WSS v2, ran the
[Prescan.exe utility](http://www.microsoft.com/downloads/details.aspx?FamilyId=E8A00B1F-6F45-42CD-8E56-E62C20FEB2F1&displaylang=en),
and subsequently backed up the updated database. I was then able to restore the
new backup on BEAST and attach it to the WSS v3 instance running on CYCLOPS.

The last part of the move process was to restore the TFS reports. Keep in mind
that when I said earlier that I confirmed that I could view the reports, I was
referring to the test project that I created.

However, I was wary of simply overwriting the **ReportServer** database on BEAST
because my backup was from SQL Server 2005 and BEAST now had a freshly rebuilt
SQL Server 2008 instance of that database (created by the TFS install). Perhaps
it's just me being a little paranoid or some form of "analysis paralysis", but
what if the SQL Server team made changes to the Reporting Services schema or
stored procedures between SQL Server 2005 and SQL Server 2008?

Instead, what I wanted to do was simply recreate the TFS reports for all of my
various projects using the new Reporting Services instance that I had created.
Note that I had not created any custom reports that I needed to worry about
getting back.

This is when I discovered that TFS 2008 doesn't support the ability to easily
recreate the reports for a project. It appears to be a somewhat frequent request
and something that has been considered as part of the
[TFS Power Tools](http://msdn.microsoft.com/en-us/teamsystem/bb980963.aspx) but
not something that has been formally addressed by DevDiv at this point (at least
not for TFS 2008).

Downloading the process template and subsequently uploading the RDL files
appears to be a supported operation, but I have 13 TFS projects and when you
multiply this by the number of reports for each project (~20), this quickly
becomes quite a burden to perform manually.

Fortunately, I found the following KB article:

{{< reference title="How to recover the default reports in TFS 2008"
linkHref="http://support.microsoft.com/kb/2003577" >}}

Unfortunately, the tool referenced in this KB article (TFSUploadReports.exe) was
originally built for upgrading from TFS 2005 Beta 3 to TFS 2005 RTM.
Consequently, it references version 8.0.0.0 of the various
Microsoft.TeamFoundation assemblies (and my freshly rebuilt TFS server only has
version 9.0.0.0 of these assemblies). I first attempted to use
[assembly binding redirection](http://msdn.microsoft.com/en-us/library/2fc472t2%28VS.80%29.aspx)
(to force the old utility to use the new assembly versions). I succeeded in
redirecting the three assemblies that I thought would suffice, but then
encountered additional errors and therefore quickly gave up on that effort
(before attempting to dive into Fusion logging).

Instead I fired up Reflector on TFSUploadReports.exe and observed that it really
wasn't doing anything too complex -- essentially just reading the report
configuration file and then uploading the RDL files using the
[SOAP interface for SQL Server Reporting Services](http://msdn.microsoft.com/en-us/library/ms154052.aspx).

Putting aside any concerns about copyright infringement or licensing issues for
this specific scenario, I copied the code from Reflector into a new console
application project in Visual Studio, added a Web Reference to
[http://cyclops/ReportServer/ReportService2005.asmx?wsdl](http://cyclops/ReportServer/ReportService2005.asmx?wsdl),
and made a few tweaks to get the code to compile (e.g. changing
`ReportingService` references to `ReportingService2005`). After a little
debugging, I discovered that I also had to change the URL of the Web service
proxy:

```
        private static void CreateReportServerProxy()
        {
            string reportsServerUrl = GetReportsServerUrl();
            new Uri(reportsServerUrl);
            //m_proxy = new ReportingService();
            m_proxy = new ReportingService2005();
            //m_proxy.Url = reportsServerUrl;
            m_proxy.Url = reportsServerUrl.Replace(
                "ReportService.asmx",
                "ReportService2005.asmx");

            m_proxy.Credentials = m_tfs.Credentials;
        }
```

This was necessary to avoid the following error:

{{< blockquote "font-italic text-danger" >}}

System.InvalidOperationException: Client found response content type of '', but
expected 'text/xml'.\
The request failed with an empty response.

{{< /blockquote >}}

I was then able to quickly upload the reports for each of my 13 TFS projects (in
about 3-4 minutes).

I'm happy (and relieved) to say that my TFS instance is now back up and running
and seems to be no worse for wear -- given all that it's been through. In fact,
I'm actually glad to have the environment upgraded from Windows Server 2003 and
SQL Server 2005 to Windows Server 2008 and SQL Server 2008 -- as well as all the
experience that I gained from it.

So, in closing, here are some thoughts to consider:

- When performing any kind of upgrade, always be prepared for the worst (because
  you never know when you might end up rebuilding from scratch)
- There appears to be a significant difference in the way Analysis Services
  performs impersonation between Windows Server 2003/SQL Server 2005 and Windows
  Server 2008/SQL Server 2008
- Before moving your TFS from one hardware configuration to another, ensure the
  "old" environment is running the same versions of Windows Server, SQL Server,
  and Windows SharePoint Services as the "new" environment (in my case, this
  would have prevented the "Prescan" issue when attaching the SharePoint content
  database as well as the issue with recovering the Reporting Services
  databases)
- During the process of troubleshooting issues, always keep track of the changes
  you make so that you can subsequently revert them if they don't end up
  resolving the problem
- Unless you're dealing with a lab environment, don't be cheap (like me) and try
  to use the same hardware for the "old" and "new" environments (thankfully, I
  didn't have a Development team waiting on me to get TFS back up and running
  again as soon as possible)
