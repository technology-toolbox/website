---
title: "Copy/Paste Gotchas with Server Core"
date: 2008-07-07T07:11:00-07:00
excerpt: "I'm building out a new virtualized environment using Windows Server 2008 and Hyper-V. In order to maximize performance and follow recommended best practices, I am using Server Core as the host OS. 
 I have to admit, doing this much administration from..."
draft: true
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/07/07/copy-paste-gotchas-with-server-core.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/07/07/copy-paste-gotchas-with-server-core.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

I'm building out a new virtualized environment using Windows Server 2008 and  Hyper-V. In order to maximize performance and follow recommended best practices,  I am using Server Core as the host OS.

I have to admit, doing this much administration from the command line really  brings back memories from my "old Unix days" (before I switched to the Microsoft  platform). I'll also admit that I've had to do quite a bit of research in order  to figure out which commands need to be run in order to get a "functional" server.  Now please don't misunderstand me, I don't mind that Server Core doesn't really  allow you to do much with its out-of-the-box configuration (after all, that's the  whole point). It simply takes a little getting used to -- and, as is usually the  case, the first time takes longer than you expected.

I'll summarize some great resources I've found for working with Server Core and  Hyper-V in a separate post sometime soon, but for now I wanted to share this little  gotcha to save you the few minutes of frustration I spent trying to figure out what  I was doing wrong.

Since I needed to reconfigure the disks on the server, I opened the Disk Management  MMC console on my Windows Vista laptop and connected to the server -- or, rather  I should say that I *tried* to connect to the server. Instead of connecting,  I was greeted with the following error:

> Disk Management could not start Virtual Disk Service (VDS) on DMX-CORE1-MAINT.
> This can happen if the remote computer does not support VDS, or if a connection
> cannot be established because it was blocked by Windows Firewall.

A quick Windows Live Search for ["Disk Management could not start Virtual Disk Service"](http://search.live.com/results.aspx?q=%22Disk+Management+could+not+start+Virtual+Disk+Service%22&form=QBRE) led me straight to [LaNae Wade's post](http://blogs.technet.com/askds/archive/2008/06/05/how-to-enable-remote-administration-of-server-core-via-mmc-using-netsh.aspx) on enabling remote administration of Server Core. I then copied  the command line for enabling the firewall rules for the Disk Management MMC snap-in  and pasted it into my RDP session to the server:

```
netsh advfirewall firewall set rule group="Remote Volume Management" new 
enable=yes
```

Unfortunately, the response wasn't exactly what I expected:

> Group cannot be specified along with other identification conditions.

It turns out that the quotes around <samp>Remote Volume Management</samp> were  "corrupted", meaning they were converted to the angled quotes that tend to break  things in very bizarre ways. The really frustrating part is that the command prompt  makes the angled quotes appear just like regular quotes.

Once I edited the command line to replace the quotation marks, the command completed  with the expected results:

C:\&gt;<kbd>netsh advfirewall firewall set rule group="Remote Volume Management" new enable=yes</kbd>

```
Updated 3 rule(s).
Ok.
```

