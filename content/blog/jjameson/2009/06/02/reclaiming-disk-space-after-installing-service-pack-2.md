---
title: "Reclaiming Disk Space After Installing Service Pack 2"
date: 2009-06-02T02:39:00-07:00
excerpt: "In yesterday's post , I noted the errors I encountered when trying to install Windows Server 2008 Service Pack 2 (SP2) due to \"insufficient\" disk space. I ended up having to expand numerous VHDs (one for each of my VMs running Windows Server 2008 x64..."
draft: true
categories: ["Infrastructure"]
tags: ["Windows Vista", "Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/06/02/reclaiming-disk-space-after-installing-service-pack-2.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/06/02/reclaiming-disk-space-after-installing-service-pack-2.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In yesterday's [post](/blog/jjameson/2009/06/01/errors-installing-windows-server-2008-sp2), I noted the errors I encountered when trying to install Windows Server 2008 Service Pack 2 (SP2) due to "insufficient" disk space. I ended up having to expand numerous VHDs (one for each of my VMs running Windows Server 2008 x64) in order to have roughly 5 GB of free space to allow SP2 to install.

Note that SP2 certainly didn't use the 5 GB of free space, but apparently it insists on having that much for some "factor of safety" during the install.

Note that SP2 for Windows Server 2008 and Windows Vista include a new tool which helps recover hard disk space: the Windows Component Clean Tool (COMPCLN.exe). According to the [Windows Server 2008 SP2 Deployment Guide](http://technet.microsoft.com/en-us/library/dd351467%28WS.10%29.aspx) this tool permanently removes the files that are archived after Windows Vista SP2 or Windows Server 2008 SP2 is applied. It also removes the files that were archived after Windows Vista SP1 was applied, if they are found on the system.

The deployment guide also states that "running this tool is optional" -- however I, personally, highly recommend it. Of course, you have to weigh this decision with the probability that you will need to uninstall SP2 -- which, in my case, is essentially "zilch."

On my x64 VMs, running COMPCLN.exe freed up roughly 800 MB of disk space. This might not seem like a lot -- given that many hard drives these days exceed 1 TB -- but it is very significant on VHDs that you are trying to keep as "lean" as possible.

Note that I'm still a little disappointed in the disk space requirements for SP2. Up until yesterday I had been running each of my domain controller VMs on 16 GB VHDs, and many of my other VMs on 20-22 GB VHDs. Note that on some of those VMs, disk space was certainly getting tight -- due to all of the patches that have been installed since I originally built them -- but they were functioning just fine until Windows Server 2008 SP2 came along. Although I have to say that on a 20 GB VHD, a WinSxS folder consuming 8-10 GB seems a little ridiculous.

I ended up having to expand the system VHD for my MOSS 2007 development VM to 25 GB -- which I really didn't want to do -- since it essentially gives the SharePoint Unified Logging System another 5 GB of free space to fill with [useless log messages](/blog/jjameson/2009/03/26/sharepoint-uls-logs-flooded-with-preserving-template-record-with-size). [That is, of course, until I get the April 2009 Cumulative Update installed -- which supposedly fixes this issue.]

