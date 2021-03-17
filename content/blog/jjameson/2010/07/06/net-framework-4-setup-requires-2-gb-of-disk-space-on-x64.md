---
title: ".NET Framework 4 Setup Requires ~2 GB of Disk Space on x64"
date: 2010-07-06T04:26:00-06:00
excerpt: "Windows Update started generating errors last week on one of my servers. Specifically, the server (JUBILEE) was encountering an error when trying to install Microsoft .NET Framework 4: 
 
 Log Name: System
Source: Microsoft-Windows-WindowsUpdateClient..."
aliases: ["/blog/jjameson/archive/2010/07/05/net-framework-4-setup-requires-2-gb-of-disk-space-on-x64.aspx", "/blog/jjameson/archive/2010/07/06/net-framework-4-setup-requires-2-gb-of-disk-space-on-x64.aspx"]
draft: true
categories: ["Infrastructure"]
tags: ["WSUS", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/07/06/net-framework-4-setup-requires-2-gb-of-disk-space-on-x64.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/07/06/net-framework-4-setup-requires-2-gb-of-disk-space-on-x64.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Windows Update started generating errors last week on one of my servers.
Specifically, the server (JUBILEE) was encountering an error when trying to
install Microsoft .NET Framework 4:

{{< log-excerpt >}}

```
Log Name:      System
Source:        Microsoft-Windows-WindowsUpdateClient
Date:          6/30/2010 3:03:25 AM
Event ID:      20
Task Category: Windows Update Agent
Level:         Error
Keywords:      Failure,Installation
User:          SYSTEM
Computer:      jubilee.corp.technologytoolbox.com
Description:
Installation Failure: Windows failed to install the following update with error 0x80070643: Microsoft .NET Framework 4 for Windows Server 2008 x64-based Systems (KB982671).
```

{{< /log-excerpt >}}

Note that all of the other servers and workstations in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter)
installed .NET Framework 4 without incident, so I was a little baffled why this
particular server was failing.

Rather than troubleshooting the error using the steps described in
[KB 958052](http://support.microsoft.com/kb/958052) (as I've done in the past
when encountering error 0x80070643 with Windows Update), I decided it would be
quicker and easier to try installing it using the
[.NET Framework 4 standalone installer](http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=0a391abd-25c1-4fc0-919f-b21f31ab88b7).

Shortly after starting the setup, I observed the following:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Microsoft-.NET-Framework-4-Setup-Requirements-503x470.png"
alt="Microsoft .NET Framework 4 Setup - disk space requirements"
class="screenshot" height="470" width="503"
title="Figure 1: Microsoft .NET Framework 4 Setup - disk space requirements" >}}

Of course, had I actually read the **System Requirements** section of the
[.NET Framework Download page](http://www.microsoft.com/net/Download.aspx)
beforehand, I would have known that the minimum disk space required by the .NET
Framework 4 is much higher than I thought (for x64, the stated minimum is 2 GB
-- slightly higher than the 1863 MB threshold enforced by the setup program).

Once I freed up the necessary disk space on the server, I was able to click
**Refresh** and then click **Next** to proceed with the install (which completed
without issue).

It still seems a little shocking that nearly 2 GB of disk space can be
subsequently consumed after installing a 48.1 MB download, but I did, in fact,
end up with a mere 107 MB of free space after installing .NET Framework 4 on
this server (that is, until I expanded the corresponding VHD file for the VM).

Live and learn.
