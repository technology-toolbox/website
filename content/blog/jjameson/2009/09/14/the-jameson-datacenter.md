---
title: "The \"Jameson Datacenter\""
date: 2009-09-14T07:53:00-06:00
excerpt: "Back in a post from February 2008, I first referred to the \"Jameson Datacenter\" while discussing one of the servers running in my basement. Since then I've referenced my home lab about a dozen times in different posts but never provided significant details..."
aliases: ["/blog/jjameson/archive/2009/09/13/the-jameson-datacenter.aspx", "/blog/jjameson/archive/2009/09/14/the-jameson-datacenter.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Infrastructure", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/14/the-jameson-datacenter.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/14/the-jameson-datacenter.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Back in a
[post](/blog/jjameson/2008/02/17/an-update-on-disk-space-usage-by-windows-vista)
from February 2008, I first referred to the "Jameson Datacenter" while
discussing one of the servers running in my basement. Since then I've referenced
my home lab about a dozen times in different posts but never provided
significant details about the various servers and corresponding configuration.

The purpose of this post is to share those details and explain some of the
reasons behind the infrastructure, since I believe my configuration represents a
suitable "development environment" for many small- and medium-size teams.

[Note that last weekend I finally got around to retrofitting my "new" house with
CAT5e cable from the second story down to the basement. Consequently I was
finally able to move the three servers out of my upstairs office and into the
basement where they belong. As expected, my office now feels a good 5 degrees
cooler during the day ;-)]

The following figure shows the physical architecture of the various computers
and network devices that comprise the "Jameson Datacenter":

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Jameson-Datacenter-2009-09-13-600x493.jpg"
alt="The \"Jameson Datacenter\" - physical architecture" height="493"
width="600"
title="Figure 1: The \"Jameson Datacenter\" - physical architecture" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Jameson-Datacenter-2009-09-13-909x747.jpg)

Note that I'm not suggesting development teams require a Windows Media Center or
an Xbox 360 (which doubles as a Media Center Extender) -- I merely included
those for the sake of completeness. [Although, personally, I believe that if you
provide your developers a relaxed environment where they can blow off some steam
for an hour or so, you shouldn't be surprised if productivity actually increases
dramatically, contrary to what you might predict ;-)]

If you've followed this blog for any significant length of time, you already
know that I'm a
[big fan of virtualization](/blog/jjameson/tags/Virtualization/default.aspx).
Notice that I dedicate two servers specifically to running Hyper-V virtual
machines (VMs). The following figure illustrates the various servers (both
logical and physical) that typically run 24x7 in my lab:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Jameson-Datacenter-Logical-(2009-09-13)-600x370.jpg"
alt="The \"Jameson Datacenter\" - logical architecture" height="370" width="600"
title="Figure 2: The \"Jameson Datacenter\" - logical architecture" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Jameson-Datacenter-Logical-%282009-09-13%29-995x614.jpg)

I should also point out that these aren't true enterprise servers -- meaning
that they don't have redundant power supplies, remote management cards, etc.
These are simply "home built" servers that I put together from components
typically purchased through [newegg](http://www.newegg.com/).

Also note that the "Jameson Datacenter" used to contain many more physical
servers. In fact, COLOSSUS was originally a
[refurbished Dell](http://www.delloutlet.com/) PowerEdge 4300 server that I
shelled out 2500 bucks ($2,500) for back in July, 2000. That's when I originally
came up with the naming convention, because the original COLOSSUS -- with its
three power supplies and six hot-swappable hard drives -- literally weighed over
100 pounds!

Less than a year later, I purchased another Dell 4300 from ebay (i.e. the
original BEAST), a PowerEdge 2300 (the original XAVIER), as well as a PowerEdge
6100 and couple of 4200s. However, I got tired of the "limited upgrade" path for
these servers (even though I only paid a couple hundred bucks for some of the
used servers) -- and even more tired of the monthly electricity bill for
powering that many servers. I subsequently started building my own.

It's pretty amazing what a few hundred dollars will get you on newegg these
days. In fact, when I somehow managed to
[kill one of my servers](http://en.wikipedia.org/wiki/Electrostatic_discharge) a
few months back, I replaced the motherboard, CPU, and memory for $260.89 (and
that included an AMD quad core processor and 8 GB of RAM). Since Microsoft isn't
paying for these servers -- well, I suppose I should say "isn't *reimbursing me*
for these servers" since my paycheck comes from Microsoft -- I typically have to
get approval from the wife before giving newegg my credit card number. But alas,
I digress...

The following table provides more detail on the various servers:

{{< table class="small" caption="Server Configurations" >}}

| <br>                    Server<br>                 | <br>                    Role(s)<br>                 | <br>                    Operating System<br>                 | <br>                    Domain<br>                 |
| --- | --- | --- | --- |
|  BANSHEE  |  E-mail server (POP3 and SMTP)  |  Windows Server 2003 Standard x64 Edition with Service Pack 2  |  TECHTOOLBOX  |
|  BEAST  |  Database server (SQL Server 2005)  |  Windows Server 2003 Enterprise x64 Edition with Service Pack 2  |  TECHTOOLBOX  |
|  COLOSSUS  |  Windows Server Update Services (WSUS)  |  Windows Server 2008 Standard x64 Edition (full installation) with Service Pack 2  |  TECHTOOLBOX  |
|  CYCLOPS  |  Team Foundation Server (TFS) application tier; note that databases reside on BEAST  |  Windows Server 2003 Enterprise x86 Edition with Service Pack 2  |  TECHTOOLBOX  |
|  DAZZLER  |  TFS build server  |  Windows Server 2008 Standard x64 Edition (full installation) with Service Pack 2  |  TECHTOOLBOX  |
|  DOGFOOD  |  Development server; SharePoint Server 2010; Visual Studio 2010  |  Windows Server 2008 Standard x64 Edition (full installation) with Service Pack 2  |  TECHTOOLBOX  |
|  FAB-DC01  |  Domain controller for corp.fabrikam.com; e-mail server (POP3 and SMTP)  |  Windows Server 2003 Enterprise x86 Edition with Service Pack 2  |  FABRIKAM  |
|  FOOBAR2  |  Development server; Microsoft Office SharePoint Server (MOSS) 2007; Visual Studio 2008  |  Windows Server 2008 Standard x64 Edition (full installation) with Service Pack 2  |  TECHTOOLBOX  |
|  ICEMAN  |  Hyper-V server  |  Windows Server 2008 Standard x64 Edition (core installation) with Service Pack 2  |  TECHTOOLBOX  |
|  JUBILEE  |  System Center Operations Manager 2007 R2; Root Management Server (RMS)  |  Windows Server 2008 Standard x64 Edition (full installation) with Service Pack 2  |  TECHTOOLBOX  |
|  ROGUE  |  Hyper-V server  |  Windows Server 2008 Standard x64 Edition (core installation) with Service Pack 2  |  TECHTOOLBOX  |
|  XAVIER1  |  Domain controller for corp.technologytoolbox.com  |  Windows Server 2008 Enterprise x64 Edition (full installation) with Service Pack 2  |  TECHTOOLBOX  |
|  XAVIER2  |  Domain controller for corp.technologytoolbox.com  |  Windows Server 2008 Enterprise x64 Edition (full installation) with Service Pack 2  |  TECHTOOLBOX  |

{{< /table >}}

Note that I use two different Active Directory domains (really two different
forests):

- **corp.fabrikam.com (FABRIKAM) -** I use this domain for development and
  testing purposes. In other words, whenever I want to do something
  "experimental", that may or may not be a permanent change.
- **corp.technologytoolbox.com (TECHTOOLBOX) -** I treat this as my "production"
  domain. In other words, I typically only make changes in this domain after
  I've tested them in my FABRIKAM development/test domain.

There is no trust relationship between these Active Directory domains. In other
words, I can't login to one of the TECHTOOLBOX servers with my
FABRIKAM\jjameson-admin account, nor can I login to any of the FABRIKAM servers
with my TECHTOOLBOX\jjameson-admin account. In some enterprise organizations
that I've consulted with, there actually *is* a trust relationship between the
"development" and "production" forests -- in order to avoid forcing developers
to manage multiple accounts.

Note that I utilize separate Domain Admin accounts (e.g.
TECHTOOLBOX\jjameson-admin and TECHTOOLBOX\jjameson) in order to adhere to the
principle of least privileges. In other words, when logging into my Windows 7
desktop (i.e. WOLVERINE) with my TECHTOOLBOX\jjameson account, I don't have
Administrator rights on the machine. Whenever I need to do something with
elevated privileges, I typically use either my Domain Admin account (for example
to RDP to one of the servers or modifying some secured content on another
server) or the local Administrator account (for example, when simply installing
new software or making a local configuration change).

Also note that I didn't show any of the VMs that I run on either my desktop
(i.e. WOLVERINE) or my Microsoft laptop (i.e. JJAMESON1). For example, I have
yet another development VM (FOOBAR) that I occasionally fire up on my desktop;
or, when working at a client site, I often fire up FAB-DC02 and FAB-FOOBAR on my
laptop in order to develop or demonstrate something in MOSS 2007. Yes, it's
true, FAB-DC01 and FAB-DC02 often complain about not being able to replicate
(since there obviously is no connectivity between my laptop sitting in a
customer location and the "Jameson Datacenter" running in my basement), but so
far this hasn't seemed to cause any significant issues. Of course, I can't
create any new objects (e.g. users and groups) on FAB-DC02 unless it can connect
to FAB-DC01, since FAB-DC01 holds all of the Flexible Single Master Operations
(FSMO) roles.

You might be wondering why several of the servers are still running Windows
Server 2003 and not Windows Server 2008. There are actually several reasons for
this -- which vary by server:

- As I discovered shortly after Windows Server 2008 came out, the POP3 service
  is no longer included in the operating system. I guess Microsoft simply wanted
  to deprecate this service in order to eventually terminate the corresponding
  support obligations. This shouldn't be a big deal to most people, since I
  seriously doubt many enterprise organizations -- or even small businesses --
  use the POP3 service in Windows Server 2003 for e-mail. However, I neither
  need nor want to use a full-blown instance of Microsoft Exchange simply for
  the purposes of, say, demonstrating various e-mail notifications from MOSS
  2007. Thus, BANSHEE and FAB-DC01 will probably not be moved to Windows Server
  2008 anytime soon. While there are certainly third-party POP3 alternatives out
  there, I really don't want to go learn how to install and configure them.
  Trust me, there are many more valuable ways that I can spend that time.
- BEAST is still running Windows Server 2003 because, well, I simply haven't
  seen any need to upgrade it. This server has been humming along for years and
  as the old saying goes, "if it ain't broke, don't fix it." ;-)
- CYCLOPS is still running Windows Server 2003 for essentially the same reason.
  Since I've been running Team Foundation Server on CYCLOPS since shortly after
  its release as part of Visual Studio 2005 Team System, I've never bothered to
  upgrade the operating system. Sure, I've since migrated the VM from Virtual
  Server to Hyper-V, and also upgraded to Team Foundation Server 2008, but I
  haven't seen any need to upgrade the base OS -- at least not yet.

It's also worth pointing out that while the three physical servers are running
x64 versions of the operating system, I have a mixture of x86 and x64 VMs.
Again, this can mostly be attributed to the history of the Jameson Datacenter
(i.e. Virtual Server 2005 never did support x64 VMs). Note that I originally
built out my development VMs (e.g. FOOBAR2) with x86, so that I could copy the
VHD from the Hyper-V server to my laptop and subsequently run it under Virtual
Server 2005 or Virtual PC. However, I discovered that while this should work in
theory, I never had any success with it. I used to take VMs from my Hyper-V
server and run them on my laptop back when it was running Windows Server 2008,
but since my laptop has moved to Windows 7, that is no longer an option.

[I tried removing the Hyper-V Integration Services and configuring the VM to
redetect the HAL before attempting to boot under Virtual PC, but the VM simply
hung (repeatedly) early in the boot cycle. Thus, I simply decided to "punt" and
concede that Hyper-V VMs will forever be Hyper-V VMs.]

The following figure provides more detail on the hardware configuration of the
various servers:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Jameson-Datacenter-Hardware-(2009-09-13)-600x454.jpg"
alt="The \"Jameson Datacenter\" - hardware configuration" height="454"
width="600"
title="Figure 2: The \"Jameson Datacenter\" - hardware configuration" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Jameson-Datacenter-Hardware-%282009-09-13%29-890x674.jpg)

Since BEAST is essentially my "production" SQL Server instance, I chose to
configure all four disks in a RAID 10 (1+0) configuration. Ideally, I'd throw
another four drives into that server -- or heck, even eight drives if the case
would actually hold that many -- in order to separate my data I/O from my
transaction log I/O, but, alas, the motherboard on BEAST only has four SATA II
connectors. I suppose I might get faster throughput if I actually split the four
disks into two RAID 1 arrays, but honestly, I doubt I really need it. If I
planned on using my home lab for performance testing, this might be worthwhile,
but since I rarely stress this SQL Server, I'm not going to bother. As you can
see from the dual proc/2 GB configuration, one of these days I should probably
throw another $260.89 at it (or perhaps even less) in order to replace the
motherboard, CPU, and RAM in order to make BEAST look more like ROGUE.

The differences in disk configuration between ICEMAN and ROGUE reflect the
trade-off I mentioned before. When I rebuilt ROGUE about six months ago (and
also upgraded to 8 GB), I decided to split the previous RAID 10 configuration
into two RAID 1 arrays. This allows me to place some VHDs on the C: drive and
others on the D: drive -- thus dedicating different spindles to different
workloads. I can tell you this definitely makes a difference because
occasionally JUBILEE (my SCOM 2007 server that monitors all of the other
servers) notifies me that the disk latency on one or more of the VMs running on
ICEMAN exceeded the specified threshold. As I've stated
[before](/blog/jjameson/2007/06/24/performance-of-virtual-machines), in this day
of incredibly fast CPUs, multiple cores, and gobs of RAM, you're most likely
going to bottleneck on disk I/O first.

So, there you have it...the "Jameson Datacenter" in all its gory detail -- or at
least enough detail to give sufficient context whenever I refer to one of the
servers in other posts. Hopefully you've also picked up some good tips and
recommendations for setting up a robust development environment for your
organization.

