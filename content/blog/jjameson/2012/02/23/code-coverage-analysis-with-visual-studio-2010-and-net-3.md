---
title: Code coverage analysis with Visual Studio 2010 and .NET 3.5 solutions (e.g. SharePoint 2010)
date: 2012-02-23T02:08:16-07:00
excerpt:
  "It takes a little more work than expected, but you actually can \"have your
  cake and eat it too\" when it comes to Visual Studio 2010 code coverage
  analysis and .NET Framework 3.5 solutions (e.g. SharePoint 2010)."
aliases:
  [
    "/blog/jjameson/archive/2012/02/22/code-coverage-analysis-with-visual-studio-2010-and-net-3.aspx",
    "/blog/jjameson/archive/2012/02/23/code-coverage-analysis-with-visual-studio-2010-and-net-3.aspx",
  ]
draft: true
categories: ["Development", "SharePoint"]
tags: ["Core Development", "MOSS 2007", "SharePoint 2010", "Visual Studio"]
---

Sometimes I wish I didn't develop in the world of SharePoint. Whenever I have
the luxury of developing plain ol' ASP.NET solutions on .NET Framework 4, I am
amazed at how _easy_ things are.

In my experience doing .NET development with Visual Studio, things typically
_just work_. When doing SharePoint development, on the other hand, things often
behave, well, let's just say _erratically_.

For example, back in the days when I worked primarily in Visual Studio 2008 and
Microsoft Office SharePoint Server (MOSS) 2007, I discovered an
[issue when running unit tests on a 64-bit VM](/blog/jjameson/2009/10/08/web-application-at-could-not-be-found-error-on-moss-2007-x64).
Fortunately I was able to workaround that bug.

Thanks to a couple of poor decisions by Microsoft product teams (specifically,
targeting SharePoint 2010 to .NET 3.5 x64 only, and
[restricting test projects in Visual Studio 2010 to .NET Framework 4](/blog/jjameson/2010/04/28/test-projects-in-visual-studio-2010-must-target-net-framework-4)),
it wasn't until Visual Studio 2010 Service Pack 1 that you could actually run
unit tests against SharePoint 2010 solutions.

However, even after getting back the ability to run tests in Visual Studio
against custom code written for SharePoint solutions, I noticed that some things
still didn't work. For example, code coverage analysis is trivial to enable in
Visual Studio, so there is really no reason not to periodically review how much
code your tests are covering, right?

Unfortunately, the instructions in the Visual Studio documentation for
[configuring code coverage](http://msdn.microsoft.com/en-us/library/dd504821.aspx)
only work when your assemblies target .NET Framework 4. When developing for
SharePoint 2010, the Visual Studio projects must target .NET Framework 3.5.

While doing some research this morning for this post, I found someone (a
Microsoft MVP, no less!) who implies
[code coverage "just works" when developing for SharePoint 2010](https://msmvps.com/blogs/sundar_narasiman/archive/2011/11/16/enabling-code-coverage-for-sharepoint-2010-automated-unit-tests.aspx)
-- but that certainly hasn't been my experience. Just to be sure I'm not losing
my mind, this morning I followed the steps in the sample
[MSDN walkthrough for creating a unit test for SharePoint](http://msdn.microsoft.com/en-us/library/gg599006.aspx)
and subsequently enabled code coverage inside Visual Studio. This resulted in
the following message from Visual Studio:

{{< div-block "fst-italic" >}}

> Debugging tests running on a remote computer or with code coverage enabled is
> not supported. The tests will be run under the debugger locally and without
> code coverage enabled.

{{< /div-block >}}

Okay, perhaps what that Microsoft MVP meant is that you need to run the unit
tests outside the debugger (e.g. by using Ctrl + F5). Ummm...yeah...I tried that
-- it doesn't work. When I click the Show Code Coverage Results toolbar button
in the **Test Results** window, I see the following:

{{< div-block "fst-italic" >}}

> Cannot find any coverage data (.coverage or .coveragexml) files. Check test
> run details for possible errors.

{{< /div-block >}}

Fortunately, there is a way to perform code coverage analysis when working with
Visual Studio 2010 and projects that target .NET Framework 3.5 (e.g. SharePoint
2010 solutions). However, you have to resort to some command-line tools, as
described in the following post:

{{< reference title="Manually Configure and Run code Coverage"
linkHref="http://blogs.microsoft.co.il/blogs/royrose/archive/2011/08/30/manually-configure-and-run-code-coverage.aspx" >}}

In
[my next post](/blog/jjameson/2012/02/23/use-powershell-to-alleviate-the-pain-of-code-coverage-analysis),
I'll share the PowerShell script that I created to make this additional work
relatively painless. First I need to get my daughter off to school (we are on
delayed start this morning due to 6 more inches of snow -- woohoo!).
