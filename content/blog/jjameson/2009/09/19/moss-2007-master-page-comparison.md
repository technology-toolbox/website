---
title: MOSS 2007 Master Page Comparison
date: 2009-09-19T07:56:00-06:00
description:
  This morning I came across an old (June 2007) Excel spreadsheet that I created
  back when I was working on the Agilent Technologies project. The spreadsheet
  lists the various placeholder elements in both application.master and
  default.master for Microsoft...
aliases:
  [
    "/blog/jjameson/archive/2009/09/18/moss-2007-master-page-comparison.aspx",
    "/blog/jjameson/archive/2009/09/19/moss-2007-master-page-comparison.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/09/19/moss-2007-master-page-comparison.aspx"
---

This morning I came across an old (June 2007) Excel spreadsheet that I created
back when I was working on the Agilent Technologies project. The spreadsheet
lists the various placeholder elements in both application.master and
default.master for Microsoft Office SharePoint Server (MOSS) 2007.

I put this together back when I was trying to figure out how to create a single
master page that could be used for:

<div class="overflow-auto">

- Site pages (e.g. /Library/default.aspx)
- System pages (e.g. /Library/Brochures/Forms/AllItems.aspx)
- Application pages (e.g. /Library/\_layouts/viewlsts.aspx)

</div>

Rather than make you open a spreadsheet attachment, I'll just copy the contents
into a simple table:

<div class="d-sm-none">
  <a href='{{< relref "resources/table-1-popout" >}}' target="_blank">Table 1 - MOSS 2007 Master Page Comparison</a>
  <i class="bi bi-arrow-up-right-square"></i>
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-sm-block">
  {{< include-html "resources/table-1.html" >}}
</div>

I found the differences between the two master pages very interesting, as well
as the fact that a couple of the placeholders are simply appended to the end of
default.master. It made me wonder if the two master pages were originally
supposed to remain in-sync (in terms of the number and names of placeholders),
but began to diverge as milestones approached and the SharePoint team was
pushing to simply get features working rather than strictly adhering to all of
the original design goals. [C'mon now, we all know that in the world of software
development, sometimes you have to do things that you fully intend to "come back
and fix later" but often end up never getting reworked as other, more important,
issues arise.]

What I learned from this analysis was that by creating a master page with all of
the placeholder elements specified in both application.master and
default.master, I would ensure that my custom master page would work for site
pages, system pages, and application pages.

I'll cover the topic of using a custom master page for application pages in a
[separate post](/blog/jjameson/2009/09/20/overriding-application-master-in-moss-2007).
