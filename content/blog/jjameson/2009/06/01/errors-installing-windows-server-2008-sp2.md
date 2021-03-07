---
title: "Errors Installing Windows Server 2008 SP2"
date: 2009-06-01T09:14:00-06:00
excerpt: "Last week I approved Windows Server 2008 Service Pack 2 (SP2) and Windows Vista SP2 on my local WSUS (Windows Server Update Services) server. My expectation was that the various physical and virtual machines in the \"Jameson Datacenter\" would subsequently..."
aliases: ["/blog/jjameson/archive/2009/05/31/errors-installing-windows-server-2008-sp2.aspx", "/blog/jjameson/archive/2009/06/01/errors-installing-windows-server-2008-sp2.aspx"]
draft: true
categories: ["Infrastructure"]
tags: ["Windows Vista", "WSUS", "Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/06/01/errors-installing-windows-server-2008-sp2.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/06/01/errors-installing-windows-server-2008-sp2.aspx)
>
> Since 			[I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that  			blog ever goes away.

Last week I approved Windows Server 2008 Service Pack 2 (SP2) and Windows  	Vista SP2 on my local WSUS (Windows Server Update Services) server. My expectation  	was that the various physical and virtual machines in the 	["Jameson
Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) would subsequently install the update around 3:00 AM the following  	morning.

Unfortunately when I examined the WSUS console this morning, I found a number  	of computers reporting errors. After selecting one of the failed computers,  	I discovered the following:

{{< blockquote "font-italic" >}}

Windows Server 2008 Service Pack 2 Standalone x64-based Systems (KB948465)  		- English, French, German, Japanese, Spanish

Event reported at 6/1/2009 3:02 AM:

Installation Failure: Windows failed to install the following update  		with error 0x80070643: Windows Server 2008 Service Pack 2 Standalone x64-based  		Systems (KB948465) - English, French, German, Japanese, Spanish.

{{< /blockquote >}}

A quick search for 0x80070643 led to the following KB article:

{{< reference title="You receive error code 0x80070643 or error code 0x643 when you use the Windows Update or Microsoft Update Web sites to install updates" linkHref="http://support.microsoft.com/kb/958052" >}}

Unfortunately, this error code indicates "generic errors that basically state  	that an error was encountered by Windows Installer." Rather than immediately  	following the steps in the KB article to enable logging and try to reproduce  	the problem, I decided to take a quick look at the event logs and discovered  	the following:

{{< blockquote "font-italic" >}}

Log Name: System
Source: Microsoft-Windows-Service Pack Installer
Date: 6/1/2009 3:02:00 AM
Event ID: 8
Task Category: None
Level: Error
Keywords:
User: SYSTEM
Computer: dazzler.corp.technologytoolbox.com
Description:
Service Pack installation failed with error code 0x800f0826.

{{< /blockquote >}}

Another quick search for 0x800f0826 suggested that the problem might be due  	to insufficient disk space. However, I checked the free space on DAZZLER and  	observed that it had 4.09 GB free. Surely, 4 GB of disk space is sufficient  	to install Windows Server 2008 SP2!

Then I came across the following:

{{< reference title="Windows Server 2008 SP2 Deployment Guide" linkHref="http://technet.microsoft.com/en-us/library/dd351467(WS.10).aspx" >}}

Here are the disk space requirements according to the deployment guide:

{{< table class="small" caption="Disk Space Requirements for Windows Server 2008 SP2" >}}

| Installation method | Approximate disk space requirements |
| --- | --- |
| Stand-alone installation | <ul>				<li>x86-based: 1.8 GB to 2.9 GB</li><br>				<li>x64-based: 3.2 GB to 4.9 GB</li><br>				<li>ia64-based: 2.9 GB to 3.2 GB </li><br>			</ul> |
| Windows Update | <ul>				<li>x86-based: 350 MB</li><br>				<li>x64-based: 600 MB</li><br>				<li>ia64-based: 2.25 GB </li><br>			</ul> |
| Integrated installation | <ul>				<li>x86-based: 9 GB</li><br>				<li>x64-based: 12 GB</li><br>				<li>ia64-based: 13 GB</li><br>			</ul> |

{{< /table >}}

Crikey! According to this table, DAZZLER needs up to 4.9 GB of free space  	in order to install the Service Pack (since it is an x64 VM). Wow!

This, quite honestly, seems absolutely absurd!

However, given that it is relatively easy to expand a VHD, I decided to just  	go ahead and add another 2 GB to the system drive on the VM and see if that  	eliminated the issue. After all, as long as SP2 doesn't actually use 4.9 GB  	of space, then the actual physical space consumed by the VHD should be significantly  	less than 5 GB.

Using **Hyper-V Manager**, I expanded the VHD from 20 GB to  	22 GB and then started the VM up again. I then logged into the VM and used the 	**Disk Management** console to extend the volume to include the  	additional 2 GB of available storage.

Next, I kicked off the installation of Windows Server 2008 SP2 again. This  	time the installation completed without error. Woohoo!

Note that DAZZLER is a dedicated Team Foundation Server "build server" --  	so I didn't expect that it would need lots of disk space. In fact, it didn't  	-- at least not until Windows Server 2008 SP2 came along.

Now all I have to do is add some more disk space to the other servers that  	are failing to install SP2.

