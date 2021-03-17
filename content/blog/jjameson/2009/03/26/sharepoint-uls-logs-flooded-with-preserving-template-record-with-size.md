---
title: "SharePoint ULS Logs Flooded with \"Preserving template record with size...\""
date: 2009-03-26T08:07:00-06:00
excerpt:
  "I was digging through my blog dashboard this morning and I came across this
  post that I started back in January but apparently never got it past \"draft
  mode.\" I figured it was time to finish it off. 
   If you've been using Windows SharePoint Services..."
aliases: ["/blog/jjameson/archive/2009/03/25/sharepoint-uls-logs-flooded-with-preserving-template-record-with-size.aspx", "/blog/jjameson/archive/2009/03/26/sharepoint-uls-logs-flooded-with-preserving-template-record-with-size.aspx"]
draft: true
categories: ["SharePoint", "My System"]
tags: ["MOSS 2007", "WSS v3", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/26/sharepoint-uls-logs-flooded-with-preserving-template-record-with-size.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/26/sharepoint-uls-logs-flooded-with-preserving-template-record-with-size.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

I was digging through my blog dashboard this morning and I came across this post
that I started back in January but apparently never got it past "draft mode." I
figured it was time to finish it off.

If you've been using Windows SharePoint Services (WSS) v3 or Microsoft Office
SharePoint Server (MOSS) 2007 since the original release, you may have
encountered the problem where, even with the out-of-the-box settings, the
SharePoint Unified Logging Service (ULS) starts emitting thousands upon
thousands of messages per minute, eventually filling up your hard drive with
tens of gigabytes of log files, and ultimately "crashing" your server.
Fortunately, the "completely fill my hard drive" feature has long since been
removed and the ULS now stops logging before consuming every last megabyte of
available space.

However, there are still some scenarios where the ULS logs can grow very quickly
and while ULS eventually exerts some sense of self-restraint, it does manage to
dump gigabytes of relatively useless information on your hard drive in a
relatively short period of time.

Such is the case with my current MOSS 2007 development VM. Most every morning, I
am greeted with the low disk space notification due to hundreds of thousands of
"{{< sample-output "Preserving template record with size..." >}}" messages being
generated over the course of the previous 24 hours, as shown in the following
log excerpt:

{{< log-excerpt >}}

```
01/26/2009 02:45:35.43  OWSTIMER.EXE (0x0158)                    0x1424 Windows SharePoint Services    General                        0 Medium   Preserving template record with size 3301, use count 7, key ...
01/26/2009 02:45:35.43  OWSTIMER.EXE (0x0158)                    0x1424 Windows SharePoint Services    General                        0 Medium   Preserving template record with size 2225, use count 59, key ...
01/26/2009 02:45:35.43  OWSTIMER.EXE (0x0158)                    0x1424 Windows SharePoint Services    General                        0 Medium   Preserving template record with size 5728, use count 1, key ...
01/26/2009 02:45:35.43  OWSTIMER.EXE (0x0158)                    0x1424 Windows SharePoint Services    General                        0 Medium   Preserving template record with size 5731, use count 1, key ...
01/26/2009 02:45:35.43  OWSTIMER.EXE (0x0158)                    0x1424 Windows SharePoint Services    General                        0 Medium   Preserving template record with size 5732, use count 1, key ...
```

{{< /log-excerpt >}}

The good news is that the "Preserving template record with size..." issue only
seems to occur when system resources are fairly tight; for example, on my VM
with 2GB of RAM, running Visual Studio 2008, MOSS 2007, and SQL Server 2008
(even though I constrain SQL to 512MB of memory).

In other words, assuming you aren't trying to run your Production SharePoint
environment on a single server with a mere 2GB of RAM, then this shouldn't be an
issue.

The better news is that the "Preserving template record with size..." issue has
been duly noted by the product team (in other words, a bug has been created) and
it appears the logging level for this event will be changed in "O14" (i.e. the
next version of SharePoint) to avoid incessantly spewing this message with the
default logging settings.

As for addressing my daily "log purging" needs, I simply created the following
script and dropped it in my SharePoint
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) folder
(\NotBackedUp\Public\Toolbox\SharePoint\Scripts\Delete SharePoint Logs.cmd):

```
@echo off
setlocal

set LogFolder=%ProgramFiles%\Common Files\Microsoft Shared\web server extensions\12\LOGS

echo This will delete all of the log files in the following folder:
echo    %LogFolder%
echo Are you sure you want to do this? (Press CTRL+C to exit)
pause

del /q "%LogFolder%\*.log"
```

Thus whenever I am greeted with the low disk space notification message, I
simply open up my Toolbox, double-click this script, and then press {{< kbd
"Enter" >}} to quickly purge my log files. Once I do this, I see that the free
space on my C: drive goes from 100-200MB of free space to around 1.4GB of free
space.

Not exactly what I would call "elegant", but as I've repeatedly said before, I
like to keep things simple.

Another useful tip that I wanted to pass along is effectively viewing very large
text files (for example, to determine exactly *why* SharePoint is creating 1.3GB
of log messages on a daily basis). While you can certainly *try* opening a
30-50MB file in Notepad, I certainly don't recommend it. Personally, I keep a
copy of [LtfViewr4U](http://search.live.com/results.aspx?q=LtfViewr4U) in my
Toolbox for just such an occasion.

> **Update (2009-06-02)**
>
> As noted in the comments, this issue appears to be fixed in the
> [April 2009 Cumulative Update](http://support.microsoft.com/kb/968850) for
> Windows SharePoint Services.
