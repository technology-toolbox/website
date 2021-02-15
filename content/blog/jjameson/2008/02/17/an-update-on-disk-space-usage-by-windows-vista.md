---
title: "An Update on Disk Space Usage by Windows Vista"
date: 2008-02-17T13:35:00+08:00
excerpt: "Today I rebuilt my laptop to allow me to dual boot between Windows Server 2008 and Windows Vista. To enable this configuration, I created the following partions on my hard drive: 
 
 20 GB - Windows Server 2008 x64 
 20 GB - Windows Vista Ultimate..."
draft: true
categories: ["Infrastructure"]
tags: ["Windows Vista"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/02/17/an-update-on-disk-space-usage-by-windows-vista.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/02/17/an-update-on-disk-space-usage-by-windows-vista.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

Today I rebuilt my laptop to allow me to dual boot between Windows Server 2008 and         Windows Vista. To enable this configuration, I created the following partions on         my hard drive:

- 20 GB - Windows Server 2008 x64
- 20 GB - Windows Vista Ultimate x64
- 53.2 GB - Program Files and Data

Here is what my Vista partition looked like immediately after the baseline installation         (no patches):

![Disk usage on Windows Vista Ultimate x64 (baseline)](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Disk%20usage%20-%20Vista%20Ultimate%20x64%20(baseline).jpg "Disk usage on Windows Vista Ultimate x64 (baseline)")
Figure 1: Disk usage on Windows Vista Ultimate x64 (baseline)

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Disk%20usage%20-%20Vista%20Ultimate%20x64%20%28baseline%29.jpg)

From this, you can see that about 10 GB of the 20 GB partition is needed to install         Windows Vista Ultimate x64.

In order to free up some substantial disk space, I dialed the paging file down from         the default (around 5 GB) to an initial size of 200 MB with a maximum size of 512         MB. I have 4 GB of RAM on this laptop -- so in my mind, if I start needing more         than 500 MB of swap space, something is horribly wrong and it's probably time for         a reboot anyway.

I then proceeded to install the following:

- Windows Vista Service Pack 1 (SP1)
- Microsoft Office Ultimate 2007 -- mostly using the default options but excluding
  Groove
- Microsoft Office Communicator 2007
- Microsoft Visio Professional 2007
- Microsoft Project Professional 2007
- Microsoft Visual Studio Team System 2008 Team Suite -- mostly using the default
  options but excluding Crystal Reports and including x64 compilers and tools
- Microsoft Expression Blend
- Microsoft Expression Web

Note: During each setup, I changed the drive letter from C: to E: in order to place         the bulk of the program files on the large 53 GB partition -- not the "small" 20         GB partition.

I then configured Vista to use my WSUS (Windows Server Update Services) server in         the ["Jameson
Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a. my basement), so that patches only need to be copied         over my LAN -- not downloaded from the Internet each time.

Windows Update then proceeded to install the following:

- Definition Update for Windows Defender - KB915597 (Definition 1.27.6677.0)
- Hold Em Poker Game -- I guess I am the only one to blame for this one, since I am
  the WSUS administrator and therefore the one who approves or declines all patches
- BitLocker and EFS enhancements
- Windows DreamScene
- 2007 Microsoft Office Suite Service Pack 1 (SP1)
- Update for Outlook Junk Email Filter 2007 (KB943597)
- 2007 Microsoft Office Suite Service Pack 1 (SP1) -- this was applied again after
  rebooting, perhaps to pick up additional patches for Visio and Project

After rebooting a couple of times, my Vista partition looks like this:

![Disk usage on Windows Vista after installing SP1, Office 2007, VS 2008, and Expression](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Disk%20usage%20-%20Vista%20with%20SP1%20and%20products.jpg "Disk usage on Windows Vista after installing SP1, Office 2007, VS 2008, and Expression")
Figure 2: Disk usage on Windows Vista after installing SP1, Office 2007, VS 2008,
and Expression

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Disk%20usage%20-%20Vista%20with%20SP1%20and%20products.jpg)

If you look closely, you'll notice that I only have 320 MB of the 20 GB free. Ouch.

When I started the rebuild today, I thought 20 GB would be sufficient, but now I         am wishing I had used two 25 GB partitions instead of two 20 GB partitions.

While I actually contemplated "nuking" the machine again and starting out with larger         system partitions, I am going to try getting by with this configuration for a little         while. However, I don't have any expectations of running with a mere 300 MB of free         space. Instead, I am going to completely disable hibernation, thereby freeing up         an additional 3.5 GB of disk space (by deleting the hiberfil.sys).

Unfortunately, disabling hibernation (and deleting the associated hiberfil.sys file)         wasn't very straightforward in Vista. Actually, this was really the driving factor         behind this post and the one that follows (I don't expect many people to find graphical         depcitions of disk space usage to be all that interesting).

