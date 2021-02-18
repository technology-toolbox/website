---
title: "Essential Add-Ins for Team Foundation Server"
date: 2009-10-25T01:32:00-07:00
excerpt: "In a previous post , I mentioned how I use SourceGear's DiffMerge instead of the out-of-the-box tool that comes with Team Foundation Server (which is also called DiffMerge). If you haven't at least evaluated the SourceGear alternative, I definitely advise..."
aliases: ["/blog/jjameson/archive/2009/10/25/essential-add-ins-for-team-foundation-server.aspx"]
draft: true
categories: ["Development"]
tags: ["Visual Studio", "TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/25/essential-add-ins-for-team-foundation-server.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/25/essential-add-ins-for-team-foundation-server.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In a [previous post](/blog/jjameson/2009/03/23/diffmerge-a-better-differencing-tool), I mentioned how I use SourceGear's DiffMerge instead of the out-of-the-box tool that comes with Team Foundation Server (which is also called DiffMerge). If you haven't at least evaluated the SourceGear alternative, I definitely advise you to take a look.

The other "add-in" for TFS that I consider to be essential is the [TFS Quick Search Plugin](http://www.acorns.com.au/projects/vsaddins/).

If you have Web access to your TFS instance (via [Team System Web Access](http://msdn.microsoft.com/en-us/teamsystem/bb980951.aspx)), then quickly locating work items by keywords is actually pretty easy. However if you either don't have access to Team System Web Access or you would simply rather prefer to search for a work item without leaving Visual Studio, then the TFS Quick Search Plugin is definitely the way to go.

Note that it doesn't search all of the fields in all work items, but rather searches the work items currently displayed within Visual Studio. However, if you know a keyword that appears in the title of a work item, this can make it very easy to locate the desired work item.

I tend to use the TFS Quick Search Plugin a lot for the scenario where I am trying to find a duplicate bug and I remember a couple of keywords in the title of the previous bug. It is certainly much faster than scrolling through a few hundred work items in order to find something, or trying to build a query on-the-fly in order to locate a work item.

I am really interested to see the improvements in Visual Studio 2010 and Team Foundation Server 2010, but honestly it's probably going to be a while before I'm able to use that in my day-to-day work. Nevertheless, I hope this kind of work item "quick search" comes out-of-the-box in the next release (as well as a better DiffMerge tool). [I did do some hands-on labs back in February that cover the new branching visualization features in TFS 2010, and I have to say that I absolutely love what I've seen so far.]

