---
title: Upgrade Team Foundation Server 2008 to TFS 2010 (and SharePoint Server 2010) - Overview
date: 2010-05-04T07:45:00-06:00
excerpt:
  This past weekend, I upgraded my Team Foundation Server (TFS) 2008 environment
  to TFS 2010. I also upgraded the TFS project sites to SharePoint Server 2010.
  Why the SharePoint upgrade? The TFS project sites previously ran on Windows
  SharePoint Services...
aliases:
  [
    "/blog/jjameson/archive/2010/05/03/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview.aspx",
    "/blog/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview.aspx",
  ]
categories: ["Infrastructure", "Development", "SharePoint"]
tags: ["Infrastructure", "Visual Studio", "TFS", "SharePoint 2010"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview.aspx"
---

This past weekend, I upgraded my Team Foundation Server (TFS) 2008 environment
to TFS 2010. I also upgraded the TFS project sites to SharePoint Server 2010.

### Why the SharePoint upgrade?

The TFS project sites previously ran on Windows SharePoint Services 3.0 (WSS v3)
-- not Microsoft Office SharePoint Server (MOSS) 2007 -- and, in fact, most of
them were originally created with TFS 2005 and WSS v2.
[Refer to [one of my previous posts](/blog/jjameson/2010/02/28/lessons-learned-moving-tfs-to-windows-server-2008-and-sql-server-2008)
for more details on the somewhat painful experience I had upgrading my TFS
environment from WSS v2 to WSS v3 (which was really my own fault).]

In the past, I have generally recommended MOSS 2007 over WSS -- due to the
numerous additional features and capabilities of MOSS 2007 over WSS -- but when
it came to TFS, I didn't see any compelling reasons for using MOSS 2007 for the
TFS project sites (unless you simply wanted to leverage an existing SharePoint
farm for your TFS projects).

In other words, with TFS 2008, you didn't get any additional functionality on
your TFS project sites by going with MOSS 2007 -- unless, of course, you
customized your project sites with MOSS 2007 features.

However, unlike TFS 2008, the new version of TFS has some great features that
are enabled via MOSS 2007 (and SharePoint Server 2010), namely _dashboards_.
While the dashboard functionality in TFS 2010 works -- to a limited extent --
with WSS 3.0 (or SharePoint Foundation 2010), you'll have to open the
corresponding Excel workbooks, rather than viewing the dashboards directly on
each TFS project site (via the Excel Services feature in MOSS 2007 and
SharePoint Server 2010).

Consequently, when it came time to upgrade to TFS 2010, I knew I also wanted to
upgrade the TFS server from WSS to SharePoint Server 2010.

### Preparing for the Upgrade

Using the
[pre-upgrade checker for SharePoint Server 2010](http://technet.microsoft.com/en-us/library/cc262231%28office.14%29.aspx)
and the
[Test-SPContentDatabase](http://technet.microsoft.com/en-us/library/ff607941%28office.14%29.aspx)
cmdlet, I confirmed my TFS project sites did not present any issues with
upgrading to the latest version of SharePoint. Fortunately, the TFS product team
did not create a custom site definition for project sites, but rather used the
out-of-the-box Team Site with a little bit of customization (e.g. custom Web
Parts and document libraries) -- which means the upgrade process to SharePoint
Server 2010 is relatively simple.

Note that in the
"[Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter)" (a.k.a.
my home lab), I use a dual server configuration for TFS -- an application-tier
server and a separate database server. The "data tier" only runs SQL Server
(Database Engine and Analysis Services), while the "application tier" runs the
other services (i.e. TFS, WSS, and Reporting Services).

While there are certainly many organizations out there that install everything
they need for TFS on a single server, I strongly recommend you "isolate" SQL
Server as much as possible in your Production environment. This makes it much
easier to monitor and troubleshoot the environment, as well as scale the
solution over time as your needs grow.

{{< div-block "note" >}}

> **Note**
>
> This recommendation really applies to most products that depend on SQL Server
> -- not just TFS. There are a few scenarios where I consider it acceptable to
> run SQL Server side-by-side with other products and technologies, but in
> general, you should try to isolate it whenever practical.
>
> I should also point out that this is just _my_ recommendation. According to
> the TFS installation guide, a single server is recommended when you have less
> than 500 users. On the other hand, I don't believe that official
> recommendation covers running SharePoint Server 2010 on the same server --
> since the SharePoint team states
> [a minimum of 8 GB of RAM for any production deployment of their latest version](http://technet.microsoft.com/en-us/library/cc262485%28office.14%29.aspx).

{{< /div-block >}}

As I've mentioned before, in the "Jameson Datacenter", BEAST is my "Production"
SQL Server instance, while CYCLOPS is the corresponding TFS server.
Consequently, when it came time to upgrade to TFS 2010, the first thing that I
did was build out two new virtual machines (BEAST-TEST and CYCLOPS-TEST) to test
the upgrade. I also learned a few things during the process of upgrading the
Test environment that made the subsequent upgrade of the Production environment
go very smoothly.

### "Migration" Upgrade

In general, I followed the TFS installation guide to perform a "migration"
upgrade of TFS. However, there were a few steps that didn't go exactly as
planned, as well as a few "shortcuts" to reduce the time and effort required for
the upgrade.

Here is the checklist I used from the TFS installation guide (**In-Place or
Migration Upgrade of Team Foundation Server on One or More Servers**), along
with my comments for each task. Note that in my
[next post](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010),
I'll provide the detailed steps for installing TFS 2010 (and SharePoint Server 2010) and upgrading a previous TFS 2008 (and WSS v3) configuration.

{{< include-html "resources-upgrade-overview/table-1.html" >}}

{{< include-html "resources-upgrade-overview/table-2.html" >}}

### Upgrading VSTS 2008 Clients for TFS 2010

Here's something I didn't see mentioned anywhere in the TFS installation
guide...

{{< div-block-start "note important" >}}

> **Important**
>
> If you need to access TFS 2010 from a VSTS 2008 client (for example, to
> continue to use the source control integration features in Expression Web 3),
> you must download and install an update:
>
> {{< reference title="Visual Studio Team System 2008 Service Pack 1 Forward Compatibility Update for Team Foundation Server 2010 (Installer)" linkHref="http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=cf13ea45-d17b-4edc-8e6c-6c5b208ec54d" >}}
>
> Refer to [KB 974558](http://support.microsoft.com/?kbid=974558) for more
> information on the compatibility update.

{{< div-block-end >}}

### Upgrading Team Foundation Build

Earlier I stated that I use a dual server configuration for TFS, but that's not
entirely correct. There is actually a third server (DAZZLER) in my environment
that performs TFS builds.

However, the only interesting part about the build server upgrade was that I had
to increase the memory on that VM from 512 MB to 1024 MB to resolve a blocking
"system check" issue when configuring the Team Foundation Build Service.

I also chose to just "bite the bullet" and install the full Visual Studio 2010
on my build server. Previously I just installed the minimum SDKs necessary to
perform a build. However, the warnings from my build server about not being able
to run my unit tests as part of the official build have finally pushed me to the
point of installing Visual Studio instead.

### Details for Upgrade Process

As mentioned throughout this post, I'll provide details about the installation,
configuration, and upgrade of TFS 2010 and SharePoint Server 2010 in my
[next blog post](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).
