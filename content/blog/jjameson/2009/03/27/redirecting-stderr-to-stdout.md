---
title: "Redirecting stderr to stdout"
date: 2009-03-27T00:29:00+08:00
excerpt: "Yesterday I replied to an email from a teammate in which I incorrectly stated that you can't redirect stderr to stdout in DOS -- er, I mean a command window in Microsoft Windows. 
 I would have sworn the last time I tried something like the following..."
draft: true
categories: ["Development", "Infrastructure"]
tags: ["Core Development", "Windows Vista", "Windows Server"]
---

> **Note**
> 
> 
> 	This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/27/redirecting-stderr-to-stdout.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/27/redirecting-stderr-to-stdout.aspx)
> 
> 
> Since
> 	[I no longer work for Microsoft](/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx), I have copied it here in case that blog 
> 	ever goes away.


Yesterday I replied to an email from a teammate in which I incorrectly stated  that you can't redirect `stderr` to `stdout` in DOS -- er,  I mean a *command window* in Microsoft Windows.

I would have sworn the last time I tried something like the following in Windows  Server 2003 (a couple of years ago), I got an error message:

<kbd>"Redeploy Features.cmd" &gt; tmp.log 2&gt;&amp;1</kbd>

Fortunately another teammate on the thread, [Prashant Nayak](http://blogs.msdn.com/pnayak), experimented with this  and confirmed that it actually *does* work. Thanks, Prashant, for setting  the record straight!

