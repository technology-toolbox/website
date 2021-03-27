---
title: Deleting Variation Labels Does Not Delete Underlying Sites
date: 2007-11-28T08:01:00-07:00
excerpt:
  I've received a number of responses to my series on the problems we
  encountered with Microsoft Office SharePoint Server (MOSS) 2007 variations.
  Several people have inquired about how to disable variations without losing
  their content. It's actually quite...
aliases:
  [
    "/blog/jjameson/archive/2007/11/27/deleting-variation-labels-does-not-delete-underlying-sites.aspx",
    "/blog/jjameson/archive/2007/11/28/deleting-variation-labels-does-not-delete-underlying-sites.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/11/28/deleting-variation-labels-does-not-delete-underlying-sites.aspx"
---

I've received a number of responses to my
[series](/blog/jjameson/2007/10/30/dumping-moss-2007-variations-part-1) on the
problems we encountered with Microsoft Office SharePoint Server (MOSS) 2007
variations. Several people have inquired about how to disable variations without
losing their content. It's actually quite simple: simply delete the variation
labels. For example, if you follow my repro steps in
[part 1](http://blogs.msdn.com/controlpanel/blogs/I'm%20not%20sure%20if%20adding%20labels%20corresponding%20to%20existing%20sites%20will%20be%20supported),
you can subsequently delete the **ja-JP** and **en-US** variation labels, but
the corresponding sites will not be deleted (i.e. you will still have
**/en-US/foo** and **/ja-JP/foo** sites).

Note that in the time since my original posts, the issues have been repro'ed by
PSS and it sounds like
[several hotfixes](http://blogs.technet.com/stefan_gossner/archive/2007/11/15/some-comments-on-common-variation-problems.aspx)
are in the works. If you can "hang in there" for a few more months, the
variations story should improve dramatically.

