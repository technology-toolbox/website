---
title: "Virtual Machine Snapshots and SharePoint Development, Part 1"
date: 2011-03-21T22:53:00-07:00
excerpt: "In a comment I made last week on one of my earlier posts , I mentioned how a few months ago I started using Hyper-V snapshots so I can quickly rollback my SharePoint development VMs to key points in time. 
 The following screenshot shows the snapshots..."
draft: true
categories: ["My System", "SharePoint", "Infrastructure"]
tags: ["My System", "MOSS 2007", "
                    Virtualization", "SharePoint
                        2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/22/virtual-machine-snapshots-and-sharepoint-development-part-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/22/virtual-machine-snapshots-and-sharepoint-development-part-1.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In a comment I made last week on [one of my earlier posts](/blog/jjameson/2011/03/11/disk-benchmarks-ssd-vs-quot-raptor-quot-vs-raid), I mentioned how a few months ago I started using         Hyper-V snapshots so I can quickly rollback my SharePoint development VMs to key         points in time.

The following screenshot shows the snapshots for my primary SharePoint Server 2010         development VM (FOOBAR5):

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/9/r%5FFOOBAR5%20Snapshots.png"
alt="FOOBAR5 snapshots"
height="383" width="600"
title="Figure 1: FOOBAR5 snapshots" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_FOOBAR5%20Snapshots.png)

For the sake of explaining these various snapshots, let me relabel them in this         post as follows:

- Baseline SharePoint Server 2010 configuration - **Snapshot 1**
  - Baseline Fabrikam Demo Site (SharePointClaimsAuthentication) - **Snapshot
    2**
  - Baseline {Client Name} Cloud Portal (Sprint-11) - **Snapshot 3**

The first snapshot was taken after installing SharePoint Server 2010 and creating         the SharePoint farm. Note that there are number of installation and configuration         steps leading up to "Snapshot 1." Here are the headings from the installation guide         to give you an idea of what's included up to that point:

- 4 Deploy and configure the server infrastructure
  - 4.1 Install Windows Server 2008
  - 4.2 Disable Internet Protocol Version 6 (TCP/IPv6)
  - 4.3 DEV - Configure domain controller
  - 4.4 Join member server to domain
  - 4.5 Create service accounts
  - 4.6 Create Active Directory container to track SharePoint 2010 installations
  - 4.7 DEV -- Map Web application to loopback address in hosts file
  - 4.8 Allow specific host names mapped to 127.0.0.1
  - 4.9 DEV -- Set MaxPatchCacheSize to 0 (Optional)
  - 4.10 DEV - Install Visual Studio 2010
  - 4.11 DEV - Install Team Explorer (Team Foundation Client)
  - 4.12 Install SQL Server 2008
  - 4.13 Install SQL Server 2008 Service Pack 2
  - 4.14 DEV -- Install TFS Power Tools
  - 4.15 DEV - Install Microsoft Office 2010
  - 4.16 DEV - Install Microsoft SharePoint Designer 2010
  - 4.17 DEV - Install Additional Service Packs and Patches
- 5 Install and configure SharePoint Server 2010
  - 5.1 Prepare the farm servers
  - 5.2 Install security update for Web applications using Claims authentication
  - 5.3 Install SharePoint Server 2010 on the farm servers
  - 5.4 Create and configure the farm
  - 5.5 Add Web servers to the farm
  - 5.6 Add SharePoint Central Administration to the Local intranet zone
  - 5.7 Add the URL for the Cloud Portal to the Local intranet zone
  - 5.8 Add the SharePoint bin folder to the PATH environment variable
  - 5.9 Grant DCOM permissions on IIS WAMREG admin Service
  - 5.10 Rename TaxonomyPicker.ascx
  - 5.11 Configure diagnostic logging and usage and health data collection
  - 5.12 Configure service accounts
  - 5.13 Configure mail services

As the actual snapshot name implies, "Snapshot 1" is essentially a baseline SharePoint         Server 2010 configuration without any Web applications (not counting Central Administration)         or service applications.

In case it's not immediately obvious, the sections prefixed with "DEV" are performed         in development environments only (since the same installation guide is used for         Test and Production environments as well). It's also worth calling out that separate         VMs are used as domain controllers for the development environment. I should also         point out that my development VM actually runs Windows Server 2008 R2 and SQL Server         2008 R2 (a slight deviation from the Test and Production environments, but one that         I'm willing to accept the risk for).

As you can imagine, it takes a couple of hours (or more, depending on how quickly         you can "fly" through the installation guide) to reach the point where "Snapshot         1" was taken. The goal, however, is not to optimize that part of the process --         since it is expected to be done very rarely (compared with other "rebuild" processes).

Now consider "Snapshot 2" which was taken after following the steps in [one of my previous posts](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010) to create and configure a sample Web application         using claims authentication (including a custom login form). "Snapshot 2" was created         yesterday morning at 7:19:27 AM. In other words, it took me about 20 minutes to:

1. Create the ASP.NET membership database
2. Configure claims authentication for Central Administration and the Security Token
   Service
3. Run the deployment scripts to create and configure the Fabrikam Demo Web application
4. Create a couple of users (and one role) in the membership database and validate
   that claims authentication works as expected

While 20 minutes isn't a lot of time to create and configure a new Web application,         taking a snapshot of the Fabrikam Demo site allows me to revert to that baseline         with almost no effort whatsoever. Essentially I just have to wait for the VM to         boot after applying the snapshot (since I tend to take snapshots after shutting         down the VM).

After creating "Snapshot 2" to baseline the Fabrikam Demo site, I then reverted         the VM back to "Snapshot 1" and proceeded to create and configure the "Cloud Portal"         environment for my current project (hence why I obscured the client name in the         screenshot above). Unfortunately, the "Cloud Portal" deployment isn't quite as automated         as the Fabrikam Demo site and therefore it took me just over an hour yesterday to         go from a "baseline SharePoint Server 2010 configuration" to a fully configured         "Cloud Portal" environment.

For the sake of understanding what is included in "Snapshot 3", here are the headings         from the installation guide:

- 6 Create and configure the Web application
  - 6.1 Set environment variables
  - 6.2 Create the Web application and initial site collections
  - 6.3 Configure claims based authentication
    - 6.3.1 Add SharePoint farm service account to \*\*\*\*\*\*\*\*\*Portal database
    - 6.3.2 Add Web.config modifications for claims based authentication
  - 6.4 Configure the People Picker to support searches across one-way trust
  - 6.5 Expand content database files
  - 6.6 Configure SSL on the Internet zone
  - 6.7 Configure object cache user accounts
  - 6.8 Enable anonymous access to the site
  - 6.9 Enable disk-based caching for the Web application
- 7 Configure service applications
  - 7.1 Configure the State Service
  - 7.2 Create and configure the Search Service Application
  - 7.3 Configure the search crawl schedules
- 8 Install and configure Office Web Apps
  - 8.1 Install Office Web Apps
  - 8.2 Run PSConfig to register Office Web Apps services
  - 8.3 Start the Office Web Apps service instances and create service applications
  - 8.4 Configure Excel Services Application trusted location
  - 8.5 Configure the Office Web Apps cache
  - 8.6 Grant access to the Web application content database for Office Web Apps
- 9 Validate basic configuration of the Web application
- 10 Deploy the Cloud Portal solution and configure the site
  - 10.1 Configure logging
  - 10.2 Install \*\*\*\*\*\*\*\*\*.CloudPortal.Web solution and activate the features
  - 10.3 Create a custom sign-in page
  - 10.4 Create and configure a client site
    - 10.4.1 Create site collection for a \*\*\*\*\*\*\*\*\* client
    - 10.4.2 Apply the "\*\*\*\*\*\*\*\*\* Client Site" template to the top-level site
    - 10.4.3 Update the client site home page
  - 10.5 Create and configure a team collaboration site
    - 10.5.1 Create the team collaboration site
    - 10.5.2 Update the team site home page

Whew! That certainly is a lot of steps to complete, but fortunately many parts are         scripted so it doesn't take as long as you might think. [To be honest, I do skip         a couple of steps -- specifically configuring SSL and the BlobCache (a.k.a. "disk-based         caching") -- since this is a development environment after all.]

There is quite a bit more I want to cover on this topic -- such as why "Snapshot         3" is not a child of "Snapshot 2" and how snapshots provide tremendous value when         deploying frequent releases (i.e. short sprints). However, I'm running short on         time this morning, so I'll tack "Part 1" onto this post and defer the rest to ["Part 2" in this series](/blog/jjameson/2011/03/23/virtual-machine-snapshots-and-sharepoint-development-part-2). Stay tuned.

