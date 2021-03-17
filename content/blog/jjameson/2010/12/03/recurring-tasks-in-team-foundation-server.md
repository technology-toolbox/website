---
title: "Recurring Tasks in Team Foundation Server"
date: 2010-12-03T13:03:00-07:00
excerpt:
  "While the vast majority of work items created for each iteration (sprint) are
  unique and therefore require some planning effort beforehand, I've gotten into
  the habit of creating a few recurring tasks in TFS each time I start a new
  iteration on a project..."
aliases: ["/blog/jjameson/archive/2010/12/02/recurring-tasks-in-team-foundation-server.aspx", "/blog/jjameson/archive/2010/12/03/recurring-tasks-in-team-foundation-server.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/12/03/recurring-tasks-in-team-foundation-server.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/12/03/recurring-tasks-in-team-foundation-server.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

While the vast majority of work items created for each iteration (sprint) are
unique and therefore require some planning effort beforehand, I've gotten into
the habit of creating a few recurring tasks in TFS each time I start a new
iteration on a project.

For example, at the start of the most recent sprint on my current project, I
added the following tasks:

- Code cleanup - Sprint-10
- Update Installation Guide for Sprint-10
- Create branch for Sprint-10 release

There's nothing special about these tasks (in other words, I typically add the
work items using Microsoft Excel just like the other work items for a particular
sprint). For example, the first task simply provides me a work item to associate
with "cleanup" check-ins (which are typically incidental changes, such as fixing
a comment or refactoring a lengthy method). The other tasks remind me of things
I need to ensure get done for each and every release (e.g. updating the Install
Guide and branching the code for release).

In
[my next post](/blog/jjameson/2010/12/03/branching-for-a-release-in-team-foundation-server),
I'll talk more about the process I use to branch code for a particular release.
