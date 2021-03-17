---
title: "Troubleshooting Layout Problems with Safari"
date: 2009-10-29T04:57:00-06:00
excerpt: "I discovered a rather nasty UI bug last week with the new portal we are building for a customer. Unfortunately, the layout issue only occurred in the Safari browser. Even worse, I discovered it only a day before the CEO of customer discovered it himself..."
aliases: ["/blog/jjameson/archive/2009/10/28/troubleshooting-layout-problems-with-safari.aspx", "/blog/jjameson/archive/2009/10/29/troubleshooting-layout-problems-with-safari.aspx"]
draft: true
categories: ["Development"]
tags: ["Web Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/29/troubleshooting-layout-problems-with-safari.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/29/troubleshooting-layout-problems-with-safari.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

I discovered a rather nasty UI bug last week with the new portal we are building
for a customer. Unfortunately, the layout issue only occurred in the Safari
browser. Even worse, I discovered it only a day before the CEO of customer
discovered it himself on his Mac! Ouch.

Note that I typically test Web sites using Internet Explorer and Firefox. In
fact, that's what our statement of work contractually obligates for this project
(and not IE6, thankfully, but rather IE7 and IE8).

I've occasionally tested with Opera and Safari in the past, but in general I've
found this to be unnecessary because most of these browsers have strong support
for modern [Web standards](http://en.wikipedia.org/wiki/Web_standards).

However, this week I discovered that a page that looks good in Internet Explorer
and Firefox doesn't necessarily look good in Safari.

Since I didn't have access to my faithful Firebug or Internet Explorer Developer
Tools, I was initially stumped on troubleshooting the problem. However, a little
bit of searching uncovered the following gem:

{{< reference title="Firebug For Safari"
linkHref="http://vision-media.ca/resources/misc/firefox-for-safari" >}}

Unlike many of the other search results that I looked at, this one doesn't
actually discuss using Firebug in Safari (which I was reluctant to try because,
from what I read, all of the CSS rules are read-only when you use Firebug with
Safari). Rather, this page shows the similar set of tools that Apple provides
with their browser. It also shows how to easily enable the Safari developer
tools without hacking a preferences file (which I initially tried locating on my
Windows VM using instructions based on a Mac, but I quickly gave up on that
effort).

After spending about 45 minutes with the Safari developer tools, I was finally
able to track down the `float` CSS rule that was causing my Login Form Web Part
to be displayed *behind* the `<h1>` heading for the main content of the page in
Safari.

A few minutes later and I had my fix checked-in to TFS. Whew!
