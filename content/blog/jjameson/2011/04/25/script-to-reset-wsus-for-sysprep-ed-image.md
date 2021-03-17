---
title: "Script to Reset WSUS for SysPrep'ed Image"
date: 2011-04-25T04:03:00-06:00
excerpt:
  "Here's a useful script for those, like me, that use SysPrep'ed images to
  create new virtual machines and also leverage Windows Server Update Services
  (WSUS) to keep machines up-to-date with the latest patches. 
   Reset WSUS for SysPrep Image.cmd 
   
  ..."
aliases:
  [
    "/blog/jjameson/archive/2011/04/24/script-to-reset-wsus-for-sysprep-ed-image.aspx",
    "/blog/jjameson/archive/2011/04/25/script-to-reset-wsus-for-sysprep-ed-image.aspx",
  ]
draft: true
categories: ["Infrastructure", "My System"]
tags: ["WSUS", "Infrastructure", "Virtualization", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/04/25/script-to-reset-wsus-for-sysprep-ed-image.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/04/25/script-to-reset-wsus-for-sysprep-ed-image.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Here's a useful script for those, like me, that
[use SysPrep'ed images to create new virtual machines](/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines)
and also leverage Windows Server Update Services (WSUS) to keep machines
up-to-date with the latest patches.

### Reset WSUS for SysPrep Image.cmd

```
net stop wuauserv

reg.exe delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate /v PingID /f
reg.exe delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate /v AccountDomainSid /f
reg.exe delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate /v SusClientId /f
reg.exe delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate /v SusClientIDValidation /f

net start wuauserv

wuauclt.exe /resetauthorization /detectnow
```

I keep a copy of this script in my
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) and run it after
creating a new VM from a SysPrep'ed image.

If you are not sure why this is helpful, refer to the following KB article:

{{< reference
title="A Windows 2000-based, Windows Server 2003-based, or Windows XP-based computer that was set up by using a Windows 2000, Windows Server 2003, or Windows XP image does not appear in the WSUS console"
linkHref="http://support.microsoft.com/kb/903262/en-us" >}}
