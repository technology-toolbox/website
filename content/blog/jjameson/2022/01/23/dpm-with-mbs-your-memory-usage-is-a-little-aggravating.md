---
title: "DPM w/ MBS, your memory usage is a little aggravating."
date: 2022-01-23T13:50:44-07:00
categories: ["DPM", "Infrastructure", "System Center"]
description:
  If you use Modern Backup Storage in DPM (and you should) be prepared to
  allocate lots of "extra" RAM.
images:
  [
    "https://assets.technologytoolbox.com/screenshots/80/073325A403A2A17B98EED04755E5703A7A32B580.png",
  ]
tags: ["Infrastructure"]
---

This morning
[I poked a little fun at Dell](/blog/jjameson/2022/01/23/dell-your-feedback-feature-is-a-little-aggravating/)
for a frustrating user experience I encountered on their website. I decided to
continue in that vein -- mostly for cathartic reasons -- to share a frustrating
issue with Microsoft System Center Data Protection Manager (DPM).

Take a look at the screenshot below -- which illustrates the memory usage on the
Technology Toolbox DPM server:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/80/073325A403A2A17B98EED04755E5703A7A32B580.png"
  class="screenshot" height="863" width="1718"
  caption="Figure 1: Memory utilization on DPM server (September 2021)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/80/073325A403A2A17B98EED04755E5703A7A32B580.png)

Note that I captured this screenshot back in September of last year (when I
spent some time investigating the issue and trying to improve the situation). It
shows the typical memory usage for the VM over a 24-hour period.

Here's the corresponding screenshot I just captured from System Center
Operations Manager:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/7A/7BBA8A73266F64691CCF7C994DD66913BE206F7A.png"
  class="screenshot" height="863" width="1720"
  caption="Figure 2: Memory utilization on DPM server (January 2022)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/7A/7BBA8A73266F64691CCF7C994DD66913BE206F7A.png)

While there are some noticeable differences in the memory utilization over a
typical day (specifically, the "sawtooth" pattern throughout the day), the
general behavior is roughly the same. By that I mean the DPM server has plenty
of available RAM for most of the day, but each night the memory consumption by
the nonpaged pool increases dramatically for an extended period of time (i.e.
several hours) -- almost to the point where the server runs out of memory.

During my investigation, I identified the large memory spike is due to the ReFS
driver. I believe this is ultimately caused by DPM/ReFS removing old backups
(i.e. deleting a large number of backup files). Note that
[Modern Backup Storage (MBS) -- introduced in DPM 2016 -- stores backups on ReFS disks](https://docs.microsoft.com/en-us/system-center/dpm/add-storage?view=sc-dpm-2019).

> **Note:** In case you are wondering how I identified the ReFS driver as the
> culprit, I captured a memory dump of the VM during one of the instances of
> "low memory" and used the
> [`!poolused`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-poolused)
> extension in WinDbg to display the memory usage summary. I subsequently
> confirmed this using the
> [PoolMon](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/poolmon)
> utility.

If, like me, you've been running DPM for a number of years, you've probably come
across one or more of the following items on the Internet:

- {{< reference title="REFS issues (server lockups, high CPU, high RAM)" linkHref="https://forums.veeam.com/veeam-backup-replication-f2/refs-4k-horror-story-t40629.html" >}}

- {{< reference title="High memory usage - DPM 2016 with Modern Backup Storage" linkHref="https://social.technet.microsoft.com/Forums/en-US/61241d24-040b-4b0e-92ed-fe3886407538/high-memory-usage-dpm-2016-with-modern-backup-storage?forum=dataprotectionmanager" >}}

- {{< reference title="DPM 2016 MBS Performance downward spiral" linkHref="https://social.microsoft.com/Forums/Windows/en-us/7e4e4da4-1168-46cd-900f-9ca2bc364d5a/dpm-2016-mbs-performance-downward-spiral" >}}

- {{< reference title="FIX: Heavy memory usage in ReFS on Windows" linkHref="https://docs.microsoft.com/en-us/troubleshoot/windows-server/backup-and-storage/fix-heavy-memory-usage-refs" >}}

- {{< reference title="ReFS volume using DPM becomes unresponsive on Windows Server 2016" linkHref="https://docs.microsoft.com/en-us/troubleshoot/windows-server/backup-and-storage/refs-volume-dpm-unresponsive" >}}

- {{< reference title="How To Optimize ReFS Performance with System Center Data Protection Manager?" linkHref="https://charbelnemnom.com/how-to-optimize-refs-performance-with-system-center-data-protection-manager-windowsserver-refs-scdpm-dpm/" >}}

Most, if not all, of these resources ultimately point you towards implementing
some registry tweaks in order to force ReFS to use less memory. Well, I wish I
could tell you that approach worked for Technology Toolbox.

If memory serves, I spent several hours that week trying out various registry
changes to reduce the nightly memory spikes in the nonpaged pool. None of them
helped -- and, yes, I did reboot the VM after making each registry change (and
subsequently waited until the following day to inspect the memory usage pattern
overnight).

In the end, I simply decided to "punt" the issue and increased the RAM for the
VM from 6 GB to 8GB. Since then, the DPM server has been running smoothly.

How much RAM will _your_ DPM server need? Well, I suspect that depends on the
particular workload you have configured. For Technology Toolbox, DPM is
currently configured with the following protection groups:

- **Clients - Gold** (synchronizes the
  [BackedUp](/blog/jjameson/2007/03/22/backedup-and-notbackedup/) folders on a
  few Windows clients every hour; recovery points configured every 2 hours
  starting at 8:00 AM; retention range is 10 days)

- **Critical Files** (synchronizes all file server volumes every 30 minutes;
  recovery points configured every 2 hours starting at 7:00 AM; retention range
  is 10 days)

- **Domain Controllers** (backs up the complete "System State" of domain
  controllers everyday at 8:00 PM; retention range is 5 days)

- **Hyper-V** (backs up 59 different VMs -- as well as the "Host Component" for
  each hypervisor cluster node -- everyday starting at 11:00 PM; retention range
  is 5 days)

- **SQL Server Databases** (backs up 42 databases on three servers -- every 15
  minutes with an "Express Full Backup" at 6:00 PM everyday; retention range is
  10 days)

- **SQL Server Databases (TEST)** (backs up 20 databases on a "test" SQL Server
  instance -- every 4 hours with an "Express Full Backup" at 7:00 PM everyday;
  retention range is 10 days)

Adding them all up, this results in approximately 2,700 jobs in DPM each day
(which I suspect is on the low end compared to large organizations).

I vaguely recall one of the posts I came across previously mentioning a server
with 384 GB of memory. Yikes! (Unfortunately, I was unable to find a link for
that in the quick search I just ran.)
