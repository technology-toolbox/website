---
title: Fiddler + WPAD - VPN = SlowPerformance
date: 2008-06-27T11:57:00-06:00
excerpt:
  I needed to look at some low-level HTTP traffic this morning, so I fired up
  Fiddler -- my tool of choice for this kind of thing. Unfortunately, I found
  that as soon as I enabled Fiddler, my browsing experience slowed to a crawl.
  Page requests that previously...
aliases:
  [
    "/blog/jjameson/archive/2008/06/26/fiddler-wpad-slowperformance.aspx",
    "/blog/jjameson/archive/2008/06/27/fiddler-wpad-slowperformance.aspx",
  ]
draft: true
categories: ["Infrastructure", "Development"]
tags: ["Windows Vista", "Debugging"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/06/27/fiddler-wpad-slowperformance.aspx"
---

I needed to look at some low-level HTTP traffic this morning, so I fired up
[Fiddler](http://www.fiddlertool.com/) -- my tool of choice for this kind of
thing. Unfortunately, I found that as soon as I enabled Fiddler, my browsing
experience slowed to a crawl. Page requests that previously completed in 1-2
seconds were subsequently taking 30-40 seconds. Ugh!

Since I typically work inside a virtual machine running Windows Server 2003, I
thought that maybe I was hitting a problem specific to Fiddler on Vista. I then
did a quick Windows Live Search using the terms **Fidder Vista slow** and
quickly found a couple of people -- apparently including
[Eric Lawrence](http://groups.msn.com/HTTPFiddler/bugs.msnw?action=get_message&mview=0&ID_Message=815&LastModified=4675632312984197215),
himself -- referring to disabling IPv6 within Fiddler when using Vista. I
assumed that this must be the problem that I was encountering, but no matter
what I tried (e.g. restarting Fiddler after disabling the option, repeatedly
enabling and disabling IPv6, etc.) I couldn't get the performance to improve.

I then fired up
[Network Monitor 3.1](http://www.microsoft.com/downloads/details.aspx?familyid=18b1d59d-f4d8-4213-8d17-2f6dde7d7aac&displaylang=en)
and performed a quick trace while I requested the page through Fiddler again.
[By the way, I absolutely love the new filtering capabilities in NetMon 3.1
versus the old version. The interface is much more intuitive than the old 2.x
version and it couldn't be any easier than right-clicking a frame in the capture
and selecting **Add Cell to Display Filter**. NetMon Team, you guys rock! I
haven't quite got around to downloading the 3.2 beta, but hopefully someday
soon.]

Also note that I had to right-click **Microsoft Network Monitor 3.1** and click
**Run as administrator** in order to avoid the dreaded "Unknown Error" that
occurs when no networks are detected to capture.

So, a couple of minutes later, sure enough, there it was right in my capture:

{{< div-block "fst-italic" >}}

> DNS: QueryId = 0x18FC, QUERY (Standard query), Query for
> wpad.northamerica.corp.microsoft.com of type Host Addr on class Internet

{{< /div-block >}}

As soon as I saw the old "Web Proxy Auto Detect" I immediately became suspicious
that this was the culprit. I then closed Fiddler and modified my Internet
Explorer options to clear the **Automatically detect settings** checkbox.

Shazam! Problem solved.

Of course, had I actually been connected to CorpNet (which would have enabled
DNS requests for "wpad.northamerica.corp.microsoft.com" to resolve), I suppose I
wouldn't have encountered the problem in the first place.

Perhaps it is the fact that I am frequently connecting to different networks
(e.g. wireless, VPN, etc.) -- whereas many others don't -- but I'd really like
to think most people don't encounter little gotchas like this. The mere thought
of having to guide my mother through capturing a NetMon trace is, quite frankly,
horrifying ;-)
