---
title: "No Error Displayed When Database Update Fails"
date: 2007-05-06T02:38:00+08:00
excerpt: "Here’s a nasty bug that I ran into about three weeks ago... 
 If you attempt to modify a view on a list in Microsoft Office SharePoint Server (MOSS) 2007, but SharePoint is unable to save your changes to the database, no error is displayed in the UI..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
> 
>       This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2007/05/06/no-error-displayed-when-database-update-fails.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/05/06/no-error-displayed-when-database-update-fails.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

Here's a nasty bug that I ran into about three weeks ago...

If you attempt to modify a view on a list in Microsoft Office SharePoint
Server (MOSS) 2007, but SharePoint is unable to save your changes to the database,
no error is displayed in the UI. It just silently ignores the error and returns
you to the list (showing the unmodified view).

I must have tried to modify the view three times before I finally tried something
else and got a "Cannot complete this action" message. Only then did I start
combing through the SharePoint log files and discover that the transaction log
on SQL was full.

