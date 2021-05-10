---
title: No Error Displayed When Database Update Fails
date: 2007-05-06T08:38:00-06:00
description:
  Here’s a nasty bug that I ran into about three weeks ago... If you attempt to
  modify a view on a list in Microsoft Office SharePoint Server (MOSS) 2007, but
  SharePoint is unable to save your changes to the database, no error is
  displayed in the UI...
aliases:
  [
    "/blog/jjameson/archive/2007/05/05/no-error-displayed-when-database-update-fails.aspx",
    "/blog/jjameson/archive/2007/05/06/no-error-displayed-when-database-update-fails.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/05/06/no-error-displayed-when-database-update-fails.aspx"
---

Here's a nasty bug that I ran into about three weeks ago...

If you attempt to modify a view on a list in Microsoft Office SharePoint Server
(MOSS) 2007, but SharePoint is unable to save your changes to the database, no
error is displayed in the UI. It just silently ignores the error and returns you
to the list (showing the unmodified view).

I must have tried to modify the view three times before I finally tried
something else and got a "Cannot complete this action" message. Only then did I
start combing through the SharePoint log files and discover that the transaction
log on SQL was full.
