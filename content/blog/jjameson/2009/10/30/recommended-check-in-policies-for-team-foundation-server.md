---
title: "Recommended Check-In Policies for Team Foundation Server"
date: 2009-10-30T22:14:00-07:00
excerpt: "I love using Team Foundation Server (TFS). There's just an amazing amount of \"goodness\" for software development that comes out-of-the-box; and there's even more available from Microsoft and other sources in the form of add-ons (many of which are free..."
aliases: ["/blog/jjameson/archive/2009/10/30/recommended-check-in-policies-for-team-foundation-server.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/31/recommended-check-in-policies-for-team-foundation-server.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/31/recommended-check-in-policies-for-team-foundation-server.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

I love using Team Foundation Server (TFS). There's just an amazing amount of "goodness" for software development that comes out-of-the-box; and there's even more available from Microsoft and other sources in the form of add-ons (many of which are free).

From a source control perspective, one of my favorite features is check-in policies. Anything that improves the quality of software development with minimal effort is really a "[no-brainer](http://wordnetweb.princeton.edu/perl/webwn?s=no-brainer)" in my opinion.

The check-in policies that I prefer to have configured on all TFS projects are listed in the following table.

**Recommended TFS Check-In Policies**

| Policy Type | Description |
| --- | --- |
| Builds | This policy requires that any code changes have been compiled and the last build was successful. |
| Work Items | This policy requires that one or more work items be associated with every check-in. |
| Changeset Comments Policy | Reminds users to add meaningful comments to their check-ins. |
| Code Analysis | This policy requires that Code Analysis is run with a defined set of rules before check-in. |
| Testing Policy | Ensures that tests from specific test lists are successfully executed before checking in. |

The first two check-in policies are virtually painless and can be applied to any existing project at any time. These policies ship with Visual Studio Team System 2008, and assuming your developers have good software development discipline already, there's absolutely no reason not to enable these policies.

After all, why would any developer want to check-in code that hasn't been verified to at least compile? Similarly, why would a developer make a code change that isn't specifically related to a work item (even if that work item is simply something like "Refactor X" or "Code cleanup (M5)")?

However, the last three policies in the table above require a little more effort.

The Changeset Comments Policy is included in the [Team Foundation Server Power Tools](http://msdn.microsoft.com/en-us/teamsystem/bb980963.aspx), which means that every member of the Development team will need to download and install this in order to use it. (Note that if you don't have the Power Tools installed, you can still override the check-in policy in order to avoid being blocked.)

The Code Analysis policy can definitely cause a little heartburn for your Development team, depending on how many code analysis rules you enable and whether or not you treat these warnings as errors. I'll talk more about Code Analysis in a separate post.

The Testing Policy is not one that I consider essential for all projects. For projects in which you have a good set of Build Verification Tests (BVTs) that give you a signficant amount of code coverage and don't take very long to execute, then enabling this policy just makes sense. However, keep in mind that you should definitely be running BVTs as part of your automated build, so if these tests require a substantial amount of time to execute (for example, they require signficant setup or teardown), then forcing developers to run these tests before every check-in could definitely impede their productivity.

Of course, you can always separate your tests into "quick tests" and "long running tests" like I've mentioned in a previous [post](/blog/jjameson/2009/03/18/argumentnullexception-with-optional-publishingpage-description-property-with-some-thoughts-on-breaking-the-build-too), and consequently only require the "quick tests" to be executed before check-in.

If you are fortunate enough to be using TFS on your project, I hope you are reaping the benefits of check-in policies.

