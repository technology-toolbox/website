---
title: "Save HUGE Amounts of Disk Space by Slipstreaming Service Packs"
date: 2007-06-23T01:12:00+08:00
excerpt: "This is a little embarrassing, but I captured numerous screenshots back in April while rebuilding my SharePoint development VM, but I never got around to writing a blog post to actually share this information with anyone. Well, it's long overdue, but..."
draft: true
categories: ["My System", "SharePoint", "Development", "Infrastructure"]
tags: ["Simplify", "MOSS 2007", "
                    Core Development", "
                        Virtualization"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2007/06/23/save-huge-amounts-of-disk-space-by-slipstreaming-service-packs.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/06/23/save-huge-amounts-of-disk-space-by-slipstreaming-service-packs.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

This is a little embarrassing, but I captured numerous screenshots back in April         while rebuilding my SharePoint development VM, but I never got around to writing         a blog post to actually share this information with anyone. Well, it's long overdue,         but here it is anyway.

One day while working on my SharePoint development VM, I started getting warnings         about low disk space on the virtual hard disk (VHD). This struck me as rather odd         when you consider that at the time I created the VHD, I selected the default size         of 16 GB, and "all" that I had loaded on the VM was Windows Server 2003, SQL Server,         Visual Studio 2005, Office 2007, and Microsoft Office SharePoint Server (MOSS) 2007.         True, some of these installations are rather large, but 16 GBs? Come on now, I don't         think so. Besides, I had been using this VM for almost 5 months and I always seemed         to have plenty of free space before.

To figure out why, gradually over time, my VM managed to consume nearly all of the         available disk space, I used a utility named wdu.exe (Windows Disk Usage, I believe)         that some developer at Microsoft wrote in his or her spare time and threw up on         Toolbox to share with others. [On a side note, [http://toolbox/](http://toolbox/)         is an intranet site within Microsoft where anyone is welcome to submit useful programs         and utilities -- similar to [CodePlex](http://www.codeplex.com/) and         other open source sites, but with a much longer history.]

The following screenshot shows the breakdown of the VHD content:

![Disk usage on VM before rebuild](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Disk%20Usage-foobar.jpg "Disk usage on VM before rebuild")
Figure 1: Disk usage on VM before rebuild

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Disk%20Usage-foobar.jpg)

So, out of the 16 GB available, my [**NotBackedUp**](/blog/jjameson/2007/03/22/backedup-and-notbackedup) folder consumed a small fraction, the **Program
Files** folder consumed a little under 6 GB, and the **Windows**         folder accounted for a whopping 9 GB! Huh? Hey, wait a minute, what's going on here?!         Surely there must be a mistake.

Looking at the results a little closer, you can see that in the **Windows**         folder, the **Installer**, **ServicePackFiles**, and **            SoftwareDistribution** folders consume roughly 5 GB of space. On a typical         desktop computer with 100, 200, or even 300 GB hard drives, this doesn't seem all         that bad. However, when working with VMs (often configured with 16 GB drives), this         is huge! Especially when you consider that I currently have 21 VMs on my server         at home (I typically only have a couple running at any given time).

Since this is a development VM that I can rebuild in a matter of hours, I went ahead         and created a new VHD by copying my SysPrep'ed version of Windows Server 2003 (for         which I had slipstreamed SP1 and subsequently installed various other patches from         Windows Update several months before).

Immediately after booting up with this new "clean" VHD, I captured the following:

![Disk usage on VM with Windows Server 2003 SP1 slipstreamed](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Disk%20Usage-win2k3ee-base%20(SP1%20slipstream).jpg "Disk usage on VM with Windows Server 2003 SP1 slipstreamed")
Figure 2: Disk usage on VM with Windows Server 2003 SP1 slipstreamed

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Disk%20Usage-win2k3ee-base%20%28SP1%20slipstream%29.jpg)

Ahhhh...that looks much better. The **Windows** folder consumed a mere         1.3 GB of the 16 GB VHD. Excellent...or so I thought.

Like any responsible computing citizen, I then proceeded to use Windows Update to         install all of the latest patches, which in the case of Windows Server 2003, included         Service Pack 2 for the OS. So, after a lengthy download (actually it wasn't lengthy         at all, since I have WSUS running on one of my servers in the basement), my new         VM was up-to-date with all of the latest patches and ready for me to begin installing         SQL Server, Visual Studio, etc.

Before beginning those installs however, I thought I would take capture the hard         drive usage one more time. Here is what I saw:

![Disk usage on VM after installing Windows Server 2003 SP2](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Disk%20Usage-win2k3ee-base%20(after%20SP2%20install).jpg "Disk usage on VM after installing Windows Server 2003 SP2")
Figure 3: Disk usage on VM after installing Windows Server 2003 SP2

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Disk%20Usage-win2k3ee-base%20%28after%20SP2%20install%29.jpg)

You've got be to kidding! Just by installing Windows Server 2003 SP2 and a handful         of other critical patches, the **Windows** folder grew from 1.3 GB         to over 3 GB! That's ridiculous (again speaking from a VM perspective -- on a desktop         with hundreds of gigabytes of available space, I would not quibble over the additional         1.7 GB needed by Windows).

So, I decided to "refresh" my SysPrep'ed image of Windows Server 2003 by rebuilding         it from a slipstreamed Windows Server 2003 SP2 (I use [nLite](http://www.nliteos.com/), by the way, which works great IMHO). Here are the results after booting         up with the new VHD:

![Disk usage on VM with Windows Server 2003 SP2 slipstreamed](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Disk%20Usage-win2k3ee-base%20(SP2%20slipstreamed).jpg "Disk usage on VM with Windows Server 2003 SP2 slipstreamed")
Figure 4: Disk usage on VM with Windows Server 2003 SP2 slipstreamed

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Disk%20Usage-win2k3ee-base%20%28SP2%20slipstreamed%29.jpg)

Comparing this with the previous snapshot from the SP1 slipstreamed version (prior         to updating with SP2), you can see that the size of the **Windows**          folder is only slightly larger. Comparing this with the previous snapshot from the         SP1 slipstreamed version after installing SP2 shows a remarkable difference. No         longer do we need an additional 1.7 GB of "stuff" in the **Windows**         folder just to ensure that we have all the latest patches.

Note that there are some caveats with starting from a slipstreamed version of the         OS. The most notable is that you have to be sure to carry around a copy of the slipstreamed         Windows Server 2003 SP2 CD. Otherwise, when you go to build a new VM (which takes         about 15-20 minutes) and subsequently add specific features, such as IIS, you will         get prompted for the Windows Server 2003 SP2 CD. At this point, if you try just         using one of the "vanilla" DVDs from MSDN, it won't work.

On a related note, you have to be careful with VMs even after using a slipstreamed         version of the OS. The **Installer** folder can still grow to be very         large, especially if you happen to encounter any failed installations of Visual         Studio 2005 Service Pack 1 -- but that is a topic for another post (perhaps after         I have some pancakes with my daughter ;-)

