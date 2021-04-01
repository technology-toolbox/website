---
title: ProcDump - An Easier Way to Create a Mini-Dump
date: 2010-12-04T04:49:00-07:00
excerpt:
  In a previous post , I mentioned an issue I've been having with Expression Web
  4 crashing on me. In that post, I mentioned a few ways that you can create a
  mini-dump for a process (e.g. with Visual Studio, WinDbg, or ADPlus). A couple
  of weeks ago...
aliases:
  [
    "/blog/jjameson/archive/2010/12/03/procdump-an-easier-way-to-create-a-mini-dump.aspx",
    "/blog/jjameson/archive/2010/12/04/procdump-an-easier-way-to-create-a-mini-dump.aspx",
  ]
categories: ["Development", "My System"]
tags: ["Core Development", "Debugging", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/12/04/procdump-an-easier-way-to-create-a-mini-dump.aspx"
---

In a previous
[post](/blog/jjameson/2010/10/24/recovering-your-work-after-an-expression-web-crash),
I mentioned an issue I've been having with Expression Web 4 crashing on me. In
that post, I mentioned a few ways that you can create a mini-dump for a process
(e.g. with Visual Studio, WinDbg, or ADPlus).

A couple of weeks ago, one of the developers on the Expression Web team
enlightened me regarding a new Sysinternals tool called
[ProcDump](http://technet.microsoft.com/en-us/sysinternals/dd996900.aspx) that
makes this even easier. I've since made ProcDump a permanent addition to my
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup).

Here's a little script that I keep on my desktop that shows how to write a
mini-dump when an unhandled exception occurs in Expression Web:

{{< console-block >}}
cd C:\NotBackedUp\Temp\ExpressionWeb

C:\NotBackedUp\Public\Toolbox\procdump.exe -e -ma ExpressionWeb.exe
{{< /console-block >}}

Note that there are many more command-line options available for ProcDump (for
example, "-h" can be specified to monitor for a hung process). If you are a
developer, I highly recommend downloading ProcDump today.

Kudos to Mark Russinovich for yet another invaluable tool!
