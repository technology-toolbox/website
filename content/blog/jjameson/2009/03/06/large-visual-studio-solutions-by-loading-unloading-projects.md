---
title: Large Visual Studio Solutions and Loading/Unloading Projects
date: 2009-03-06T10:03:00-07:00
description:
  "As I noted in my previous post , I typically work with \"large\" Visual
  Studio solutions. Note that I put this in quotes, because the definition of
  \"large\" will likely vary widely based on your individual experience. Note
  that I'm not referring to \"large..."
aliases:
  [
    "/blog/jjameson/archive/2009/03/05/large-visual-studio-solutions-by-loading-unloading-projects.aspx",
    "/blog/jjameson/archive/2009/03/06/large-visual-studio-solutions-by-loading-unloading-projects.aspx",
  ]
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Core Development", "WSS v3", "Visual Studio"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/03/06/large-visual-studio-solutions-by-loading-unloading-projects.aspx"
---

As I noted in my
[previous post](/blog/jjameson/2009/03/06/why-i-m-not-a-fan-of-wspbuilder), I
typically work with "large" Visual Studio solutions. Note that I put this in
quotes, because the definition of "large" will likely vary widely based on your
individual experience. Note that I'm not referring to "large" like the source
for the .NET Framework itself, but rather "large" like most enterprise customers
that I typically engage with. If you want a number, then let's say somewhere in
the range of 50-75 projects in a single Visual Studio solution.

On my previous project, we had 52 projects in our solution. On my current
project, there are 30.

You may very well have more or less, but the following concepts and tips should
still apply. If you are on a development team with several hundred projects,
then either you are a member of the NetFx team or you more than likely already
know about what I'm about to tell you in this post ;-)

If you work with multiple Visual Studio solutions that only have a handful of
projects, but this is simply because your team decided to break up one big
solution into several smaller ones, then this post is intended especially for
you.

As you start adding more and more projects to a solution, you'll inevitably
discover the following to be true:

- Load time increases (i.e. the time it takes to start Visual Studio and open
  your solution)
- Incremental build time increases, even when the only changes that have been
  made are in "leaf" projects (i.e. projects that don't reference other
  projects)

Obviously we should expect the build time to take a while whenever we make a
change in one project that is referenced by many others (e.g.
[a "CoreServices" project](/blog/jjameson/2007/04/18/structure-visual-studio-solutions)
like I've described in the past).

If, like me, you've grown accustomed to Test Driven Development (TDD) however,
then you know that it's essential to minimize the
["Red, Green, Refactor"](http://msdn.microsoft.com/en-us/library/aa730844%28VS.80%29.aspx)
cycle. If your
[incremental build time requires 28 seconds](/blog/jjameson/2009/03/06/why-i-m-not-a-fan-of-wspbuilder),
then your developer productivity is going to take a hit in a big, big way.

So, what can we do to mitigate these issues?

One approach is to create multiple Visual Studio solutions with a subset of
projects. Developers could then open the specific solution (or, possibly,
_solutions_ -- plural) containing a subset of the projects they need to work on
at any given time.

I really don't like -- or obviously recommend -- this approach, because it can
be very problematic (for example, having to synchronize changes to the build
configuration). Even worse, though, is that depending on how you partition your
solution, you may end up having to switch from project references to file
references for assembly dependencies. Anyone who has ever used file references
for their own code should tell you that this is something to avoid.

Note that the recommendation to use a single, "master" solution is nothing new.
We've had this prescriptive guidance out for a number of years now (please
forgive the fact that MSDN still refers to using Visual SourceSafe instead of
Team Foundation Server in
[this article](http://msdn.microsoft.com/en-us/library/ms998208.aspx) -- there
may very well be an updated article that I simply haven't bothered to take the
time and find).

Unfortunately, what was fundamentally missing from the original prescriptive
guidance was any mention of the feature in Visual Studio that allows you to
effectively work with a solution containing numerous projects. This has become
critical in light of the shift to TDD.

If you right-click a project in Visual Studio, you'll see the **Unload Project**
option way down near the bottom of the context menu. When you unload a project,
Visual Studio completely ignores the project (and all of the items in the
project). Unloaded projects are not compiled when you press {{< kbd
"CTRL+SHIFT+B" >}}, which can substantially reduce your incremental build time,
thereby making you a much more productive TDD developer! You will also find the
time required to open the solution can be greatly reduced by unloading projects.

Whenever you need to change something in an unloaded project, simply right-click
the project and click **Reload Project**.

Unloading a project is also useful whenever you need to edit the MSBuild file --
for example, to
[rebuild a CAB or WSP whenever a dependency changes](/blog/jjameson/2008/04/10/a-better-way-to-build-sharepoint-solution-packages-and-cab-files).

An important thing to understand about loading and unloading projects is that
the settings are stored in the solutions options (.suo) file which is specific
to each developer and should never be stored in source control. In other words,
if I unload a project in my workspace, this has no effect on other team members.
Removing a project, on the other hand, changes the solution file (.sln) itself
and therefore impacts other members of the development team.

Note that you can quickly unload or reload multiple projects at a time by using
solution folders within Visual Studio. If you right-click a solution folder, you
will see the option to **Unload Projects in Solution Folder**. I typically
structure my Visual Studio solutions in a hierarchical fashion (i.e. nested
solution folders) to make it very easy to load or unload various "subsystems" or
features.

Also note that you can use Visual Studio macros to quickly unload or reload all
of the projects in a solution with a single click (well, actually a
double-click, but you get the point).

I'll share the macros that I developed and have been using for years in
[a separate post](/blog/jjameson/2009/03/11/visual-studio-macros-for-unloading-reloading-projects).

There is one caveat that you should be aware of when unloading projects. Visual
Studio warns you when you attempt to unload projects with pending changes in
source control. Generally speaking, you want to avoid proceeding whenever Visual
Studio displays the warning for this scenario.

Note that modified files in unloaded projects are still shown in the **Pending
Changes** window when using TFS as the source code control provider. However,
this isn't true for all providers (e.g. Visual SourceSafe).

I've been using the loading/unloading project feature since my old Visual C++
4.2 days, but -- at least based on my experience -- there seem to be a number of
developers who are unaware of this great feature in Visual Studio. If I recall
correctly, this was missing in the original Visual Studio.NET but, thankfully,
was added back in a subsequent version, either Visual Studio .NET 2003 or Visual
Studio 2005 (honestly, it's been too long to remember). Perhaps that the reason
why the original prescriptive guidance on MSDN makes no mention of
unloading/loading projects. Thank goodness it's back!
