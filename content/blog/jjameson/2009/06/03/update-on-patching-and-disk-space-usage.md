---
title: Update on Patching and Disk Space Usage
date: 2009-06-03T09:09:00-06:00
excerpt:
  About a year ago, I wrote a post about saving huge amounts of disk space by
  slipstreaming service packs . Having just been through an ordeal installing
  Windows Server 2008 SP2, I thought it would be worthwhile to provide an update
  (since that original...
aliases:
  [
    "/blog/jjameson/archive/2009/06/02/update-on-patching-and-disk-space-usage.aspx",
    "/blog/jjameson/archive/2009/06/03/update-on-patching-and-disk-space-usage.aspx",
  ]
draft: true
categories: ["Infrastructure"]
tags: ["Windows Vista", "Windows Server", "Infrastructure", "Virtualization"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/06/03/update-on-patching-and-disk-space-usage.aspx"
---

About a year ago, I wrote a post about
[saving huge amounts of disk space by slipstreaming service packs](/blog/jjameson/2007/06/23/save-huge-amounts-of-disk-space-by-slipstreaming-service-packs).
Having just been through an
[ordeal](/blog/jjameson/2009/06/01/errors-installing-windows-server-2008-sp2)
installing Windows Server 2008 SP2, I thought it would be worthwhile to provide
an update (since that original post refers to disk space usage with Windows
Server 2003).

Note that since my original post, I have switched from using Virtual Server in
favor of Hyper-V. Among other things, this allows me to run x64 virtual machines
(VMs). Many months ago, I consolidated numerous physical machines onto a couple
of "Server Core" machines running Hyper-V. In that time, I've also switched to
running Windows Vista x64 on my primary desktop and Windows Server 2008 x64 on
my laptop.

One of the things that I've noticed is that x64 versions of the operating system
tend to use more disk space than their corresponding x86 equivalents. In
particular, the "side-by-side" folder (WinSxS) is typically significantly larger
on x64 installations. The storage differences are negligible on my physical
machines, but on VMs I make a deliberate effort to "clamp down" the size of the
VHDs. This can save me considerable time when copying VHDs from one server to
another or from an internal hard drive to an external hard drive whenever I need
to take one or more of them "on the road" with me.

Minimizing VHD sizes also allows me to cram more VMs onto my 100 GB external
drive
[I know, these days this isn't very big from a capacity perspective, but at least it's 7200 RPM (a [must](/blog/jjameson/2007/06/24/performance-of-virtual-machines)
for running VMs) and it isn't nearly as bulky as my larger drive enclosure. It
also doesn't require a separate power supply either.]

Here is a baseline of the disk space usage on a Windows Server 2008 Standard x64
VM:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-usage-Windows-Server-2008-(baseline)-600x450.jpg"
alt="Disk usage on Windows Server 2008 Standard x64 VM (baseline)"
class="screenshot" height="450" width="600"
title="Figure 1: Disk usage on Windows Server 2008 Standard x64 VM (baseline)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-usage-Windows-Server-2008-%28baseline%29-800x600.jpg)

Notice that the total disk usage is about 7.5 GB and the Windows folder consumes
a little over 7 GB. Also note that Windows Server 2008 included SP1 (i.e.
Microsoft slipstreamed it into the initial installation in order to simplify the
servicing model for both Windows Server 2008 and Windows Vista).

I then immediately installed Windows Server 2008 SP2 and captured the following:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-ws2008std-x64-SP2-600x431.png"
alt="Disk usage on Windows Server 2008 x64 VM (after installing SP2)"
class="screenshot" height="431" width="600"
title="Figure 2: Disk usage on Windows Server 2008 x64 VM (after installing SP2)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-ws2008std-x64-SP2-801x576.png)

Observe that the Windows folder now consumes a little over 10 GB of storage.
Ouch...3 GB for a service pack. That seems a little, um, _irritating_ -- for
VMs, anyway. Obviously for physical machines with 100+ GB hard drives, the
additional space is trivial.

I then ran the Windows Component Clean tool (COMPCLN.exe) as described in my
[previous post](/blog/jjameson/2009/06/02/reclaiming-disk-space-after-installing-service-pack-2),
which reclaimed approximately 900 MB of space.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-WS2008-x64-(WS2008-SP2-compcln)-600x429.png"
alt="Disk usage on Windows Server 2008 x64 VM (after installing SP2 and running COMPCLN.exe)"
class="screenshot" height="429" width="600"
title="Figure 3: Disk usage on Windows Server 2008 x64 VM (after installing SP2 and running COMPCLN.exe)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-WS2008-x64-%28WS2008-SP2-compcln%29-801x573.png)

Notice that the Windows folder now consumes about 8.5 GB of space (but the
overall free space on the 20 GB VHD increased from roughly 9.7 GB to 10.6 GB).
In other words, SP2 adds roughly 3 GB, but COMPCLN.exe trims this to a little
over 2 GB.

Lastly, I want to point out the current disk space usage on COLOSSUS -- an x64
VM that I run WSUS (Windows Server Update Services) for managing patches and
updates on the various machines in the
["Jameson Datacenter."](/blog/jjameson/2009/09/14/the-jameson-datacenter) Note
that this server only has WSUS (which requires IIS and SQL Server) but nothing
else. Consequently, after installing Windows Server 2008 SP2 and running
COMPCLN.exe, I was hoping it would have comparable disk space usage to that
shown in Figure 3 (after deducting the space used by the WSUS database, of
course).

Unfortunately, it isn't even close, as shown in the following figure.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-COLOSSUS-(after-WS2008-SP2-install)-600x454.png"
alt="Disk usage on a patched WSUS server (after installing SP2)" height="454"
width="600"
title="Figure 4: Disk usage on a patched WSUS server (after installing SP2)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-COLOSSUS-%28after-WS2008-SP2-install%29-758x574.png)

Notice that the Windows folder on COLOSSUS consumes almost 16.5 GB of space, of
which roughly 10.5 GB is used by the WinSxS folder.

The lesson here is that you should expect some "bloat" in the Windows folder
over time (largely due to the WinSxS folder), and while the Windows Component
Clean tool (COMPCLN.exe) undeniably reclaims _some_ hard drive space after
installing SP2, it's definitely not the same as starting with a "fresh" SP2
install.

