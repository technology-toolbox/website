---
title: Script to Restart SharePoint 2010 Services
date: 2011-03-01T04:43:00-07:00
excerpt:
  A couple of years ago, I shared a script ( Restart SharePoint Services.cmd )
  for restarting the various services in Microsoft Office SharePoint Server
  (MOSS) 2007. 
   I've since created a new version of the script for use with SharePoint Server
  2010....
aliases:
  [
    "/blog/jjameson/archive/2011/02/28/script-to-restart-sharepoint-2010-services.aspx",
    "/blog/jjameson/archive/2011/03/01/script-to-restart-sharepoint-2010-services.aspx",
  ]
draft: true
categories: ["SharePoint", "My System"]
tags: ["SharePoint 2010", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/01/script-to-restart-sharepoint-2010-services.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/01/script-to-restart-sharepoint-2010-services.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

A couple of years ago, I shared a script
([Restart SharePoint Services.cmd](/blog/jjameson/2009/03/26/script-to-restart-sharepoint-services))
for restarting the various services in Microsoft Office SharePoint Server (MOSS)
2007.

I've since created a new version of the script for use with SharePoint Server
2010. As I mentioned in my previous post, this is primarily intended to be used
in development environments whenever you want to simulate a reboot of the server
(because this is much faster than actually rebooting). While I certainly hope
the memory leaks that often plagued MOSS 2007 solutions are a thing of the past,
I still find occasions where I want to quickly stop the various SharePoint
services and then restart them.

To do this, I made a copy of the previous script in my SharePoint
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) folder, and
updated the various service names for SharePoint 2010
(\NotBackedUp\Public\Toolbox\SharePoint\Scripts\Restart SharePoint 2010
Services.cmd):

```
@echo off

@echo Stopping SharePoint 2010 services...

iisreset /stop /noforce

net stop "SharePoint 2010 User Code Host"

net stop "SharePoint 2010 Timer"

net stop "SharePoint 2010 Administration"

net stop "SharePoint Server Search 14"

net stop "SharePoint Foundation Search V4"

net stop "SharePoint 2010 Tracing"

@pause

@echo Starting SharePoint 2010 services...

net start "SharePoint 2010 Tracing"

net start "SharePoint Foundation Search V4"

net start "SharePoint Server Search 14"

net start "SharePoint 2010 Administration"

net start "SharePoint 2010 Timer"

net start "SharePoint 2010 User Code Host"

iisreset /start

@pause
```

Note that I still use a `pause` statement so that I can optionally perform other
tasks while SharePoint is stopped (e.g. detach a content database, move it to a
different disk, and then reattach it).

Also note that, depending on your SharePoint configuration, you may see messages
about not being able to stop or start various services. For example, I only have
SharePoint Server Search currently running in my environment (not SharePoint
Foundation Search). Consequently, when I run the script above, I see the
following messages when stopping the services:

{{< console-block-start >}}

{{< sample-block >}}

The SharePoint Foundation Search V4 service is not started.\ \ More help is
available by typing NET HELPMSG 3521.

{{< /sample-block >}}

{{< console-block-end >}}

...and the corresponding message when starting the services:

{{< console-block-start >}}

{{< sample-block >}}

System error 1058 has occurred.\ \ The service cannot be started, either because
it is disabled or because it has no enabled devices associated with it.

{{< /sample-block >}}

{{< console-block-end >}}

I suppose the "hip" thing to do would be to upgrade this simple batch file to a
PowerShell script and then implement the "smarts" to determine whether the
services are actually configured or not (and consequently ignore ones that
aren't). Honestly, I just can't justify the effort in doing that given the
nature of this script.
