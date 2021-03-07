---
title: "Windows 7 SP1/SSD Rebuild (and MaxPatchCacheSize = 0)"
date: 2011-03-09T08:34:00-07:00
excerpt: "I spent a few hours last night (and another hour this morning) rebuilding my Windows 7 desktop that I use as my primary workstation (WOLVERINE). 
 In case you haven't heard, Service Pack 1 for Windows 7 and Windows Server 2008 R2 was released a few weeks..."
aliases: ["/blog/jjameson/archive/2011/03/08/windows-7-sp1-ssd-rebuild-and-maxpatchcachesize-0.aspx", "/blog/jjameson/archive/2011/03/09/windows-7-sp1-ssd-rebuild-and-maxpatchcachesize-0.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Windows 7"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/09/windows-7-sp1-ssd-rebuild-and-maxpatchcachesize-0.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/09/windows-7-sp1-ssd-rebuild-and-maxpatchcachesize-0.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

I spent a few hours last night (and another hour this morning) rebuilding my
Windows 7 desktop that I use as my primary workstation (WOLVERINE).

In case you haven't heard,
[Service Pack 1 for Windows 7 and Windows Server 2008 R2](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=c3202ce6-4056-4059-8a1b-3a9b77cdfdda)
was released a few weeks ago. Note that I certainly don't rebuild my computers
every time an OS service pack is released. Rather, the driving factor behind my
latest rebuild was a new solid-state drive (SSD) that I recently received.

I've been running an SSD in my Microsoft laptop for a while now and I must say
the difference is amazing. If you are a developer running a laptop without an
SSD then, believe me, I feel your pain. *Especially* if you are a SharePoint
developer running on a laptop without an SSD. Been there...done that...won't
ever do it again.

These days, I do 99% of my development work using a number of Hyper-V VMs
running in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab) and -- at least in my opinion -- this is the only way to go. Well,
okay, maybe not the *only* way, but it certainly eliminates any sensation of not
"firing on all cylinders" in terms of productivity.

I've mentioned in a previous post about
[using MaxPatchCacheSize to save signficant disk space](/blog/jjameson/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0).
When I first rebuilt my laptop with an 80 GB SSD, I didn't set MaxPatchCacheSize
to 0 and consequently ended up wasting several gigabytes of "expensive storage"
in the \Windows\Installer\$PatchCache$ folder (remember, this is SSD storage
that I'm talking about here). I've since rebuilt my laptop and ensured that I
applied this setting immediately after installing the operating system.

So when it came time last night to rebuild my desktop with a shiny new SSD, I
immediately set MaxPatchCacheSize to 0 after installing Windows 7. As you can
see from the following screenshot, this doesn't eliminate all of the space
consumed by the \Windows\Installer folder. On my desktop, the Installer folder
still consumes about 2 GB of space (which certainly seems like a waste, in my
opinion, but oh well). However, at least I can take some satisfaction in knowing
that it could be much worse (i.e. by not constraining the patch cache).

{{< figure src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-WOLVERINE-Baseline-600x507.png" alt="Disk usage on WOLVERINE (baseline with MaxPatchCacheSize = 0)" class="screenshot" height="507" width="600" title="Figure 1: Disk usage on WOLVERINE (baseline with MaxPatchCacheSize = 0)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-WOLVERINE-Baseline-927x784.png)

In case anyone is interested in how I recommend building out a workstation, here
are the details.

> **Update (2011-03-11)**
>
> I've updated the following to reflect a few omissions in my original post (such as installing the Remote Server Administration Tools for Windows 7 *before* installing SP1).

Start by installing Windows 7. When prompted to create a new user, specify a
user name that will be used whenever you need administrator privileges (for
example, to install software or make other configuration changes). Personally,
I've been using "foo" for as long as I can remember. I strongly prefer my
primary domain account (i.e. TECHTOOLBOX\jjameson) *not* be a member of the
Administrators group.

I then immediately installed the
[Remote Server Administration Tools for Windows 7](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=7d2f6ad7-656b-4313-a005-4e344e43997d&displaylang=en).

> **Warning**
>
> If you plan on using the Remote Server Administration Tools for Windows 7, then now is the time to install them. Otherwise [you won't be able to install RSAT after upgrading to Windows 7 SP1](/blog/jjameson/2011/03/11/before-you-install-windows-7-service-pack-1) -- at least not until the updated version is released.

Next, I installed Windows 7 SP1 and subsequently joined the machine to the
domain (TECHTOOLBOX). After rebooting, my various group policies kicked in, thus
ensuring, for example, the desktop is configured to use my local WSUS server for
Windows Update. Fortunately -- having just installed Windows 7 SP1 -- the number
of updates detected was actually quite small (although it still required a
couple of reboots to get the .NET Framework 4 Client Profile installed and
patched).

I then proceeded to install the following:

1. **Microsoft Security Essentials**
2. Alternate Web browers -- primarily used for testing and verifying sites that I develop:
   - **Firefox**
   - **Safari**
   - **Chrome**
   - **Opera**
3. **Virtual CloneDrive --** my preferred tool for dealing with .iso and .img files downloaded from MSDN
4. Essential productivity "stuff":
   - **Microsoft Office Professional Plus 2010**
   - **Microsoft SharePoint Designer 2010**
   - **Microsoft Visio Premium 2010**
   - **Microsoft Outlook Hotmail Connector**
   - **Microsoft Office Live Meeting 2007 client**
5. **Microsoft SQL Server 2008 R2**
6. **Microsoft Visual Studio 2010 Ultimate**
7. **Visual Studio 2010 Productivity Power Tools**
8. **Team Foundation Server Power Tools**
9. **Visual Studio 2010 Service Pack 1**
10. **Microsoft Expression Studio 4**
11. **System Center Operations Manager (SCOM) 2007 R2** -- just the Operations Console (so I can easily manage the various servers in the "Jameson Datacenter" from my Windows 7 desktop)
12. **Mozilla Thunderbird (**if you are wondering why I use this in addition to Microsoft Outlook, then apparently you haven't seen [one of my other posts](/blog/jjameson/2010/04/26/outlook-2010-does-not-work-with-windows-server-2003-pop3-service))
13. **FeedDemon**
14. Useful development and debugging "stuff":
    1. **Debugging Tools for Windows (x86)**
    2. **Debugging Tools for Windows (x64)**
    3. **Microsoft NetMon**
    4. **Fiddler**
    5. **MSBuild Community Tasks**
    6. **Paint.NET**
    7. **Rooler**
15. **Adobe Flash** -- I wish I didn't need this, but the Web "is what it is"...although there's probably some people out there saying the same thing about Silverlight :-)
16. **Adobe Reader** -- another thing I wish I didn't need to install (especially since the Adobe developers apparently can't figure out how -- or are simply too lazy -- to get it to work with roaming profiles, thereby forcing me to install the old 8.1.x version)
17. Personal "stuff":
    1. **Microsoft Money** (yes, the "sunset" edition -- but [I'm not switching to Quicken](/blog/jjameson/2010/03/28/you-ll-have-to-pry-that-money-from-my-cold-dead-hands)...not now...probably not ever)
    2. **ActivePython 2.7.1.3 (64-bit)** -- for running PocketSense (Python) scripts in order to download quotes and credit card statements into Money via OFX
    3. **Windows Live Essentials --** primarily for Windows Live Photo Gallery and Windows Live Movie Maker
    4. **Sony Picture Utility --** unfortunately, I can't import video directly onto my computer from my five-year old digital camcorder; intead I have to use this (admittedly poor) software instead

After installing all of this, I then went through the Windows Update cycle a few
more times in order to apply additional updates (for example, security updates
for Office 2010).

Wow, look at the time! It looks like I'll be working through lunch today to make
up for my late start ;-)

