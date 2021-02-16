---
title: "Stagger the Startup of Your Virtual Machines"
date: 2010-04-16T22:50:00-07:00
excerpt: "I've mentioned before how I run two Hyper-V servers in the \" Jameson Datacenter \" (each one hosting a variety of different VMs). 
 On the rare occasion that a patch comes out on Windows Update that affects Windows Server 2008 Server Core edition (such..."
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/17/stagger-the-startup-of-your-virtual-machines.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/17/stagger-the-startup-of-your-virtual-machines.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

I've mentioned before how I run two Hyper-V servers in the "[Jameson
Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter)" (each one hosting a variety of different VMs).

On the rare occasion that a patch comes out on Windows Update that affects Windows         Server 2008 Server Core edition (such as earlier this week), I typically need to         schedule a reboot of the Hyper-V servers. (From what I recall, most patches affecting         Server Core seem to require a reboot -- which makes me glad they don't come out         very often.)

While rebooting one of the servers this morning, I was reminded how I had previously         staggered the startup of the various VMs to avoid "hammering" the hard disks by         starting all of the VMs simultaneously.

Here's a screenshot that I captured this morning to illustrate what I'm talking         about:

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/8/r%5FHyper-V%20Staggered%20Start.png"
alt="Hyper-V staggered start"
height="310" width="600"
title="Figure 1: Hyper-V staggered start" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Hyper-V%20Staggered%20Start.png)

Notice how XAVIER1 is already started, but the other VMs are waiting to start.

Even though I spread the six VMs running on ROGUE across two RAID 1 arrays (to help         mitigate I/O contention), whenever the server is rebooted, you can imagine the load         when trying to restore all of the VMs from saved state. Even worse, though, is when         the power goes out in my house (which fortunately isn't very often at all), in which         case all of the VMs have to start from scratch since Hyper-V didn't have a chance         to save the state of each VM due to the power outage. [I used to have a UPS (Uninterruptable         Power Supply), but the battery in it died a number of years ago and now I just risk         the occasional "hard" reboot.]

All of the VMs that run on ROGUE are configured with the following setting:

- **Automatic Start Action**
  - What do you want this virtual machine to do when the physical computer starts? **Always start this virtual machine automatically**

In the **Startup delay** field, I specify the different values for         each VM.

**Automatic Start Settings**

|                     Virtual Machine<br>                 |                     Startup Delay [sec]<br>                 |
| --- | --- |
|                     XAVIER1<br>                 |                     0<br>                 |
|                     BANSHEE<br>                 |                     60<br>                 |
|                     CYCLOPS<br>                 |                     120<br>                 |
|                     DAZZLER<br>                 |                     180<br>                 |
|                     DOGFOOD<br>                 |                     240<br>                 |
|                     JUBILEE<br>                 |                     300<br>                 |

In my environment, 60 seconds appears to be a sufficient in most cases. Honestly,         it's probably not enough to cover a "cold" boot of each VM (after a power outage)         but it still gives each VM time to load up most of the way before the next VM starts         accessing the disk.

