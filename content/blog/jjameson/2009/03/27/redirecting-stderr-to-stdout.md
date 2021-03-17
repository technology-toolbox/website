---
title: "Redirecting stderr to stdout"
date: 2009-03-27T06:29:00-06:00
excerpt: "Yesterday I replied to an email from a teammate in which I incorrectly stated that you can't redirect stderr to stdout in DOS -- er, I mean a command window in Microsoft Windows. 
 I would have sworn the last time I tried something like the following..."
aliases: ["/blog/jjameson/archive/2009/03/26/redirecting-stderr-to-stdout.aspx", "/blog/jjameson/archive/2009/03/27/redirecting-stderr-to-stdout.aspx"]
draft: true
categories: ["Development", "Infrastructure"]
tags: ["Core Development", "Windows Vista", "Windows Server"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/27/redirecting-stderr-to-stdout.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/27/redirecting-stderr-to-stdout.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Yesterday I replied to an email from a teammate in which I incorrectly stated
that you can't redirect `stderr` to `stdout` in DOS -- er, I mean a *command
window* in Microsoft Windows.

I would have sworn the last time I tried something like the following in Windows
Server 2003 (a couple of years ago), I got an error message:

```
"Redeploy Features.cmd" > tmp.log 2>&1
```

Fortunately another teammate on the thread,
[Prashant Nayak](http://blogs.msdn.com/pnayak), experimented with this and
confirmed that it actually *does* work. Thanks, Prashant, for setting the record
straight!
