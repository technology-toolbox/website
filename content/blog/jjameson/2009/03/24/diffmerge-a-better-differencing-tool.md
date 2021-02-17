---
title: "DiffMerge - A Better Differencing Tool"
date: 2009-03-24T03:01:00-07:00
excerpt: "Last summer, I added DiffMerge to my Toolbox and I haven't used WinDiff since. 
 DiffMerge can do everything WinDiff can, plus a whole lot more -- like intra-line highlighting, merging, and comparing files using configurable rulesets (although you'll..."
aliases: ["/blog/jjameson/archive/2009/03/23/diffmerge-a-better-differencing-tool.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "Visual Studio", "TFS", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/24/diffmerge-a-better-differencing-tool.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/24/diffmerge-a-better-differencing-tool.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Last summer, I added [DiffMerge](http://www.sourcegear.com/diffmerge/) to my [Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) and I haven't used WinDiff since.

DiffMerge can do everything WinDiff can, plus a whole lot more -- like intra-line highlighting, merging, and comparing files using configurable rulesets (although you'll likely never need any more than those that come "out-of-the-box" with DiffMerge).

And since the folks at SourceGear don't mind giving away DiffMerge for free, there's no reason *not* to use it!

Heck, I even configure Visual Studio to use DiffMerge as my comparison tool instead of the default tool that comes with Team Foundation Server (TFS).

To customize your TFS comparison tool in Visual Studio 2008:

1. On the **Tools** menu, click **Options...**
2. In the **Options** window, expand **Source Control**, select **Visual Studio Team Foundation Server**, and then click **Configure User Tools...**
3. In the **Configure User Tools** window, click the Add... or Modify... button as necessary to configure the following:

- Extension: **.\***
- Operation: **Compare**
- Command: **C:\NotBackedUp\Public\Toolbox\DiffMerge\DiffMerge.exe**
- Arguments: **%1 %2**

Obviously, you may need to adjust the path to **DiffMerge.exe** as necessary for your environment.

In the past, I tried other tools like BeyondCompare -- but I preferred the simplicity of WinDiff. Then, a couple of years ago, one of my peers pointed me to an internal tool called "Odd" that would do intra-line highlighting. However, I couldn't get over the fact that this was an internal tool only -- and thus not something I could recommend to customers.

When I stumbled across DiffMerge, I was delighted and I haven't looked back since.

Perhaps Visual Studio 2010 will ship with a vastly improved differencing tool, but in the meantime -- or just in case it doesn't -- DiffMerge is my tool of choice.

