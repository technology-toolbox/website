---
title: Script to Restart SharePoint Services
date: 2009-03-26T08:33:00-06:00
excerpt:
  Since my previous post introduced one of my SharePoint Toolbox scripts, I
  thought I should share another one that is more applicable to a broader
  audience. As I've noted in the past , memory leaks are certainly not uncommon
  in the world of SharePoint...
aliases:
  [
    "/blog/jjameson/archive/2009/03/25/script-to-restart-sharepoint-services.aspx",
    "/blog/jjameson/archive/2009/03/26/script-to-restart-sharepoint-services.aspx",
  ]
draft: true
categories: ["SharePoint", "My System"]
tags: ["MOSS 2007", "WSS v3", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/03/26/script-to-restart-sharepoint-services.aspx"
---

Since my
[previous post](/blog/jjameson/2009/03/26/sharepoint-uls-logs-flooded-with-preserving-template-record-with-size)
introduced one of my SharePoint Toolbox scripts, I thought I should share
another one that is more applicable to a broader audience.

As I've
[noted in the past](/blog/jjameson/2008/04/09/memory-leak-in-splimitedwebpartmanager-a-k-a-idisposables-containing-idisposables),
memory leaks are certainly not uncommon in the world of SharePoint.
Consequently, you may want to periodically simulate a reboot of your development
environment in order to free up memory and get back to being productive. Trust
me, *simulating* a reboot is much, much faster than actually rebooting.

To do this, I created the following script and dropped it in my SharePoint
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) folder
(\NotBackedUp\Public\Toolbox\SharePoint\Scripts\Restart SharePoint
Services.cmd):

```Text
@echo off

@echo Stopping services...

iisreset /stop /noforce

net stop "Windows SharePoint Services Timer"

net stop "Windows SharePoint Services Administration"

net stop "Office SharePoint Server Search"

net stop "Windows SharePoint Services Search"

net stop "Windows SharePoint Services Tracing"

@pause

@echo Starting services...

net start "Windows SharePoint Services Tracing"

net start "Windows SharePoint Services Search"

net start "Office SharePoint Server Search"

net start "Windows SharePoint Services Administration"

net start "Windows SharePoint Services Timer"

iisreset /start

@pause
```

Note that I use a `pause` statement so that I can optionally perform other tasks
while SharePoint is stopped (e.g. detach a content database, move it to a
different disk, and then reattach it).
