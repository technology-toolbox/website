---
title: "HVRemote and Remote Administration of Hyper-V from Windows 7"
date: 2009-10-08T06:21:00-06:00
excerpt: "Last Sunday, I rebuilt my desktop ( WOLVERINE ) with the RTM build of Windows 7 Ultimate (x64). Previously, I'd been running the RC1 bits and I figured it was about time I got around to \"upgrading\" to the RTM version. [I say \"upgrading\" because -- at..."
aliases: ["/blog/jjameson/archive/2009/10/07/hvremote-and-remote-administration-of-hyper-v-from-windows-7.aspx", "/blog/jjameson/archive/2009/10/08/hvremote-and-remote-administration-of-hyper-v-from-windows-7.aspx"]
draft: true
categories: ["Infrastructure", "My System"]
tags: ["Infrastructure", "Virtualization", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/08/hvremote-and-remote-administration-of-hyper-v-from-windows-7.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/08/hvremote-and-remote-administration-of-hyper-v-from-windows-7.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Last Sunday, I rebuilt my desktop ([WOLVERINE](/blog/jjameson/2009/09/14/the-jameson-datacenter))  with the RTM build of Windows 7 Ultimate (x64). Previously, I'd been running the  RC1 bits and I figured it was about time I got around to "upgrading" to the RTM  version. [I say "upgrading" because -- at least it in my experience -- it is typically  better in the long run to simply reformat the hard drive and start over when installing  a newer version of the operating system. Obviously I don't always stick to this  principle. For example, I've upgraded some of my "Production" VMs from Windows Server  2003 to Windows Server 2008. However, I've seen a few bugs in my time that only  occur in upgraded environments and therefore try to avoid these whenever possible.]

In order to get back to managing my Hyper-V servers from my Windows 7 desktop,  I installed the [Remote Server Administration Tools for Windows 7](http://www.microsoft.com/downloads/details.aspx?FamilyID=7d2f6ad7-656b-4313-a005-4e344e43997d&displaylang=en) and turned on the corresponding  Windows feature. However, I found that I still couldn't connect Hyper-V Manager  to either of my Hyper-V servers. It was then that I remembered John Howard's [blog series on Hyper-V Remote Management](http://blogs.technet.com/jhoward/archive/2008/03/28/part-1-hyper-v-remote-management-you-do-not-have-the-requested-permission-to-complete-this-task-contact-the-administrator-of-the-authorization-policy-for-the-computer-computername.aspx) that describe the various hoops you  have to jump through in order to get remote administration of Hyper-V working. However,  this time when I went searching for John's blog posts, I stumbled across the [Hyper-V Remote Management Configuration
Utility](http://code.msdn.microsoft.com/HVRemote) (HVRemote).

Apparently -- quite some time ago -- John scripted the various configuration  steps described in his blog series, thus making it much quicker and almost effortless.

Note that in my environment, both the client and server are in the same domain.  Therefore, following the "10-second guide" for HVRemote, I created a firewall rule  to allow Microsoft Management Console:

{{< console-block-start >}}

C:\NotBackedUp\Public\Toolbox\HVRemote&gt;{{< kbd "cscript hvremote.wsf /mmc:enable" >}}

{{< sample-block >}}

Microsoft (R) Windows Script Host Version 5.8\
Copyright (C) Microsoft Corporation. All rights reserved.\
\
\
Hyper-V Remote Management Configuration & Checkup Utility\
John Howard, Hyper-V Team, Microsoft Corporation.\
http://blogs.technet.com/jhoward\
Version 0.7 7th August 2009\
\
INFO: Computername is WOLVERINE\
INFO: Computer is in domain corp.technologytoolbox.com\
INFO: Current user is TECHTOOLBOX\jjameson-admin\
INFO: Assuming /mode:client as the Hyper-V role is not installed\
INFO: Build 7600.16385.amd64fre.win7\_rtm.090713-1255\
INFO: Detected Windows 7/Windows Server 2008 R2 OS\
INFO: Remote Server Administration Tools are installed\
INFO: Hyper-V Tools Windows feature is enabled\
INFO: No TCP rule was found\
INFO: No UDP rule was found\
INFO: MMC Firewall exception changes OK.\
INFO: Are running the latest version

{{< /sample-block >}}

{{< console-block-end >}}

However, when I attempted to connect to Hyper-V Manager to my Hyper-V servers,  I encountered errors similar to the following:

{{< blockquote "font-italic text-danger" >}}

Access denied. Unable to establish communication between 'ROGUE' and 'WOLVERINE'.

{{< /blockquote >}}

In addition to enabling the firewall rule, I found that I also needed to allow  Anonymous Logon remote DCOM access:

{{< console-block-start >}}

C:\NotBackedUp\Public\Toolbox\HVRemote&gt;{{< kbd "cscript hvremote.wsf /mode:client /anondcom:grant" >}}

{{< sample-block >}}

Microsoft (R) Windows Script Host Version 5.8\
Copyright (C) Microsoft Corporation. All rights reserved.\
\
\
Hyper-V Remote Management Configuration & Checkup Utility\
John Howard, Hyper-V Team, Microsoft Corporation.\
http://blogs.technet.com/jhoward\
Version 0.7 7th August 2009\
\
INFO: Computername is WOLVERINE\
INFO: Computer is in domain corp.technologytoolbox.com\
INFO: Current user is TECHTOOLBOX\jjameson-admin\
INFO: Build 7600.16385.amd64fre.win7\_rtm.090713-1255\
INFO: Detected Windows 7/Windows Server 2008 R2 OS\
INFO: Remote Server Administration Tools are installed\
INFO: Hyper-V Tools Windows feature is enabled\
\
INFO: Obtaining current Machine Access Restriction...\
INFO: Examining security descriptor\
INFO Granted Remote DCOM Access to Anonymous Logon\
WARN: See documentation for security implications\
INFO: Are running the latest version

{{< /sample-block >}}

{{< console-block-end >}}

After both of these changes were made, I found that I could once again perform  remote administration using Hyper-V Manager.

Kudos to John for simplifying the necessary configuration steps!

