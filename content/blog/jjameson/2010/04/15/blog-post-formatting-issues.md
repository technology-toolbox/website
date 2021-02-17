---
title: "Blog Post Formatting Issues"
date: 2010-04-15T05:16:00-07:00
excerpt: "Earlier this week, a colleague of mine was building out his own version of the Jameson Datacenter based on a variety of posts I've written in the past. Over an IM conversation, he mentioned that some of my posts tend to run off the page -- making them..."
aliases: ["/blog/jjameson/archive/2010/04/14/blog-post-formatting-issues.aspx"]
draft: true
categories: ["Personal"]
tags: ["Personal"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/15/blog-post-formatting-issues.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/15/blog-post-formatting-issues.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Earlier this week, a colleague of mine was building out his own version of the [Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter) based on a variety of posts I've written in the past. Over an IM conversation, he mentioned that some of my posts tend to run off the page -- making them difficult to read unless you can resize the browser window wide enough to accommodate the text.

I told him that I was aware of the issue, but that I had only seen it when viewing multiple posts at a time. I mentioned that it seemed to be an issue with the structure of the HTML emitted by the Community Server template that doesn't always "play nicely" with `<pre>` elements (which I tend to use somewhat heavily). The issue is caused by very long lines of preformatted text.

I looked into the issue sometime last year, but ended up punting it because I found that if the content of one of my blog posts was scrolling off the right-side of the page (obscured by the navigation elements on the right), then I could simply click the individual post (to view just that post) and the problem always went away.

If you encounter similar issues when reading my blog, please use this workaround for the time being (or forever, if I never get around to figuring out which CSS rule I need to add or tweak).

Sorry for any inconvenience -- but on the other hand, are you aware that you can buy a decent 24" flat-panel LCD monitor from [newegg](http://www.newegg.com/) for less than 200 bucks? ;-)

By the way, if you are a real life CSS-wizard (I only play one on TV) and you feel like taking a look at this on your own dime, then please go right ahead. I'll be sure to credit you if you send me the CSS changes that I need to make ;-)

