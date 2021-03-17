---
title: "Performance of Virtual Machines"
date: 2007-06-24T08:52:00-06:00
excerpt: "Yesterday I had to rebuild our Test environment (TEST) to replace a VM (running on VMWare) with a physical server, due to very poor performance of the VM. 
 When I was copying files from the VM to a different server in order to deploy our solution, robocopy..."
aliases: ["/blog/jjameson/archive/2007/06/23/performance-of-virtual-machines.aspx", "/blog/jjameson/archive/2007/06/24/performance-of-virtual-machines.aspx"]
draft: true
categories: ["Infrastructure"]
tags: ["Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/06/24/performance-of-virtual-machines.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/06/24/performance-of-virtual-machines.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Yesterday I had to rebuild our Test environment (TEST) to replace a VM (running
on VMWare) with a physical server, due to very poor performance of the VM.

When I was copying files from the VM to a different server in order to deploy
our solution, robocopy reported throughput of 13.418 MegaBytes/min. Yes, that
number really is [MB/min]. It took almost 10 minutes to copy the 120 MB that
comprises the entire build. Ouch.

Just to make sure that I was not completely losing it, I performed the copy
again. The second time throughput doubled to 28.313 MB/min. Wow. Impressive.
[That's sarcasm, in case you missed it.]

About a month ago, I pointed out that our document migration process appeared to
be suffering from a network bottleneck since the CPU, memory, and disk metrics
on the front-end Web servers and backend database server were all well below the
performance threshold values.

[Side note: Months ago, we compromised on our infrastructure design where a
single 100 Mbps network adapter on each of the front-end Web servers serves all
traffic except for backups -- rather than having a separate "front-end" network
(for Web requests) and backend network (for SQL, AD, and other "infrastructure"
traffic). In other words, all network packets, regardless of whether they are
HTTP requests from Web clients or TDP requests to read from or write to SQL
Server, flow through a single NIC. Therefore I have long suspected that our
first bottleneck would be the network.]

During subsequent testing,
[Windows Server 2003 Performance Advisor](http://www.microsoft.com/downloads/details.aspx?FamilyID=09115420-8c9d-46b9-a9a5-9bffcd237da2&DisplayLang=en)
reported a high number of TCP retransmits during the document migration. In
order to troubleshoot the issue with a network engineer, we used a simple
robocopy operation to examine the network throughput and found that we weren't
seeing anywhere near the maximum throughput on a 100 Mbps network. We discovered
that the duplex setting on the network switch was not set correctly. After
correcting the setting on the switch, throughput went up dramatically.

Therefore, on Friday when I observed the horrible throughput numbers when
robocopying files in TEST, I immediately suspected that we had a network problem
again. When we called the help desk to have someone look at the network
utilization, they stated that the network was "fine" and suggested that the
problem was due to VMWare -- not the network.

At this point, the VMWare support team was engaged to investigate the problem.
They pointed out that the CPU usage on our VM was 13%, while memory was only 7%.
In other words, they only looked at two of the four resources that ultimately
bottleneck any computing operation.

Prior to the horrible throughput numbers I experienced on Friday when copying
files, I had noted the performance of that particular VM has been an issue for a
couple of months, and stated that it was either misconfigured or else the host
VMWare server was overloaded.

I have seen many VM environments where organizations use a high-end server with
4 or 8 state-of-the-art processors and "gobs" of memory and subsequently assume
that they can run 4 or more VMs on this one server without a problem. However,
as noted earlier, CPU and memory represent only half of the resources that can
ultimately become a bottleneck. In my experience, most VMs bottleneck first on
disk I/O.

Don't spend all of your money on multi-core processors and RAM, then skimp on a
single 300 GB drive (even though this is more than large enough for the VMs). If
you do, I can almost guarantee that your VM users will quickly complain about
performance. When you start looking into the issue, you'll find that the disk
queue length has "gone through the roof."
[I noted in a [previous post](/blog/jjameson/2007/06/09/virtual-server-issues)
that while I am at the customer site (i.e. when I can't use the server in my
basement) I run my local Microsoft Office SharePoint Server (MOSS) 2007
development VM on an external 7200 RPM USB drive to achieve acceptable
performance.]

Furthermore, let's suppose that you do spec your disk configuration like the
server in my basement (4x 300 GB SATA2 drives in a RAID 10 configuration). You
still need to keep in mind that, depending on the number and speed of the NICs
on the host server, performance may bottleneck on the network before CPU and
memory become an issue.

At this point we don't know why the robocopy operation reported extremely slow
performance (and honestly I don't really care anymore since we have replaced
that VM with a physical server). Honestly, I don't know much about this
particular VMWare host, but clearly there are some fundamental problems with the
configuration.

Nevertheless, this seemed like a good opportunity to share some important
information for those running VMs.
