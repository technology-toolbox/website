---
title: "C++ Compiler in Visual Studio 2010 Must Target .NET Framework 4"
date: 2010-05-06T23:38:00+08:00
excerpt: "Another \"hiccup\" this week after upgrading my Team Foundation Server (TFS) 2008 environment to TFS 2010 ... 
 This morning I discovered that when you upgrade a managed C++ project from Visual Studio 2008 to Visual Studio 2010, the project is updated..."
draft: true
categories: ["Development"]
tags: ["Core Development", "Visual Studio", "TFS"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/07/c-compiler-in-visual-studio-2010-must-target-net-framework-4.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/07/c-compiler-in-visual-studio-2010-must-target-net-framework-4.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.


Another "hiccup" this week after [upgrading my Team Foundation Server (TFS) 2008 environment to TFS 2010](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview)...

This morning I discovered that when you upgrade a managed C++ project from Visual Studio 2008 to Visual Studio 2010, the project is updated automatically to target .NET Framework 4. Unlike most other project types, you can't just quickly change a project setting in order to target an earlier version of the .NET Framework.

Note that you can force a managed C++ project to continue to target one of the older versions of the .NET Framework, as described in the following:

<cite>Visual Studio 2010 C++ Project Upgrade Guide</cite>
[http://blogs.msdn.com/vcblog/archive/2010/03/02/visual-studio-2010-c-project-upgrade-guide.aspx](http://blogs.msdn.com/vcblog/archive/2010/03/02/visual-studio-2010-c-project-upgrade-guide.aspx)


However, you'll need to have Visual Studio 2008 installed on your build server -- or, presumably, you could choose to install the Microsoft Windows SDK for Windows Server 2008 and .NET Framework 3.5 instead (as I described in [one of my blog posts last year](/blog/jjameson/2009/11/07/compiling-c-projects-with-team-foundation-build)).

Instead of tweaking the build (and having to re-install the SDK or Visual Studio 2008 on the build server), I chose instead to upgrade the other projects (that reference the managed C++ project) to .NET Framework 4.

As I've mentioned in the past, I'm not doing very much in C++ these days, and my primary objective was to get my "Toolbox" solution to build in my upgraded environment.

For lots more useful information on upgrading C++ projects to Visual Studio 2010, refer to the "Project Upgrade Guide" post referenced above.

