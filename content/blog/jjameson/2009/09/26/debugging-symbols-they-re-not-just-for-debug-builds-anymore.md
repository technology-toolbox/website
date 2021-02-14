---
title: "Debugging Symbols -- They're Not Just for Debug Builds Anymore"
date: 2009-09-26T02:20:00+08:00
excerpt: "I started another new project this week. 
 Typically one of the first tasks on any new development project is to create a Development Plan that provides consistent guidelines and processes for the Development team. On this new project, another Microsoft..."
draft: true
categories: ["Development"]
tags: ["Core Development", "Debugging"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/26/debugging-symbols-they-re-not-just-for-debug-builds-anymore.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/26/debugging-symbols-they-re-not-just-for-debug-builds-anymore.aspx)
> 
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.


I started another new project this week.

Typically one of the first tasks on any new development project is to create a Development         Plan that provides consistent guidelines and processes for the Development team.         On this new project, another Microsoft consultant had already created a draft of         the Development Plan, but in the process of reviewing it, I added some content from         a Development Plan that I had created a few years ago.

In the process of reviewing my old document, I came across the following:


> ###             Installation
> 
>         ...
>         
> ####             Debug Symbols
> 
> All Debug builds should create symbol files for debugging purposes. These symbols             are included as part of the setup to facilitate debugging in other environments             such as DEV.
> 
> 
> > **Important **
> > 
> >                 Do not include Debug symbols in the Release configuration of the setup projects.


When I read this, I actually let out an audible laugh (okay, I suppose it was more         of a chuckle). It must have been the old C++ developer in me that originally put         this in the Development Plan (thinking you should never provide PDB files in your         Release builds because it makes it all too easy for an outsider to understand your         code).

Well, any .NET developer who has ever fired up Reflector on somebody else's assembly         (which hasn't been obfuscated, obviously) knows very well how easy it is to decompile         -- er, I mean *disassemble* -- source code. In fact, regardless of whether         you have the PDB files, you can actually debug .NET code -- including setting breakpoints         and examining variables -- using tools like [WinDbg](http://www.microsoft.com/whdc/devtools/debugging/default.mspx). I've had to do a little of this in the past and while it's not exactly         easy -- especially for a developer who doesn't use WinDbg frequently -- it does         help out in a pinch when troubleshooting some nasty problem in Production.

However, including debugging symbols (i.e. PDB files) in Release builds certainly         makes debugging .NET code easier. This is a key point that John Robbins makes in         [Debugging Microsoft .NET 2.0 Applications](http://amzn.com/0735622027).         In fact, here's a direct quote from page 38:


> ###             Build All Builds with Debugging Symbols
> 
>         ...build all builds, including release builds, with full debugging symbols. [...]


In other words, the Development Plan should say:


> **Important **
> 
>             Always include Debug symbols in the Release configuration of the setup projects
>             -- or, preferably, make them available from a symbol server.


Regarding John's book...I strongly recommend this book to anyone who considers himself         or herself a "serious developer." It is chock full of great tips and recommendations         for developing .NET solutions.

