---
title: "ProcDump - An Easier Way to Create a Mini-Dump"
date: 2010-12-03T21:49:00+08:00
excerpt: "In a previous post , I mentioned an issue I've been having with Expression Web 4 crashing on me. In that post, I mentioned a few ways that you can create a mini-dump for a process (e.g. with Visual Studio, WinDbg, or ADPlus). 
 A couple of weeks ago..."
draft: true
categories: ["Development", "My System"]
tags: ["Core Development", "Debugging", "Toolbox"]
---

> **Note**
> 
> 
> 	This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/12/04/procdump-an-easier-way-to-create-a-mini-dump.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/12/04/procdump-an-easier-way-to-create-a-mini-dump.aspx)
> 
> 
> Since
> 	[I no longer work for Microsoft](/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx), I have copied it here in case that blog 
> 	ever goes away.


In a previous [post](/blog/jjameson/archive/2010/10/24/recovering-your-work-after-an-expression-web-crash.aspx), I mentioned an issue I've been having with Expression Web 4 crashing on  me. In that post, I mentioned a few ways that you can create a mini-dump for a process  (e.g. with Visual Studio, WinDbg, or ADPlus).

A couple of weeks ago, one of the developers on the Expression Web team enlightened  me regarding a new Sysinternals tool called [ProcDump](http://technet.microsoft.com/en-us/sysinternals/dd996900.aspx)  that makes this even easier. I've since made ProcDump a permanent addition to my [Toolbox](/blog/jjameson/archive/2007/03/22/backedup-and-notbackedup.aspx).

Here's a little script that I keep on my desktop that shows how to write a mini-dump  when an unhandled exception occurs in Expression Web:



    cd C:\NotBackedUp\Temp\ExpressionWeb
    
    C:\NotBackedUp\Public\Toolbox\procdump.exe -e -ma ExpressionWeb.exe



Note that there are many more command-line options available for ProcDump (for  example, "-h" can be specified to monitor for a hung process). If you are a developer,  I highly recommend downloading ProcDump today.

Kudos to Mark Russinovich for yet another invaluable tool!

