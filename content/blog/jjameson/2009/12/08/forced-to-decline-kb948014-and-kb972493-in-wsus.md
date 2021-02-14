---
title: "Forced to Decline KB948014 and KB972493 in WSUS"
date: 2009-12-08T00:48:00+08:00
excerpt: "In last night's post , I discussed the solution for an issue I encountered installing KB 970892 on one of my servers. Thankfully, this morning I confirmed the server no longer increments the Computers with Errors column in the daily report I receive from..."
draft: true
categories: ["Infrastructure"]
tags: ["WSUS", "Infrastructure"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/12/08/forced-to-decline-kb948014-and-kb972493-in-wsus.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/12/08/forced-to-decline-kb948014-and-kb972493-in-wsus.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In [last night's post](/blog/jjameson/2009/12/07/error-installing-kb-970892-when-reporting-services-configured-with-domain-account), I discussed the solution for an issue I encountered installing [KB 970892](http://support.microsoft.com/kb/970892) on one of my servers. Thankfully, this morning I confirmed the server no longer increments the **Computers with Errors** column in the daily report I receive from Windows Server Update Services (WSUS).

I should also mention that in order to get my WSUS report to come up completely clean, I had to decline the the following updates:

- **Description of the Windows Server Update Services 3.0 Service Pack 1 package
  **[http://support.microsoft.com/kb/948014](http://support.microsoft.com/kb/948014)
- **Windows Server Update Services 3.0 SP2 Dynamic Installer for Server Manager
  **[http://support.microsoft.com/kb/972493](http://support.microsoft.com/kb/972493)

To be perfectly frank, I am actually rather disappointed that there doesn't seem to be any alternative to declining these updates altogether.

Up until I declined these two updates, my WSUS server complained each morning that four of my servers were not 100% patched (even though I have configured a Group Policy to automatically download and install updates every morning). The four servers that it complained about are all running Windows Server 2008.

Apparently there are known issues with these two updates -- both of which are for WSUS -- that cause them to be detected for install even on servers that are not running WSUS. [I suppose to be perfectly fair, the whole point of KB972493 is to allow you to more easily install WSUS by simply enabling a server role.]

The problem with KB 972493 started after installing Windows Server 2008 Service Pack 2 (SP2), because that includes the prerequisite patch ([KB 940518](http://support.microsoft.com/kb/940518)).

My initial attempt at resolving the warnings in WSUS was to create a separate computer group under my existing **Technology Toolbox** group called **WSUS Servers**. I then moved COLOSSUS (my one and only WSUS server) to this new group and approved the aforementioned patches on that group alone.

Sadly, the current version of WSUS does not provide a way to decline updates for a specific group only. Instead the approval field is simply **Not Approved** for all computers groups except **WSUS Servers**.

Consequently, Windows Update would detect these two patches for install, but the subsequent install would refuse to actually apply the patches -- thus perpetually leaving all of my Windows Server 2008 SP2 servers that are not running WSUS with a "99%" **Installed/Not Applicable Percentage** (and also causing them to appear in the **Computers Needing Updates** columns in the daily WSUS report).

I really wish there were some other way to resolve the issue without declining the updates altogether -- but I certainly couldn't find one, and based on a number of support cases I read regarding similar issues, this seems to be the recommended solution.

