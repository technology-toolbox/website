---
title: "New Blog Template and Styling"
date: 2010-04-19T14:25:00-07:00
excerpt: "Last Thursday, I mentioned a problem that would occasionally occur with the formatting on my blog posts (but only when viewing the default page -- listing the most recent posts -- and not when viewing individual posts). Tonight I thought I should spend..."
aliases: ["/blog/jjameson/archive/2010/04/19/new-blog-template-and-styling.aspx"]
draft: true
categories: ["Personal"]
tags: ["Personal"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/19/new-blog-template-and-styling.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/19/new-blog-template-and-styling.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Last Thursday, I mentioned a problem that would occasionally occur with the [formatting on my blog posts](/blog/jjameson/2010/04/14/blog-post-formatting-issues) (but only when viewing the default page -- listing the most recent posts -- and not when viewing individual posts). Tonight I thought I should spend a little more effort trying to resolve the issue.

However, rather than trying to come up with a CSS hack to avoid the problem with viewing multiple posts (when one of the posts contains a long line of preformatted text), I decided instead to see if the problem was specific to the Community Server template that I was using.

Over the last few months, I've noticed more and more MSDN bloggers switching over to a new look-and-feel based on the recently overhauled MSDN site (in anticipation of the recent launch of Visual Studio 2010). I first noticed the new styling on [Jim Lamb's blog](http://blogs.msdn.com/jimlamb/), and since then I've seen numerous other MSDN blogs updated with similar styling.

Tonight I switched my blog from the **default** template in Community Server to the **Simple - right sidebar** template. I then snarfed the custom CSS from Jim's blog [thanks, Jim!], pasted it before the custom CSS rules I had previously created for my blog, and then made a few tweaks (for example to use a width of 960px instead of 900 for the content).

After checking the new layout over a little bit, I decided to keep the change in place. I confirmed the issue with some posts scrolling off the right side of the page is now a thing of the past, although I also noticed that I'm getting horizontal scrollbars in some of the code blocks that weren't there before. [This is one of the biggest trade-offs when switching from a fluid layout (e.g. the default Community Server template I was using before) to a fixed layout (e.g. the "Simple" template that I'm now using along with the corresponding CSS rules to constrain the width of the content.]

Over the next few days, I'll tweak the formatting a little to resolve the new layout issues.

