---
title: "Test Projects in Visual Studio 2010 Must Target .NET Framework 4"
date: 2010-04-28T04:01:00-06:00
excerpt:
  "Last week I installed Visual Studio 2010 on my primary desktop. This morning,
  I opened my Fabrikam.Demo solution in the new version of Visual Studio, but
  chose not to upgrade the target framework when prompted by Visual Studio
  during the solution upgrade..."
aliases:
  [
    "/blog/jjameson/archive/2010/04/27/test-projects-in-visual-studio-2010-must-target-net-framework-4.aspx",
    "/blog/jjameson/archive/2010/04/28/test-projects-in-visual-studio-2010-must-target-net-framework-4.aspx",
  ]
draft: true
categories: ["Development"]
tags: ["Core Development", "Visual Studio"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/28/test-projects-in-visual-studio-2010-must-target-net-framework-4.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/28/test-projects-in-visual-studio-2010-must-target-net-framework-4.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Last week I installed Visual Studio 2010 on my primary desktop. This morning, I
opened my Fabrikam.Demo solution in the new version of Visual Studio, but chose
not to upgrade the target framework when prompted by Visual Studio during the
solution upgrade. In other words, I chose to leave the **Target framework**
project setting set to **.NET Framework 3.5** -- at least for now.

There's no doubt I'll eventually upgrade my demo solution to .NET Framework 4,
but in the short term, I imagine that I'll still be working primarily with the
older version of the .NET Framework, especially since I don't believe SharePoint
is going to support .NET Framework 4 anytime soon.

After viewing the conversion log and verifying that no errors or warnings
occurred during the upgrade process, I then proceeded to rebuild the solution.

I was surprised to see that the build failed.

My first thought was "that's weird, the solution compiled without any warnings
or errors last week when I built it using Visual Studio 2008."

Then I looked more closely at the error:

{{< blockquote "font-italic text-danger" >}}

The type 'System.Web.Security.MembershipProvider' is defined in an assembly that
is not referenced. You must add a reference to assembly
'System.Web.ApplicationServices, Version=4.0.0.0, Culture=neutral,
PublicKeyToken=31bf3856ad364e35'.

{{< /blockquote >}}

Hmmm...that is strange indeed...didn't I just tell Visual Studio not to target
.NET Framework 4?

I then looked at the settings for the project that failed to build
(Security.DeveloperTests) and confirmed that, yes, the **Target framework**
setting had indeed been changed to **.NET Framework 4**. Hmmm...must have just
been a glitch, I'll just change the setting back to **.NET Framework 3.5**...

Unfortunately, that's when I got the following warning:

{{< blockquote "font-italic" >}}

Attempted re-targeting of the project has been canceled. You cannot change the
specified .NET framework version or profile for a test project.

{{< /blockquote >}}

After clicking the **OK** button, I noticed the project setting -- not
suprisingly -- was changed back to **.NET Framework 4**.

After adding the reference in the Security.DeveloperTests project to the new
System.Web.ApplicationServices assembly, my solution compiled without error (and
all of the unit tests passed).

While I don't anticipate there to be any issues with this project setting for
test projects, I would feel more comfortable if you could truly target an older
version of the .NET Framework using Visual Studio 2010 (meaning, across all
project types that you could create in Visual Studio 2008).
