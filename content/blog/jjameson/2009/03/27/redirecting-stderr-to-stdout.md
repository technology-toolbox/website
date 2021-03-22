---
title: Redirecting stderr to stdout
date: 2009-03-27T06:29:00-06:00
excerpt:
  Yesterday I replied to an email from a teammate in which I incorrectly stated
  that you can't redirect stderr to stdout in DOS -- er, I mean a command window
  in Microsoft Windows. I would have sworn the last time I tried something like
  the following...
aliases:
  [
    "/blog/jjameson/archive/2009/03/26/redirecting-stderr-to-stdout.aspx",
    "/blog/jjameson/archive/2009/03/27/redirecting-stderr-to-stdout.aspx",
  ]
draft: true
categories: ["Development", "Infrastructure"]
tags: ["Core Development", "Windows Vista", "Windows Server"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/03/27/redirecting-stderr-to-stdout.aspx"
---

Yesterday I replied to an email from a teammate in which I incorrectly stated
that you can't redirect `stderr` to `stdout` in DOS -- er, I mean a _command
window_ in Microsoft Windows.

I would have sworn the last time I tried something like the following in Windows
Server 2003 (a couple of years ago), I got an error message:

```Console
"Redeploy Features.cmd" > tmp.log 2>&1
```

Fortunately another teammate on the thread,
[Prashant Nayak](http://blogs.msdn.com/pnayak), experimented with this and
confirmed that it actually _does_ work. Thanks, Prashant, for setting the record
straight!
