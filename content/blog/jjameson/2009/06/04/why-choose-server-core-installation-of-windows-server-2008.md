---
title: "Why choose \"Server Core\" installation of Windows Server 2008?"
date: 2009-06-04T19:12:00-06:00
excerpt:
  "If you ever find yourself looking for reasons or evidence why you should
  choose the \"Server Core\" installation option for Windows Server 2008, try
  searching for the following: 
   \"Windows Server 2008 Server Core installation not affected\"
  site:microsoft..."
aliases:
  [
    "/blog/jjameson/archive/2009/06/04/why-choose-server-core-installation-of-windows-server-2008.aspx",
  ]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["Simplify", "Windows Server", "Infrastructure", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/06/04/why-choose-server-core-installation-of-windows-server-2008.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/06/04/why-choose-server-core-installation-of-windows-server-2008.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

If you ever find yourself looking for reasons or evidence why you should choose
the "Server Core" installation option for Windows Server 2008, try searching for
the following:

{{< blockquote "font-italic" >}}

["Windows Server 2008 Server Core installation not affected" site:microsoft.com/technet/security](http://www.bing.com/search?q=%22Windows+Server+2008+Server+Core+installation+not+affected%22+site%3Amicrosoft.com%2Ftechnet%2Fsecurity)

{{< /blockquote >}}

You will find page after page of results similar to the following:

> [Microsoft Security Bulletin MS08-054 - Critical](http://www.microsoft.com/technet/security/Bulletin/MS08-054.mspx)\
> **Windows Server 2008 server core installation not affected**. The
> vulnerability addressed by this update does not affect supported editions of
> Windows Server 2008 if Windows Server ...\
> <cite>www.microsoft.com/technet/security/Bulletin/MS08-054.mspx</cite> •
> [Cached page](http://cc.bingj.com/cache.aspx?q=%22windows+server+2008+server+core+installation+not+affected%22&d=76133794257994&mkt=en-US&setlang=en-US&w=e671a5b0,e59d79e9)
>
> [Microsoft Security Bulletin MS08-078 - Critical](http://www.microsoft.com/technet/security/bulletin/ms08-078.mspx)\
> **Windows Server 2008 server core installation not affected**. The
> vulnerabilities addressed by this update do not affect supported editions of
> Windows Server 2008 if Windows Server ...\
> <cite>www.microsoft.com/technet/security/bulletin/ms08-078.mspx</cite> •
> [Cached page](http://cc.bingj.com/cache.aspx?q=%22windows+server+2008+server+core+installation+not+affected%22&d=76162242072335&mkt=en-US&setlang=en-US&w=c3f59bce,63fef00c)
>
> [Microsoft Security Bulletin MS08-053 - Critical](http://www.microsoft.com/technet/security/Bulletin/MS08-053.mspx)\
> **Windows Server 2008 server core installation not affected**. The
> vulnerability addressed by this update does not affect supported editions of
> Windows Server 2008 if Windows Server ...\
> <cite>www.microsoft.com/technet/security/Bulletin/MS08-053.mspx</cite> •
> [Cached page](http://cc.bingj.com/cache.aspx?q=%22windows+server+2008+server+core+installation+not+affected%22&d=76116313320319&mkt=en-US&setlang=en-US&w=92aafff1,c365475a)
>
> [Microsoft Security Bulletin MS08-024 - Critical: Cumulative Security ...](http://www.microsoft.com/technet/security/Bulletin/MS08-024.mspx)\
> **Windows Server 2008 server core installation not affected**. The
> vulnerabilities addressed by these updates do not affect supported editions of
> Windows Server 2008 if Windows Server ...\
> <cite>www.microsoft.com/technet/security/Bulletin/MS08-024.mspx</cite> •
> [Cached page](http://cc.bingj.com/cache.aspx?q=%22windows+server+2008+server+core+installation+not+affected%22&d=76113650584856&mkt=en-US&setlang=en-US&w=f7f0adec,d0a922b0)
>
> [Microsoft Security Bulletin MS08-052 - Critical](http://www.microsoft.com/technet/security/bulletin/ms08-052.mspx)\
> **Windows Server 2008 Server Core installation not affected**. The
> vulnerabilities addressed by this update do not affect supported editions of
> Windows Server 2008 if Windows Server ...\
> <cite>www.microsoft.com/technet/security/bulletin/ms08-052.mspx</cite> •
> [Cached page](http://cc.bingj.com/cache.aspx?q=%22windows+server+2008+server+core+installation+not+affected%22&d=76123006445241&mkt=en-US&setlang=en-US&w=59991b53,79c72b54)
>
> ...

Seeing all these results is refreshing when I think back on the challenges I had
to overcome when building out my first Hyper-V server using the Server Core
installation, such as
[configuring remote administration of Hyper-V](/blog/jjameson/2008/08/28/some-gotchas-with-remote-administration-of-hyper-v),
or whenever I need to view PerfMon data on a Server Core machine (which is
trivial on a "Full" installation, but not quite so easy on a Server Core
installation).

Thanks to Ana Paula Moreira Franco, a Senior Consultant with Microsoft
Consulting Services in Brazil, for pointing out these search terms.
