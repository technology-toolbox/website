---
title: "Before you install Windows 7 Service Pack 1..."
date: 2011-03-10T22:00:00-07:00
excerpt: "...make darn sure you have already installed the Remote Server Administration Tools for Windows 7 (if you want to use them, of course). 
 Otherwise, like me, you'll likely resort to nuking your desktop and starting over. 
 Earlier this week, I posted..."
draft: true
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure", "Windows 7"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/11/before-you-install-windows-7-service-pack-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/11/before-you-install-windows-7-service-pack-1.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

...make darn sure you have already installed the [Remote Server Administration Tools for Windows 7](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=7d2f6ad7-656b-4313-a005-4e344e43997d&displaylang=en) (if you want to use them, of course).

Otherwise, like me, you'll likely resort to nuking your desktop and starting over.

Earlier this week, I posted about [how I rebuilt my Windows 7 desktop](/blog/jjameson/2011/03/09/windows-7-sp1-ssd-rebuild-and-maxpatchcachesize-0) while installing a new solid-state drive (SSD). Unfortunately, I later discovered that if you install Windows 7 SP1 before installing the current RSAT package, then you're essentially screwed.

If you attempt to install the current RSAT release after installing Windows 7 SP1, then you'll be greeted with a not-so-friendly message that the update not applicable to your computer.

Of course, this is clearly documented on the download page:

{{< reference title="Remote Server Administration Tools for Windows 7" linkHref="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=7d2f6ad7-656b-4313-a005-4e344e43997d&displaylang=en" >}}

> ...
>
> **\*\*Remote Server Administration Tools for Windows 7 can be installed ONLY on computers that are running the Enterprise, Professional, or Ultimate editions of Windows 7. This software CANNOT BE INSTALLED on computers that are running Windows 7 with Service Pack 1 (SP1). To run Remote Server Administration Tools for Windows 7 on a computer on which you want to run Windows 7 with SP1, first install Remote Server Administration Tools, and then upgrade to Service Pack 1.\*\***
>
> ...
>
> This software is not supported on computers that are running Windows 7 with Service Pack 1 (SP1). If your computer is running Windows 7 with SP1, and you try to install Remote Server Administration Tools for Windows 7, the following error message is displayed: "The update is not applicable to your computer." This limitation is by design, and is documented in the [Windows 7 with SP1 Deployment Guide](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=61924cea-83fe-46e9-96d8-027ae59ddc11).

Then again, if you downloaded RSAT for Windows 7 a long time ago (like I did back in 2009), then you probably missed this announcement. Apparently I *really* should have read the Windows 7 SP1 Deployment Guide before rebuilding my desktop. Ah...so much to read, so little time.

While researching the issue, I noticed that somebody mentioned you can uninstall SP1, install RSAT for Windows 7, and then re-install SP1. However, that didn't seem like the best idea in my mind. After all, since I rarely rebuild my desktop I figured I might as well "do it right" instead of resorting to some hack.

The updated RSAT package for Windows 7 SP1 should be available soon ("scheduled for release in Spring 2011").

Mercy...why couldn't this have sim-shipped with Windows 7 SP1?

