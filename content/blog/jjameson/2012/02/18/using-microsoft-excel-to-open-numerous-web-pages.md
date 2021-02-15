---
title: "Using Microsoft Excel to open numerous Web pages"
date: 2012-02-18T23:44:52-07:00
excerpt: "Think you can just add a HYPERLINK column in Excel and easily open dozens of Web pages at a time? Think again."
draft: true
categories: ["My System"]
tags: ["My System", "Web Development"]
---

I've been meaning to cleanup my Windows desktop for months. So this
morning when I noticed
[another addition to my desktop](/blog/jjameson/2012/02/19/stop-putting-shortcuts-on-my-windows-desktop), I thought "no time like the present."

While I'm not one for adding dozens of shortcuts to my desktop, I have a
bad habit of creating shortcuts for "to-do" items. For example, when I have
an idea to blog about some topic or useful tip, I sometimes create an item on
my desktop to remind me to create a post.

The problem is, some of these shortcuts linger for months. Case in point:
the Temp.xlsx file that has been parked on my Windows desktop since last October.

If you've been following my blog in recent months, then you'll recall that
when I left Microsoft, I migrated the content from my MSDN blog to my new blog
on TechnologyToolbox.com. During that process, I wanted to review each migrated
post to ensure it rendered as expected.

The problem was the sheer number of items to review (approximately 300).

Typically when I have to do some repetitive task a large number of times,
I turn to Excel to save some precious time. Thus when I needed to review about
300 blog posts as quickly as possible, I fired up Excel.

Using a simple SQL query against the Subtext database, I copied the server-relative
URLs of the blog posts into Excel and added a formula in the adjacent column
to create a hyperlink for each post (using the
[HYPERLINK formula](http://office.microsoft.com/en-us/excel-help/hyperlink-HP005209116.aspx)). This allowed me to easily click a batch of posts to
review...or so I thought.

The problem I discovered is that some of the hyperlinks would fail to open
-- silently fail, mind you. One of the worst possible user experiences I can
imagine.

Why some links failed to open, I can't say for sure -- but I speculate it
is due to some "bad code" in the DDE communication between Excel and Internet
Explorer (assuming, of course, that DDE is the mechanism being used in this
scenario).

Keep in mind that I was only trying to open 10 links at a time, so it wasn't
very difficult to notice when only 8 or 9 pages opened in Internet Explorer.

Not wanting to "punt" the use of Excel entirely -- since clicking through
300+ links on the website would have taken a looooong time -- or attempt to
"throttle" my click rate in Excel to avoid the bug, I decided to try to circumvent
the issue by using the command line instead.

In the **A** column, I had values like:

> /archive/2007/03/03/who-is-this-guy.aspx

The **B** column contained formulas based on my original attempt
using HYPERLINK:

> `=HYPERLINK("http://www-dev.technologytoolbox.com/blog/jjameson" &  A1)`

In the **C** column I added a new formula:

> `="""C:\Program Files\Internet Explorer\iexplore.exe"" -framemerging  " &B1`

By copying the "computed" values from the **C** column into
a command prompt, I noticed that I could open numerous pages in Internet Explorer
without issue.

