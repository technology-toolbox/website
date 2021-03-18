---
title: Temporary ASP.NET Files Are Not Deleted
date: 2009-04-01T07:14:00-06:00
excerpt:
  "Yesterday afternoon, I discovered that there were 64,725 items (consuming
  2.41 GB) in the C:\\Windows\\Microsoft.NET\\Framework\\v2.0.50727\\Temporary
  ASP.NET Files\\root folder of my development VM for Microsoft Office
  SharePoint Server (MOSS) 2007. Apparently..."
aliases:
  [
    "/blog/jjameson/archive/2009/03/31/temporary-asp-net-files-are-not-deleted.aspx",
    "/blog/jjameson/archive/2009/04/01/temporary-asp-net-files-are-not-deleted.aspx",
  ]
draft: true
categories: ["SharePoint", "Development", "My System"]
tags: ["MOSS 2007", "Core Development", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/04/01/temporary-asp-net-files-are-not-deleted.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/04/01/temporary-asp-net-files-are-not-deleted.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Yesterday afternoon, I discovered that there were 64,725 items (consuming 2.41
GB) in the **C:\Windows\Microsoft.NET\Framework\v2.0.50727\Temporary ASP.NET
Files\root** folder of my development VM for Microsoft Office SharePoint Server
(MOSS) 2007. Apparently, each time I do an IISRESET or recycle an application
pool, a new set of temporary files is generated but the old set is not being
deleted.

I don't recall ever encountering this problem in the past, which makes me
suspect that it is related to the fact that I am now running Windows Server
2008. Also note that I run SharePoint in "least privileged mode" -- meaning the
various service accounts used by SharePoint are not members of the local
**Administrators** group.

Doing a random spot check of the files, I noticed that they are indeed owned by
the various SharePoint service accounts (e.g. the identity of the app pool
running the Web application itself, or the identity running the SSP app pool).
However, I also confirmed that the corresponding service account has been
granted **Full Control** on the containing folder -- and thus it *seems* like
ASP.NET should have no problem deleting these files.

Since this is only happening on my local development environment -- at least for
the moment -- I'm not going to spend any more time investigating the issue at
this point. In other words, I'll just "punt" it and create another script and
add it to my Toolbox so I can periodically purge these files with minimal
effort.

Here is my script (**C:\NotBackedUp\Public\Toolbox\Scripts\Delete Temporary
ASP.NET Files root folder.cmd**), in case it might help anyone else out there
with the same problem:

```
@echo off

setlocal

set ROOT_FOLDER=%SystemRoot%\Microsoft.NET\Framework\v2.0.50727\Temporary ASP.NET Files\root

if not exist "%ROOT_FOLDER%" goto RootFolderDoesNotExist

echo Stopping IIS...
iisreset /stop

echo Removing Temporary ASP.NET Files root folder...
rmdir /Q /S "%SystemRoot%\Microsoft.NET\Framework\v2.0.50727\Temporary ASP.NET Files\root"

echo Starting IIS...
iisreset /start

exit

:RootFolderDoesNotExist
echo Root folder does not exist: "%ROOT_FOLDER%"
pause
```

Now all I need to do is periodically right-click this script, click **Run as
administrator**, wait for the files to be purged, and then get back to whatever
I was doing before I noticed that I was running out of disk space ;-)

Note that I tried using `%FrameworkDir%` instead of
`%SystemRoot%\Microsoft.NET\Framework`, but that doesn't work unless you run the
script from an Administrator command prompt.
