---
title: "Starting and Stopping Hyper-V VMs with Server Core"
date: 2009-08-13T05:21:00+08:00
excerpt: "Last week before heading out to the airport for my SharePoint 2010 training, I powered down the \"Jameson Datacenter\" (i.e. the four computers running in my home office). Since I would be gone for almost 8 full days, it didn't make sense to waste the electricity..."
draft: true
categories: ["Infrastructure", "My System"]
tags: ["Infrastructure", "Virtualization", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/08/13/starting-and-stopping-hyper-v-vms-with-server-core.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/08/13/starting-and-stopping-hyper-v-vms-with-server-core.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Last week before heading out to the airport for my SharePoint 2010 training,  I powered down the ["Jameson
Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (i.e. the four computers running in my home office). Since I would  be gone for almost 8 full days, it didn't make sense to waste the electricity just  so my SQL Server could periodically send email messages to me about backups being  completed successfully throughout the day. [Note that my wife uses only the Media  Center in the family room and her laptop to fulfill all of her computing needs (primarily  sending and receiving email, as well as surfing Facebook) and my daughter is too  young to use any computer by herself.]

The day after I got back from Redmond, I powered up the servers, but then realized  that this was the first time that I had explicitly shutdown both Hyper-V VMs that  serve as domain controllers (i.e. XAVIER1 and XAVIER2, that I referred to in a [previous post](/blog/jjameson/2008/11/05/server-core-installation-accessing-windows-in-notification-period)). Note that I had since moved one of the domain controller VMs  to a different Hyper-V server. In other words, two of the four computers in my home  office (ROGUE and ICEMAN) run Windows Server 2008 x64 Server Core, and each server  hosts a DC so that I have some degree of fault tolerance.

When I attempted to start the VMs using Hyper-V Manager from my Windows 7 desktop,  I encountered the following error shortly after seeing the "Connecting to Virtual  Machine Management service..." message:

> Cannot connect to the RPC service on computer 'ICEMAN'. Make sure your RPC service
> is running.

Well, the RPC service was definitely running, but since both DCs were powered  down, Hyper-V Manager had no way of authenticating me in order to access the remote  servers (note that I got the same error message when trying to access ROGUE).

I could login to both Hyper-V servers (with my domain account, thanks to cached  credentials), but as I've [noted before](/blog/jjameson/2008/08/28/some-gotchas-with-remote-administration-of-hyper-v), I struggled with managing Hyper-V from the command line. I had  seen some examples of how to do it using PowerShell, but never anything from a simple  command console. Note that I never did install PowerShell on my Server Core boxes.

Fortunately, last Sunday I found Ben Armstrong's post for [Starting a Hyper-V Virtual Machine](http://blogs.msdn.com/virtual_pc_guy/archive/2008/01/29/starting-a-hyper-v-virtual-machine.aspx) which shows not only the PowerShell version  that I had seen before, but also a WMI version (which can be used with plain old  VBScript). While Ben's code certainly works, I ended up snarfing some similar [sample code
from MSDN](http://msdn.microsoft.com/en-us/library/cc723874%28VS.85%29.aspx) to create a new ManageVM.vbs script (which seems more intuitive --  at least to me -- than using "StartVM.vbs" to stop a VM).

Now, whenever I am unable to manage my Hyper-V servers remotely, I can use something  like the following to start one of the VMs:

```
cscript \NotBackedUp\Public\Toolbox\Scripts\ManageVM.vbs XAVIER1 start
```

