---
title: "Latest Version of Opera Ignores Hosts File"
date: 2009-10-28T23:17:00+08:00
excerpt: "As I mentioned in my previous post , I discovered a rather nasty UI bug last week with the new portal we are building for a customer. Unfortunately, the layout issue only occurred in the Safari browser. 
 Since I couldn't repro the issue in Internet..."
draft: true
categories: ["Development"]
tags: ["Web Development"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:  
>   
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/29/latest-version-of-opera-ignores-hosts-file.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/29/latest-version-of-opera-ignores-hosts-file.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.


As I mentioned in my [previous post](/blog/jjameson/2009/10/29/troubleshooting-layout-problems-with-safari), I discovered a rather nasty UI bug last week with the new portal we are building for a customer. Unfortunately, the layout issue only occurred in the Safari browser.

Since I couldn't repro the issue in Internet Explorer or Firefox, I decided to see if Opera exhibited the same problem as Safari. Consequently, I downloaded and installed the latest version of Opera (note that I've rebuilt both my desktop and development VM since the last time I used Opera).

Unfortunately. I discovered there's a known bug with Opera version 10 that causes it to ignore your local hosts file (e.g. %SystemRoot%\drivers\etc\hosts).

Oh c'mon, Opera Team, you've got to be kidding!

How can you expect a developer to even try your browser if he can't browse to a site in his local development environment using a fully qualified domain name (e.g. http://www-local.fabrikam.com)?

Needless to say, I didn't spend much time trying to workaround the issue. My prediction: Opera is all but extinct in another year or two.

That's too bad. From what I remember, Opera used to be a pretty good browser -- and maybe it still is, aside from the bug with the hosts file. It was certainly years ahead of IE6 in terms of Web standards compliance.

Does anyone know the current browser stats? It wouldn't surprise me if Opera was surpassed by Safari some time ago.

