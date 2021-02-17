---
title: "Issues with Running MOSS 2007 on Windows Server 2008"
date: 2009-02-10T10:41:00-07:00
excerpt: "In a previous post , I hinted at some issues that I recently encountered after switching from Windows Server 2003 to Windows Server 2008 on my primary development VM for Microsoft Office SharePoint Server (MOSS) 2007. 
 To make this a little more fun..."
aliases: ["/blog/jjameson/archive/2009/02/10/issues-with-running-moss-2007-on-windows-server-2008.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/02/10/issues-with-running-moss-2007-on-windows-server-2008.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/02/10/issues-with-running-moss-2007-on-windows-server-2008.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In a [previous post](/blog/jjameson/2009/01/23/error-installing-moss-2007-december-cumulative-update), I hinted at some issues that I recently encountered after         switching from Windows Server 2003 to Windows Server 2008 on my primary development         VM for Microsoft Office SharePoint Server (MOSS) 2007.

To make this a little more fun, let's start with a pop quiz for the SharePoint experts         out there.

What's wrong with this picture?

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_MOSS2007-WS2008-Operations(non-Admin).jpg"
alt="MOSS 2007 Central Administration - Operations on Windows Server 2008"
height="436"
width="600"
title="Figure 1: MOSS 2007 Central Administration - Operations on Windows Server 2008" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_MOSS2007-WS2008-Operations%28non-Admin%29.jpg)

A few of you might even be able to detect the problem without viewing the full-sized         image.

See it?

No? How about a hint...

Suppose you are configuring your farm for the first time and you need to start the         **Office SharePoint Server Search** service. Which link do you need         to click?

See it now?

No? Okay, how about if I make this really easy by zooming in on the area where you         should be focusing your attention.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_MOSS2007-WS2008-Operations(non-Admin)-closeup.jpg"
alt="MOSS 2007 Central Administration - Operations on Windows Server 2008 (close-up of Topology and Services section)"
height="225"
width="414"
title="Figure 2: MOSS 2007 Central Administration - Operations on Windows Server 2008 (close-up of Topology and Services section)" >}}

See anything missing?

Actually, there are two things missing:

- The **Services on server** link
- The **Incoming e-mail settings** link

Hmmm...that's odd, isn't it?

It turns out that you need to run Internet Explorer as an administrator whenever         you access Central Administration. In other words, you need to right-click Internet         Explorer in the Start menu and then click **Run as administrator**.         (Note that you cannot right-click the **Internet** shortcut -- since         it doesn't give you the option to **Run as administrator**; you need         to click Start, then click **All Programs**, and then right-click *that***Internet Explorer**).

You will then find all of the expected links under the **Topology and Services** section.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_MOSS2007-WS2008-Operations(Admin).jpg"
alt="MOSS 2007 Central Administration - Operations on Windows Server 2008 (IE running as administrator)"
height="463"
width="600"
title="Figure 3: MOSS 2007 Central Administration - Operations on Windows Server 2008 (IE running as administrator)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_MOSS2007-WS2008-Operations%28Admin%29.jpg)

This certainly isn't the only time you'll need to run as administrator when using         MOSS 2007 on Windows Server 2008. Actually, you'll find yourself doing this *a lot*.         For example, in order to run         {{< kbd "stsadm.exe" >}}, you'll need to first start a command prompt as administrator.

I suppose that, technically speaking, you might be able to disable User Account         Control altogether -- but a) I didn't bother to check if this works, and b) I certainly         don't recommend it.

Is running MOSS 2007 on Windows Server 2008 -- especially in a development environment         -- more difficult than on Windows Server 2003? Sure.

Is it that big of deal? No -- at least not in my opinion.

Incidentally, you might be wondering how I discovered this. I found that when I         browsed to Central Administration using Firefox, all of the links appeared as expected         -- which obviously made me think it was related to the browser (Internet Explorer).

This might very well be documented somewhere on TechNet, but at this point I am         not aware of it. If I discover it later, I'll update this post.

> **Update (2009-03-12)**
>
> Also note the following blurb from my [earlier post](/blog/jjameson/2009/01/15/sharepoint-configuration-wizard-hangs-with-ipv6-address) (duplicated here since it took me more than 20 seconds to find it when the issue came up during a team discussion)...
>
> Note that after aliasing my local VM name to the loopback address (127.0.0.1), I                 had to use the workaround in [KB 896861](http://support.microsoft.com/kb/896861)                 in order to resolve "access denied" errors when indexing content:
>
> > Access is denied. Check that the Default Content Access Account has access to this
> > content, or add a crawl rule to crawl this content. (The item was deleted because
> > it was either not found or the crawler was denied access to it.)

> **Update (2009-04-01)**
>
> Also see my post describing the issue where the [**Temporary ASP.NET Files folder** is not being cleaned up](/blog/jjameson/2009/04/01/temporary-asp-net-files-are-not-deleted) on my Windows Server 2008 development VM. I don't recall ever encountering this problem before when using Windows Server 2003.

