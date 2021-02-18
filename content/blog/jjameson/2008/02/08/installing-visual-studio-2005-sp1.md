---
title: "Installing Visual Studio 2005 Service Pack 1"
date: 2008-02-08T01:12:00-07:00
excerpt: "Stepping into the \"Wayback Machine\" for a moment, I realized that I hinted about a problem with installing Visual Studio 2005 SP1 , but I never got around to blogging about it in more detail. Unfortunately, I just got bit by this problem yet again this..."
aliases: ["/blog/jjameson/archive/2008/02/08/installing-visual-studio-2005-sp1.aspx"]
draft: true
categories: ["Development"]
tags: ["Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/02/08/installing-visual-studio-2005-sp1.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/02/08/installing-visual-studio-2005-sp1.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Stepping into the ["Wayback
Machine"](http://en.wikipedia.org/wiki/Wayback_Machine) for a moment, I realized that I hinted about [a problem with installing Visual Studio 2005 SP1](/blog/jjameson/2007/06/23/save-huge-amounts-of-disk-space-by-slipstreaming-service-packs), but I never got around to  blogging about it in more detail. Unfortunately, I just got bit by this problem  yet again this morning because I naively tried to use the vanilla Windows Update  functionality after installing SQL Server 2005 on my freshly-rebuilt Windows Server  2003 machine. [Note: SQL Server 2005 uses the Visual Studio 2005 IDE as the shell  for SQL Server Management Studio. Consequently, even though you may not have installed  Visual Studio 2005, you need to install Visual Studio 2005 SP1 after installing  SQL Server 2005. This is separate from any SQL Server service packs.]

After double-clicking the "Golden Shield" (a.k.a. the Windows Update icon in  the task tray), I was quickly informed that the attempt to install Visual Studio  2005 Service Pack 1 failed...miserably.

Looking at the event log, I noticed the following errors:

{{< blockquote "font-italic" >}}

Event Type: Error

Event Source: MsiInstaller

Event Category: None

Event ID: 1008

Date: 2/8/2008

Time: 6:03:44 AM

User: JJAMESON1\Administrator

Computer: JJAMESON1

Description:

The installation of C:\WINDOWS\Installer\49e8e21.msp is not permitted due to
an error in software restriction policy processing. The object cannot be trusted.

Event Type: Error

Event Source: MsiInstaller

Event Category: None

Event ID: 11718

Date: 2/8/2008

Time: 6:03:44 AM

User: JJAMESON1\Administrator

Computer: JJAMESON1

Description:

Product: Microsoft Visual Studio 2005 Premier Partner Edition - ENU -- Error
1718.File C:\WINDOWS\Installer\49e8e21.msp did not pass the digital signature
check. For more information about a possible resolution for this problem, see
[http://go.microsoft.com/fwlink/?LinkId=73863](http://go.microsoft.com/fwlink/?LinkId=73863).

Event Type: Error

Event Source: MsiInstaller

Event Category: None

Event ID: 1023

Date: 2/8/2008

Time: 6:03:55 AM

User: JJAMESON1\Administrator

Computer: JJAMESON1

Description:

Product: Microsoft Visual Studio 2005 Premier Partner Edition - ENU - Update
'Microsoft Visual Studio 2005 Team Explorer - ENU Service Pack 1 (KB926601)'
could not be installed. Error code 1603. Additional information is available
in the log file C:\NOTBAC~1\Temp\Volatile\VS80sp1-KB926601-X86-ENU\VS80sp1-KB926601-X86-ENU-msi.0.log.

{{< /blockquote >}}

These errors quickly jogged my memory about the known issues with installing  Visual Studio 2005 Service Pack 1. Consequently, I proceeded to delete the 49e8e21.msp  file in my Installer folder (since I really don't like having 455MB "orphan" files  hanging out on my hard disk) and subsequently used my trusty script to install VS  2005 SP1 instead of relying on Windows Update to do this for me.

Here is the script I developed a long time ago for installing VS 2005 SP1 (well,  actually, I just stole the pieces from [Heath's
blog](http://blogs.msdn.com/heaths)):

```
:: This is basically a combination of two of Heath Stewart's blog postings
:: 
:: http://blogs.msdn.com/heaths/archive/2007/01/11/workaround-for-error-1718.aspx
:: http://blogs.msdn.com/heaths/archive/2006/11/28/save-time-and-space-for-vs-2005-sp1-by-disabling-the-patch-cache.aspx

reg export HKLM\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers "CodeIdentifiers.reg" /y
reg add HKLM\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers /v PolicyScope /t REG_DWORD /d 1 /f

reg export HKLM\Software\Policies\Microsoft\Windows\Installer "Installer.reg" /y
reg add HKLM\Software\Policies\Microsoft\Windows\Installer /v MaxPatchCacheSize /t REG_DWORD /d 0 /f

net stop msiserver

start /wait VS80sp1-KB926601-X86-ENU.exe /L*v+ "VS80sp1-KB926601-X86-ENU.log" /quiet

reg delete HKLM\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers /v PolicyScope /f
reg import "CodeIdentifiers.reg"

reg delete HKLM\Software\Policies\Microsoft\Windows\Installer /v MaxPatchCacheSize /f
reg import "Installer.reg"

net stop msiserver

del /q "CodeIdentifiers.reg" 2>nul

del /q "Installer.reg" 2>nul
```

Okay, I'll be honest, I didn't really run this script this time. Actually, I  just ran the commands up to the point of installing the actual service pack (i.e.  to tweak the software installation policy and MaxPatchCacheSize). I then double-clicked  the "Golden Shield" and, behold, the service pack was installed successfully. [Imagine  trumpets blaring at this point...it's funnier.]

After Windows Update completed, I ran the commands from the script to revert  the registry hacks.

The reason I chose not to use the script is because Windows Update had already  downloaded a copy of the service pack from my WSUS server in the basement, and I  really didn't feel like downloading a 455MB file (VS80sp1-KB926601-X86-ENU.exe)  again.

I really hope Windows Server 2008 has been updated to acknowledge the fact that  patches from Microsoft may very well be several hundred megabytes (so we don't have  to hack the registry to get them to install in the future).

