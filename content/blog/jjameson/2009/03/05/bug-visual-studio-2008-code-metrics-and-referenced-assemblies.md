---
title: "Bug: Visual Studio 2008 Code Metrics and Referenced Assemblies"
date: 2009-03-05T10:04:00-07:00
excerpt:
  "Since I seem to be on a roll this morning with blogging, I figured I might as
  well get one more post in before moving on to my \"day job.\" During the
  process of authoring a different post earlier today, I stumbled across a bug
  while using the Code Metrics..."
aliases:
  [
    "/blog/jjameson/archive/2009/03/04/bug-visual-studio-2008-code-metrics-and-referenced-assemblies.aspx",
    "/blog/jjameson/archive/2009/03/05/bug-visual-studio-2008-code-metrics-and-referenced-assemblies.aspx",
  ]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "WSS v3", "Visual Studio"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/03/05/bug-visual-studio-2008-code-metrics-and-referenced-assemblies.aspx"
---

Since I seem to be on a roll this morning with blogging, I figured I might as
well get one more post in before moving on to my "day job."

During the process of authoring a different post earlier today, I stumbled
across a bug while using the Code Metrics feature in Visual Studio 2008.

After clicking the **Analyze** menu, and then clicking **Calculate Code Metrics
for Solution**, I encountered errors similar to the following for several
projects in the solution:

{{< blockquote "font-italic text-danger" >}}

An error occurred while calculating code metrics for target file
'E:\NotBackedUp\...' in project ... The following error was encountered while
reading module 'Microsoft.SharePoint': Could not resolve type: T ObjectModel.

{{< /blockquote >}}
(Don't ask me what the '' character is supposed to mean -- I just copied and pasted this directly from Visual Studio.)
It turns out that this is a known bug in Visual Studio 2008 due to the default
setting that allows dependant assemblies to be used in a project without
explicitly referencing them.

The workaround is to explicitly add the dependant assemblies to the project.

Consider the following projects from my solution:

**PublicationLibrary.Workspaces**

References:

- Microsoft.SharePoint
- Microsoft.SharePoint.Security
- System
- System.Data
- System.Web
- System.Xml

**PublicationLibrary.Workspaces.DeveloperTests** (i.e. the corresponding unit
tests project)

References:

- Microsoft.SharePoint
- PublicationLibrary.Workspaces
- System
- System.Data
- System.Web
- System.Xml

With these references, Visual Studio is able to successfully compile both
projects. However, it was only able to successfully calculate code metrics on
the **PublicationLibrary.Workspaces** project; attempting to calculate code
metrics on the second project yielded the aforementioned error.

After explicitly referencing **Microsoft.SharePoint.Security** in the
**PublicationLibrary.Workspaces.DeveloperTests** project, the error went away
and I could view the code metrics for both projects.
