---
title: "Server Core Installation - \"Accessing Windows in Notification period\""
date: 2008-11-05T09:41:00-07:00
excerpt:
  I had a rather rough start this morning. When I attempted to boot up my
  primary workstation and login, I kept encountering a problem loading my
  roaming profile. I could login to Vista, but my desktop was blank and I kept
  getting prompted to enter my...
aliases:
  [
    "/blog/jjameson/archive/2008/11/04/server-core-installation-accessing-windows-in-notification-period.aspx",
    "/blog/jjameson/archive/2008/11/05/server-core-installation-accessing-windows-in-notification-period.aspx",
  ]
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/11/05/server-core-installation-accessing-windows-in-notification-period.aspx"
---

I had a rather rough start this morning.

When I attempted to boot up my primary workstation and login, I kept
encountering a problem loading my roaming profile. I could login to Vista, but
my desktop was blank and I kept getting prompted to enter my credentials to
access the server (BEAST) where my user profile is stored. My first thought was
that BEAST was either locked up or disconnected from the network (although I was
really puzzled because the only times I can remember this server failing is when
we experience a power outage). Plus, if the server was actually down I should
have seen a message about not being able to load my profile -- not a dialog
prompting for my credentials.

Using my KVM switch, I toggled over to the BEAST console and logged in. I
noticed the login took considerably longer than usual, but using a couple of
ping operations from the command-line, I verified the network was up and
running.

I then tried to switch over to my other server running Hyper-V (ROGUE), which
hosts my domain controller -- or actually, I should say domain controllers,
since I recently built out a second DC when I virtualized my original domain
controller (XAVIER). In other words, ROGUE is now running XAVIER1 and XAVIER2 --
two Windows Server 2008 VMs with the **Active Directory Domain Services** role
enabled.

Unfortunately, I got "no love" from ROGUE when I tried to login. The screen was
blank and no amount of {{< kbd "CTRL+ALT+DELETE" >}} combinations could
resuscitate it. I then checked the physical server to ensure the it hadn't
somehow been turned off. Unfortunately, the "lights were on, but nobody was
home", so I forced a hard reboot. (I really hate doing that -- especially on a
Hyper-V server running 5 VMs. Oh well, desperate times call for...well, you know
the rest.

After ROGUE rebooted, I noticed that I still had problems accessing it. When I
was finally able to examine the event log, I discovered the following:

{{< log-excerpt >}}

Log Name: Application\
Source: Microsoft-Windows-Winlogon\
Date: 11/5/2008 6:04:29 AM\
Event ID: 4104\
Task Category: None\
Level: Information\
Keywords: Classic\
User: N/A\
Computer: ROGUE.corp.technologytoolbox.com\
Description:\
Accessing Windows in Notification period.

{{< /log-excerpt >}}

Ugh...from this I determined that it must have been exactly 61 days this morning
since I rebuilt ROGUE with Windows Server 2008 (Server Core) and virtualized a
couple of other physical servers. Apparently, I neglected to perform a vital
configuration step -- specifically, setting the product key.

For a Server Core installation, you need to use the Windows Software Licensing
Management Tool (slmgr.vbs) to enter the product key:

```Console
slmgr.vbs -ipk <Product Key>
```

After changing the product key, you then need to activate Windows:

```Console
slmgr.vbs -ato
```

In my opinion, the fact that the Windows Server 2008 setup does not prompt for a
product key is problematic. At least this is true for the MSDN version (which I
use to run the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) --
a.k.a. my home lab). I understand that many organizations use volume licensing
(and
[volume activation](http://technet.microsoft.com/en-us/library/cc303274.aspx)),
so I certainly can see why entering the product key at installation time should
be optional. I would just prefer that it wasn't skipped altogether. Also, I know
that I'm not the only one who has found it a little confusing to enter MSDN
product keys for Windows Server 2008 after installation.

Anyway, enough ranting -- I need to get back to my "day job" ;-)
