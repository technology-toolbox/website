---
title: "Why I'm Not a Fan of WspBuilder"
date: 2009-03-06T01:06:00+08:00
excerpt: "After 3 years, 2 months, and 30 days, my involvement with migrating a large customer from a legacy Web platform to Microsoft Office SharePoint Server (MOSS) 2007 came to an end a few weeks ago. Since then, I have joined another team helping a different..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/06/why-i-m-not-a-fan-of-wspbuilder.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/06/why-i-m-not-a-fan-of-wspbuilder.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

After 3 years, 2 months, and 30 days, my involvement with migrating a large customer  from a legacy Web platform to Microsoft Office SharePoint Server (MOSS) 2007 came  to an end a few weeks ago. Since then, I have joined another team helping a different  customer with a MOSS 2007 solution.

One of the challenges with integrating into a team of developers is handling  various development styles and approaches.

One such issue that quickly came to light on this most recent venture is using [WspBuilder](http://www.codeplex.com/wspbuilder) to create the WSP files  for deploying custom features to a SharePoint farm. The issue isn't with the actual  WSPs created by WspBuilder, but how WspBuilder works --or, rather how it *doesn't*  work. Specifically, I am referring to the lack of true dependency checking (i.e.  determining whether any of the files packaged in the WSP have changed since the  last build) and consequently the impact this can have on build times and developer  productivity.

Note that this issue really applies only to larger SharePoint solutions. If you  typically work with a Visual Studio solution containing only a handful of projects  and one or two WSPs -- then you likely aren't experiencing long build times even  if you are using WspBuilder and you've already read more of this post than you needed  to ;-)

However, if you routinely work with a Visual Studio solution with dozens of projects  and numerous WSPs, then by all means, keep reading!

On my previous project, we had 52 projects in our solution. On this new project,  there were 27 projects when I first joined, and we've since added a few and now  stand at 30 projects.

One of the first issues that I noticed is that the new solution would take a  long time to build, even for trivial changes (for example, adding a comment to a  unit test).

Using a simple [stopwatch utility](http://www.online-stopwatch.com),  I just timed the build for the new solution after adding a comment to a unit test.  I chose this scenario because there are no dependencies on the unit test project  (and thus you would expect the incremental build time to be very short). On the  contrary, the build took *1 minute 52 seconds* from the time I pressed <kbd>CTRL+SHIFT+B</kbd> to the time I received the <samp>"Build: 30 succeeded or
up-to-date, 0 failed, 0 skipped"</samp> message in the **Output** window.

For comparison purposes, I fired up the solution from my previous project and  performed a similar test (i.e. timing the build after adding a comment to a unit  test). For this solution, the build took a mere *28 seconds* from the time  I pressed <kbd>CTRL+SHIFT+B</kbd> to the time I received the <samp>"Build: 51 succeeded,
0 failed, 1 up-to-date, 0 skipped"</samp> message in the **Output**  window. While I'd certainly like it if this were even less than 28 seconds, I'm  also trying to be realistic in light of the thousands of files that comprise the  solution.

Now, I'll be the first to admit that this is somewhat of an ["apples to oranges"](http://en.wikipedia.org/wiki/Apples_to_oranges)  comparison, meaning that the underlying projects (and code) have almost nothing  in common. They were, after all, created by two completely disparate teams with  no common team members (ignoring for the moment that I joined the project a couple  of weeks ago).

However, simply for the sake of comparison, consider that the solution with 30  projects has ~23K lines of code, whereas the solution with 52 projects has ~39K  lines of code (as calculated by the new Code Metrics feature in Visual Studio 2008).  Also note on my previous project, we chose to partition our solution into a fairly  large number of WSPs (17 to be exact), whereas the current team chose to minimize  the number of WSPs that need to be deployed (specifically four).

To summarize:

<caption>Comparison of Visual Studio Solution Build Times</caption>| Visual Studio Solution | Number of Projects | Lines of Code | Number of WSPs | Incremental Build Time |
| --- | --- | --- | --- | --- |
| Solution1 | 30 | 23,110 | 4 | 00:01:52 |
| Solution2 | 52 | 38,905 | 17 | 00:00:28 |
From the data in this table, we can clearly see that something is amiss. The  smaller solution (i.e. with 40% less code and 1/4 as many WSPs) takes almost four  times longer to incrementally build! Ouch.

As I told my new teammates shortly after joining the project, WspBuilder doesn't  have the "smarts" to determine that no work needs to be done when nothing has changed  in any of the items included in a WSP (i.e. there is no need to rebuild the WSP  when you press <kbd>CTRL+SHIFT+B</kbd> and then <kbd>CTRL+SHIFT+B</kbd> again immediately  after the previous build completed). This is effectively the same as using post-build  events in Visual Studio to invoke makecab.exe to package the WSP.

As I pointed out about a year ago, there is actually a much [better way of building WSPs (and CAB files)](/blog/jjameson/2008/04/10/a-better-way-to-build-sharepoint-solution-packages-and-cab-files) by avoiding post-build events and  leveraging true dependency checking in MSBuild.

Note that the purpose of this post is not to bash WspBuilder or say that WspBuilder  should be abandoned around the world. Honestly, prior to joining this team, I had  only read about WspBuilder -- I'd never actually used it before. I love the fact  that it takes the grunt work out of creating and maintaining a Diamond Definition  File (DDF) to define the structure of your CAB file, er, I mean *WSP*.

However, offloading this effort from the development team comes at a very steep  price for larger solutions (i.e. lengthy build times). In speaking with other team  members about this, most of them said they avoided <kbd>CTRL+SHIFT+B</kbd> and instead  would right-click on a specific project and then click **Build** in  order to avoid having to wait for the whole solution to build. They would then manually  copy files to the GAC and/or the "SPDir12" folder (%ProgramFiles%\Common Files\Microsoft  Shared\web server extensions\12).

Up until a few weeks ago, they were also excluding the WspBuilder projects from  the **Debug **configuration (to minimize build times). However, I found  this to be very problematic, since developers may forget to switch to the **Package **configuration when getting latest and redeploying the entire  solution to update their local environment. Even worse, though, is the fact that  the **Package **configuration specifies the **Release
**builds of the various assemblies -- and therefore whenever developers would  rebuild in "package" mode to update their environment, all of the deployed assemblies  would not have DEBUG fined. I'll admit, there are times when developers need to  deploy Release builds to their local environments for debugging purposes, but these  occasions are extremely rare (i.e. a bug that cannot be reproduced in the Debug  build).

We have since configured the **Debug** configuration to package  the WSPs -- thus avoiding the need for developers to routinely toggle the build  configuration. As a result of this, the time it takes to build the **Debug**  configuration increased substantially (and hence the purpose of this post).

Are there ways of mitigating the lengthy build times caused by using WspBuilder  without giving up <kbd>CTRL+SHIFT+B</kbd> and building individual projects? Absolutely!  It relies on using an apparently little-known feature in Visual Studio that has  been around since (at least) the Visual C++ 4.0 days: loading and unloading projects.  I cover this in [a separate post](/blog/jjameson/2009/03/06/large-visual-studio-solutions-by-loading-unloading-projects).

So, when deciding whether or not to use WspBuilder, think about the tradeoffs  between creating and managing the DDF files yourself (which, honestly, is rather  easy once you have a pattern to copy/paste from) versus structuring your SharePoint  projects so that WspBuilder can automatically detect the files and structure when  building the WSP.

Personally speaking -- and again, this is just my opinion here -- I prefer makecab.exe  and hand-crafted DDF files. However, this is obviously influenced by the fact that  I built lots of these on my previous project that I can reference ;-)

> **Update (2009-03-31)**
> 
> Note that I have posted an
> [update on WSPBuilder](/blog/jjameson/2009/03/31/updated-thoughts-on-wspbuilder) based on some feedback I received from Carsten
> Keutmann, the creator of WSPBuilder.

