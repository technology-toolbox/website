---
title: "Branching Strategy in Team Foundation Server"
date: 2009-02-10T06:34:00-07:00
excerpt: "While attending TechReady (an internal Microsoft training conference) last week, I learned a lot -- not only about future versions of our products, but also numerous tips and tricks for current versions. One of the most valuable insights I gained was..."
draft: true
categories: ["Development"]
tags: ["Core Development", "TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/02/10/branching-strategy-in-team-foundation-server.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/02/10/branching-strategy-in-team-foundation-server.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

While attending TechReady (an internal Microsoft training conference) last week,         I learned a lot -- not only about future versions of our products, but also numerous         tips and tricks for current versions. One of the most valuable insights I gained         was from a session on branching in Team Foundation Server (TFS). If you've read         my [previous post on structuring Visual Studio solutions](/blog/jjameson/2007/04/18/structure-visual-studio-solutions), then you know that         I'm a proponent of branching from the start in order to easily support parallel         development and periodically stabilizing for a release.

I had previously read the [TFS Branching
Guidance on CodePlex](http://www.codeplex.com/BranchingGuidance), although I admit it has been a rather long time. Nevertheless,         I still understood the commonly used **Main**, **Dev**,         and **Release** branches. However, I discovered during the session         last week that I had a fundamental misunderstanding about *baseless merges*         in TFS. Prior to last week, I thought a baseless merge occurred when the source         and target do not share any ancestry in the branching         tree. In other words, I thought that as long as the target resides on a branch that         can be traced through some lineage to the source branch, then TFS has sufficient         "knowledge" (i.e. algorithms) to merge changes from the source to the target.

As I stated before, this was a misunderstanding on my part. When I asked a question         about trying to delay the branching for stabilization for as long as possible, the         presenters of the session -- [James Pickell](http://blogs.msdn.com/jampick)         and [Mario Rodriguez](http://blogs.msdn.com/mrod) -- pointed out that         a baseless merge occurs whenever the target is not in a "first level" branch from         the source. In other words, you can automerge from parent-to-child (or child-to-parent),         but you cannot automerge from grandchild-to-grandparent.

So, what does all of the mean in practical terms?

It all comes down to how -- and when -- you actually branch your code.

In the interest of not reinventing the wheel, allow me to pilfer one of the images         from the [updated TFS Branching
Guide 2.0](http://www.codeplex.com/TFSBranchingGuideII):

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Branch-Plan-Basic.png"
alt="Basic branch plan"
height="260"
width="600"
title="Figure 1: Basic branch plan" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Branch-Plan-Basic.png)

Note that this is a basic branch plan, but it is sufficient to demonstrate the concept.         You may very well have a more complex branching strategy (e.g. a large organization         like Microsoft in order to support "feature teams"). However, the essential pattern         still applies.

The question that I asked during the session was essentially why not postpone branching         for as long as possible? In other words, why not branch **Release**         first, and then branch for hotfixes and service packs as necessary? In my mind,         this alternative approach reduces the merge effort by only branching at the point         in time when we actually need a branch (e.g. to stabilize for a release). At some         later point in time after the release when we actually need a hotfix or service         pack branch, then we'll create it from the **Release** branch. In response         to my question, Mario termed this "reactive branching" -- which seems like a very         good way to describe it. To help visualize the difference, I mocked up the following         figure.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Branch-Plan-Problematic.png"
alt="\"Reactive\" branch plan"
height="260"
width="600"
title="Figure 2: \"Reactive\" branch plan" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Branch-Plan-Problematic.png)

There are a couple of significant problems with reactive branching:

- In order to merge code changes from a QFE (a.k.a. hotfix) or service pack back into
  **Main** (and remember, we almost always want the fixes in QFEs to
  be incorporated into **Main**), we would either need to first automerge
  the changes into the **Release** branch and then merge the changes
  into **Main** -- or else perform a baseless merge (which, generally
  speaking, we want to avoid). If we merge the changes into the **Release**
  branch, then that code no longer reflects the actual Release build (e.g. v1.0) but
  rather the Release build plus some number of changes. Yes, we could still use labels
  to "snapshot" the source code for the Release version, but as James pointed out,
  labels can be manipulated and therefore may not be sufficiently "bulletproof" to
  meet your requirements.
- We could still use reactive branching, but avoid merging through **Release**
  by renaming a branch (e.g. the **Release** branch is renamed to **Service Pack**, and a new **Release** branch is created
  and "locked down" from a permissions perspective). However, this requires significantly
  more work with each release -- work that can be avoided altogether by using the
  branching strategy shown in Figure 1.

So, in summary, while it may seem a little backwards to create your **Service
Pack** branch before your **Release** branch, there are         compelling reasons to do so. As for me, I now consider my "reactive branching" strategy         to be a thing of the past.

Thanks to James and Mario for setting me straight!

By the way, if you haven't read the branching guidance created by the VSTS Rangers,         I highly recommend it. Very good stuff.

