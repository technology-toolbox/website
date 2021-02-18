---
title: "Upgrade Team Foundation Server 2008 to TFS 2010 (and SharePoint Server 2010) - Overview"
date: 2010-05-04T01:45:00-07:00
excerpt: "This past weekend, I upgraded my Team Foundation Server (TFS) 2008 environment to TFS 2010. I also upgraded the TFS project sites to SharePoint Server 2010. 
 Why the SharePoint upgrade? 
 The TFS project sites previously ran on Windows SharePoint Services..."
aliases: ["/blog/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview.aspx"]
draft: true
categories: ["Infrastructure", "Development", "SharePoint"]
tags: ["Infrastructure", "Visual Studio", "TFS", "SharePoint 
		2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

This past weekend, I upgraded my Team Foundation Server (TFS) 2008 environment  to TFS 2010. I also upgraded the TFS project sites to SharePoint Server 2010.

### Why the SharePoint upgrade?

The TFS project sites previously ran on Windows SharePoint Services 3.0 (WSS  v3) -- not Microsoft Office SharePoint Server (MOSS) 2007 -- and, in fact, most  of them were originally created with TFS 2005 and WSS v2. [Refer to [one of my previous posts](/blog/jjameson/2010/02/27/lessons-learned-moving-tfs-to-windows-server-2008-and-sql-server-2008) for more details on the somewhat painful experience  I had upgrading my TFS environment from WSS v2 to WSS v3 (which was really my own  fault).]

In the past, I have generally recommended MOSS 2007 over WSS -- due to the numerous  additional features and capabilities of MOSS 2007 over WSS -- but when it came to  TFS, I didn't see any compelling reasons for using MOSS 2007 for the TFS project  sites (unless you simply wanted to leverage an existing SharePoint farm for your  TFS projects).

In other words, with TFS 2008, you didn't get any additional functionality on  your TFS project sites by going with MOSS 2007 -- unless, of course, you customized  your project sites with MOSS 2007 features.

However, unlike TFS 2008, the new version of TFS has some great features that  are enabled via MOSS 2007 (and SharePoint Server 2010), namely *dashboards*.  While the dashboard functionality in TFS 2010 works -- to a limited extent -- with  WSS 3.0 (or SharePoint Foundation 2010), you'll have to open the corresponding Excel  workbooks, rather than viewing the dashboards directly on each TFS project site  (via the Excel Services feature in MOSS 2007 and SharePoint Server 2010).

Consequently, when it came time to upgrade to TFS 2010, I knew I also wanted  to upgrade the TFS server from WSS to SharePoint Server 2010.

### Preparing for the Upgrade

Using the [pre-upgrade
checker for SharePoint Server 2010](http://technet.microsoft.com/en-us/library/cc262231%28office.14%29.aspx) and the [Test-SPContentDatabase](http://technet.microsoft.com/en-us/library/ff607941%28office.14%29.aspx)  cmdlet, I confirmed my TFS project sites did not present any issues with upgrading  to the latest version of SharePoint. Fortunately, the TFS product team did not create  a custom site definition for project sites, but rather used the out-of-the-box Team  Site with a little bit of customization (e.g. custom Web Parts and document libraries)  -- which means the upgrade process to SharePoint Server 2010 is relatively simple.

Note that in the "[Jameson
Datacenter](/blog/jjameson/2009/09/13/the-jameson-datacenter)" (a.k.a. my home lab), I use a dual server configuration for TFS  -- an application-tier server and a separate database server. The "data tier" only  runs SQL Server (Database Engine and Analysis Services), while the "application  tier" runs the other services (i.e. TFS, WSS, and Reporting Services).

While there are certainly many organizations out there that install everything  they need for TFS on a single server, I strongly recommend you "isolate" SQL Server  as much as possible in your Production environment. This makes it much easier to  monitor and troubleshoot the environment, as well as scale the solution over time  as your needs grow.

> **Note**
>
> This recommendation really applies to most products that depend on SQL Server
> -- not just TFS. There are a few scenarios where I consider it acceptable
> to run SQL Server side-by-side with other products and technologies, but
> in general, you should try to isolate it whenever practical.
>
> I should also point out that this is just *my* recommendation. According
> to the TFS installation guide, a single server is recommended when you have
> less than 500 users. On the other hand, I don't believe that official recommendation
> covers running SharePoint Server 2010 on the same server -- since the SharePoint
> team states
> [a minimum of 8 GB of RAM for any production deployment of their latest version](http://technet.microsoft.com/en-us/library/cc262485%28office.14%29.aspx).

As I've mentioned before, in the "Jameson Datacenter", BEAST is my "Production"  SQL Server instance, while CYCLOPS is the corresponding TFS server. Consequently,  when it came time to upgrade to TFS 2010, the first thing that I did was build out  two new virtual machines (BEAST-TEST and CYCLOPS-TEST) to test the upgrade. I also  learned a few things during the process of upgrading the Test environment that made  the subsequent upgrade of the Production environment go very smoothly.

### "Migration" Upgrade

In general, I followed the TFS installation guide to perform a "migration" upgrade  of TFS. However, there were a few steps that didn't go exactly as planned, as well  as a few "shortcuts" to reduce the time and effort required for the upgrade.

Here is the checklist I used from the TFS installation guide (**In-Place
or Migration Upgrade of Team Foundation Server on One or More Servers**),  along with my comments for each task. Note that in my [next post](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010), I'll provide the detailed steps for installing TFS 2010 (and SharePoint  Server 2010) and upgrading a previous TFS 2008 (and WSS v3) configuration.

**Preparation**

| Task | Comments |
| --- | --- |
| Check for the latest installation guide. | The version of the installation guide that I used was last updated:
March 29, 2010 (TFSInstall-3292010.chm). |
| Back up your data. | I performed full backups of all "user" databases as well as a backup
of the **TfsWarehouse** OLAP database.<br><br>I also ensured that I had a backup of the encryption key for SQL Server
Reporting Services. |
| Check for supported hardware and software. | Note that TFS 2008 supported x64 on the data-tier but not on the application-tier.
Consequently, when I built out my TFS 2008 environment, I used Windows Server
2008 x86 edition.<br><br>However, SharePoint Server 2010 is x64 only and therefore cannot be installed
on Windows Server 2008 x86 edition. Consequently, in order to upgrade to
SharePoint Server 2010, I rebuilt the application-tier using Windows Server
2008 R2 (which, like SharePoint Server 2010, is only available in x64).<br><br>Note that there is a known issue with Windows Server 2008 R2 that may
effect TFS (see [KB 981898](http://support.microsoft.com/kb/981898)).
So far, I haven't experienced that issue, but if I do, I'll install the
QFE to resolve the issue. |
| Check for required permissions and user accounts. | I am using different service accounts for SQL Server, TFS, and SharePoint
Server:
<ul>
<li>TECHTOOLBOX\svc-sql</li>
<li>TECHTOOLBOX\svc-sql-agent</li>
<li>TECHTOOLBOX\svc-tfs</li>
<li>TECHTOOLBOX\svc-tfsreports</li>
<li>TECHTOOLBOX\svc-build</li>
<li>TECHTOOLBOX\svc-sharepoint</li>
<li>TECHTOOLBOX\svc-spserviceapp</li>
<li>TECHTOOLBOX\svc-web-tfs</li>
</ul>
None of the service accounts are members of the **Administrators** security group. |
| Check for supported environment settings. | The functional level of my Active Directory domain (TECHTOOLBOX) is Windows
Server 2008 mode.<br><br>I ensured the firewall ports for SQL Server were configured as necessary. |
| Set up Internet Information Services (IIS). | To reduce the time and effort required for the upgrade, I relied on the
installation of SharePoint Server 2010 to install and configure IIS.<br><br>Note that SharePoint Server 2010 installs all of the role services required
for TFS 2010 (and more). |
| Set up SQL Server.<br><br>Set up Reporting. | I installed SQL Server 2008 SP1 CU2 (with the update for KB 976761) on
the backend database server (BEAST), since this is required by SharePoint
Server 2010.<br><br>I also deleted the following obsolete SharePoint databases (since I created
a new farm with SharePoint Server 2010):<br><ul>
<li><strong>SharePoint_AdminContent_92259a02-a753-4c18-8ba8-b917d726c7df</strong></li>
<li><strong>SharePoint_Config_TFS</strong></li>
</ul><br>Note that I deferred the setup of Reporting Services until after I had
installed SharePoint Server 2010. |
| Verify SQL Server. |  |
| Prepare Portal Server. | Note that I had previously upgraded my TFS instance from WSS v2 to WSS
v3.<br><br>To upgrade my TFS projects to SharePoint Server 2010, I performed a "clean
install" and then attached the content database with the existing TFS project
sites.<br><br>(more detail provided in my next blog post) |
| Team Foundation Server Administrator Fills out Worksheet. | In my environment, the "TFS Administrator" and the "SharePoint Administrator"
are one and the same (i.e. me).<br><br>Refer to the "planning" section in my next blog post for specific details
on my environment. |
| Install and provision SharePoint Products. | I installed SharePoint Server 2010 using the following TechNet article
as a guide:<br><br>{{< reference title="Deploy a single server with SQL Server (SharePoint Server 2010)" linkHref="http://technet.microsoft.com/en-us/library/cc262243(office.14).aspx" >}}
<br><br><br><br>(more detail provided in my next blog post) |
| Configure Microsoft Office SharePoint Server 2007. | Jon Tsao has written a helpful blog post with details on how to configure
SharePoint Server 2010 for TFS dashboards:<br><br>{{< reference title="Configuring SharePoint Server 2010 for Dashboard Compatibility with TFS 2010" linkHref="http://blogs.msdn.com/team_foundation/archive/2010/03/06/configuring-sharepoint-server-2010-beta-for-dashboard-compatibility-with-tfs-2010-beta2-rc.aspx" >}}
<br><br><br><br>It's a little out-of-date (due to changes in SharePoint Server 2010 RTM
from Beta 2), but once I found the links to configure Excel Services and
the Secure Store Service, I was able to complete the configuration.<br><br>Unfortunately, my configuration is a little different from the configuration
Jon used and subsequently revealed (what I believe to be) a bug in SharePoint
Server 2010 with Excel Services. Refer to the following post for more details:<br><br>{{< reference title="\"The workbook cannot be opened\" Error with SharePoint Server 2010 (and TFS 2010)" linkHref="/blog/jjameson/2010/05/04/the-workbook-cannot-be-opened-error-with-sharepoint-server-2010-and-tfs-2010" linkText="http://blogs.msdn.com/jjameson/archive/2010/05/04/the-workbook-cannot-be-opened-error-with-sharepoint-server-2010-and-tfs-2010.aspx" >}}
<br><br> |
| Install and Configure Extensions. | I skipped this step because in my environment SharePoint Server 2010
is installed on the same server as TFS. |
| Administrator for SharePoint Products Fills out Worksheet. | Refer to the "planning" section in next blog post for specific details
on my environment. |
| Verify local SharePoint Products. | As instructed in the installation guide, I added the TFS service account
to the SharePoint Farm Administrators group. |
| Uninstall the previous version of Team Foundation
Server. | Since I installed TFS 2010 on a new server (VM),
I didn't have to uninstall the previous version. |
| Restore your data. | When restoring data from the Production environment to the Test environment
to perform a test of the upgrade process, I used SQL Server Management Studio
to add permissions for the corresponding test accounts.<br><br>For example, I added **TECHTOOLBOX\svc-tfs-test** to the
corresponding TFS databases and gave it the same permissions as **TECHTOOLBOX\svc-tfs**. Similarly, I added **TECHTOOLBOX\CYCLOPS-TEST$**
and give it the same permissions as **TECHTOOLBOX\CYCLOPS$**
(to grant acceess to services configured to run as **NT AUTHORITY\NETWORK
SERVICE**).<br><br>I used the SharePoint **Mount-SPContentDatabase** cmdlet
to attach the content database containing the TFS project sites and discovered
a few issues with the upgraded TFS sites (more detail provided in my next
blog post). |

**Team Foundation Server Upgrade**

| Task | Comments |
| --- | --- |
| Set up Team Foundation Server. | The only option I selected for the application-tier was **Team
Foundation Server**.<br><br>You do not need to select the option for **Extensions for SharePoint
Products and Technologies** if SharePoint Server 2010 is installed
on the same server as TFS. |
| Final configuration of Microsoft Office SharePoint Server 2007.  | In addition to configuring the enterprise application definition, I also
had to add the service account used for Excel Services to the SharePoint
content databases for the TFS Web application.<br><br>This is described in more detail in my next two blog posts. |

### Upgrading VSTS 2008 Clients for TFS 2010

Here's something I didn't see mentioned anywhere in the TFS installation guide...

> **Important**
>
> If you need to access TFS 2010 from a VSTS 2008 client (for example,
> to continue to use the source control integration features in Expression
> Web 3), you must download and install an update:
>
> {{< reference title="Visual Studio Team System 2008 Service Pack 1 Forward Compatibility Update for Team Foundation Server 2010 (Installer)" linkHref="http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=cf13ea45-d17b-4edc-8e6c-6c5b208ec54d" >}}
>
> Refer to [KB 974558](http://support.microsoft.com/?kbid=974558)
> for more information on the compatibility update.

### Upgrading Team Foundation Build

Earlier I stated that I use a dual server configuration for TFS, but that's not  entirely correct. There is actually a third server (DAZZLER) in my environment that  performs TFS builds.

However, the only interesting part about the build server upgrade was that I  had to increase the memory on that VM from 512 MB to 1024 MB to resolve a blocking  "system check" issue when configuring the Team Foundation Build Service.

I also chose to just "bite the bullet" and install the full Visual Studio 2010  on my build server. Previously I just installed the minimum SDKs necessary to perform  a build. However, the warnings from my build server about not being able to run  my unit tests as part of the official build have finally pushed me to the point  of installing Visual Studio instead.

### Details for Upgrade Process

As mentioned throughout this post, I'll provide details about the installation,  configuration, and upgrade of TFS 2010 and SharePoint Server 2010 in my [next blog post](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).

