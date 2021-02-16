---
title: "Virtual Server Issues and Recommendations for MOSS Virtual Environments"
date: 2007-06-08T22:42:00-07:00
excerpt: "One of the tasks that I completed this week was splitting our Development environment (DEV) into multiple VMs -- one SQL Server VM, one Microsoft Office SharePoint Server (MOSS) 2007 VM for the SSP, and another MOSS 2007 VM for the front-end Web server..."
draft: true
categories: ["SharePoint", "Infrastructure"]
tags: ["MOSS 2007", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/06/09/virtual-server-issues.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/06/09/virtual-server-issues.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

One of the tasks that I completed this week was splitting our Development
environment (DEV) into multiple VMs -- one SQL Server VM, one Microsoft Office
SharePoint Server (MOSS) 2007 VM for the SSP, and another MOSS 2007 VM for the
front-end Web server. Up to this point, DEV was comprised of two VMs: a domain
controller and another single-server MOSS configuration (combined SQL Server
and MOSS SSP/Web).

[As a side note -- I am not sure if I've mentioned this in a previous post
or not -- I recommend that you always use a separate VM for the domain controller.
I can't tell you how many times I've seen bugs that only surface when running
on a DC. You're not going to use that configuration in PROD, so why use it in
DEV? You can "clamp down" the VM running Active Directory to 96 MB if memory
is really tight on your Virtual Server host (e.g. your laptop). I ran that configuration
for a couple of years and performance is quite acceptable. About a year ago
I increased the memory on the domain controller to 256MB since I also run SMTP
and POP3 services on the DC so I can have basic e-mail functionality (this is
my one and only exception to isolating the domain controller).]

To be honest, until this week I had not been using DEV all that much since
I do all of my development on a different VM -- a single-server MOSS configuration
that I run on one of my servers in the Jameson Data Center (a.k.a. my basement).
I simply check in my code to source control after developing and testing it
locally.

However, during our feature walkthroughs this week, the entire team experienced
firsthand how bad the performance was in DEV. The CPU was pegged nearly all
of the time. Apparently the dual core CPU in my home server -- that I built
last year with $1,000 worth of components from Newegg.com -- is quite a bit
faster than the 8-CPU server we use to run DEV. Since each VM can only use a
single CPU, we all sat idly by waiting for the VM to respond to my mouse clicks.
[By the way, the walkthrough wasn't the first hint that performance was really
bad in DEV. I noticed that the last deployment to DEV required almost 3-1/2
hours to complete, whereas the same process typically takes a little over an
hour on my local VM (deployment involves creating a couple of Web applications,
creating and configuring an SSP, and building out roughly 180 site collections).]

In order to leverage the 8 processors on the DEV server, we needed to create
separate VMs for SQL, MOSS Web, and MOSS SSP. We'll probably bottleneck on disk
I/O after this change (due to the fact that the DEV server only has one drive
we can use for VMs) but perf will certainly be better than it was before.

During the process of building out the new VMs, I encountered a couple of
problems related to Virtual Server. In order to install SQL Server and MOSS
2007 on the new VMs, I downloaded the ISO images from MSDN to the Virtual Server
host. However, when I attached the SQL Server ISO image to the new VM, I encountered
the following error:

{{< blockquote "font-italic text-danger" >}}

setup.exe - Bad Image

The application or DLL D:\SQL Server x86\Servers\Setup\setupex.dll is not
a valid Windows image. Please check this against your installation disk.

{{< /blockquote >}}

Assuming that I had somehow downloaded a corrupt ISO image from MSDN, I went
ahead and downloaded it again. However, even with the freshly downloaded ISO,
I got the same error.

I then tried installing VS 2005 on the new SSP VM as well as the new front-end
Web server VM (figuring I could deal with the SQL issue later before running
the SharePoint Configuration Wizard). On both VMs, I got the following error
when starting the setup:

{{< blockquote "font-italic text-danger" >}}

Visual Studio 2005 Setup

An unknown error occured while copying files to your temporary folder. Setup
will now exit.

{{< /blockquote >}}

The last thing that I could think of was to copy the files from the "CD"
(i.e. the mounted ISO file) to the local VHD. I opened a command prompt and
started robocopying files, but I soon got the following error:

{{< blockquote "font-italic text-danger" >}}

Error Performing Inpage Operation

{{< /blockquote >}}

And, no, KB [141117](http://support.microsoft.com/kb/141117) was
not any help.

Ugh!

Since I had never encountered these problems before when building out numerous
other VMs, I figured the problem had to be with the Virtual Server host itself
(i.e. the 8-proc server that runs the VMs for DEV). Therefore, I resorted to
robocopying the VMs locally to my laptop (actually an external USB drive connected
to my laptop), where I was able to successfully install SQL Server, MOSS, and
Visual Studio 2005). I subsequently copied them back to the server.

[Another side note: if you are developing MOSS solutions on a laptop, I highly
recommend that you use an external 7200 RPM USB drive for the VHD files. A few
months ago, I started running my MOSS 2007 development VM on my laptop -- I
had previously been running it exclusively on my home server -- and I noticed
a huge impact on my productivity. The disk churn was unbearable. I tried running
it on both the internal hard drive as well as an external 5400 RPM drive. Performance
in both of these cases was unacceptable. I sent a message to one of our internal
SharePoint DLs inquiring if others were suffering a similar slow death that
I was. I suspected that the answer was to get a faster drive, but I wanted to
get some evidence before placing the order.

Several people indicated that they lugged around full-size external USB hard
drive enclosures in order to get acceptable perfomance. I've done that before
-- I am not doing it again. I ordered a Seagate ST910021U2-RK 100GB 7200 RPM
External Hard Drive and I've been quite pleased with the performance ever since.
Sure, my home server with 4 drives in a RAID10 configuration is still much faster,
but it is much, much better than it was.]

Anyway, back to the Virtual Server issues...I could not understand why there
would be problems mounting certain ISO images on VMs running on the DEV server
(before robocopying the VMs to my laptop, I was able to mount my slipstreamed
Windows Server 2003 SP2 ISO image in order to install IIS, so I knew the problem
wasn't with all ISO images).

I had encountered some strange errors in this Virtual Server environment
before. Most notably, when I would robocopy a non-trivial amount of content
from the Virtual Server host to one of the VMs, it seemed like the network adapter
would get "saturated" -- if I was connected via Terminal Services to the VM
then my connection would disconnect, or if I was using the VMRC then I would
get "The network path was not found" errors soon after the starting the copy.

I was starting to think that perhaps there was just something fundamentally
wrong with the DEV server and it was time for a complete rebuild of the server.
However, I figured that I would first try reinstalling Virtual Server.

Before I uninstalled Virtual Server, I made a note of the version that was
previously running in DEV (somebody else had initially configured this server
long ago). The Virtual Server Administration Website displayed: **1.1.465.0
SE**.

I removed Virtual Server and then downloaded the latest version from microsoft.com
(it is now a
[free download](http://www.microsoft.com/technet/virtualserver/software/default.mspx)). The Virtual Server Administration Website now displays:
**1.1.465.292 EE R2**.

You know what's coming next...

All of the problems appear to be resolved! After upgrading to Virtual Server
2005 R2, I can mount the ISOs and start the installation without getting the
errors I encountered before. It also appears that the network adapter "saturation"
problem is fixed as well (I was able to robocopy Visual Studio 2005 SP1 -- which
I definitely consider to be a "non-trivial" amount of content).

Bottom line: it appears that quite a few bugs were fixed in Virtual Server
2005 R2. I highly recommend that you verify that all of your VM environments
are running the latest version. I have been running R2 on my laptop as well
as in the Jameson Data Center for so long that I never encountered these problems
before.

