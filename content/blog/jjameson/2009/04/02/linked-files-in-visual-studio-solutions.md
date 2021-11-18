---
title: Linked Files in Visual Studio Solutions
date: 2009-04-02T08:20:00-06:00
description:
  'A couple of years ago, I wrote a post introducing my system for structuring
  Visual Studio solutions . However, I apparently forgot to post a follow-up
  providing additional details, such as configuring assembly versioning and what
  I like to call "shared...'
aliases:
  [
    "/blog/jjameson/archive/2009/04/01/linked-files-in-visual-studio-solutions.aspx",
    "/blog/jjameson/archive/2009/04/02/linked-files-in-visual-studio-solutions.aspx",
  ]
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "Visual Studio", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/04/02/linked-files-in-visual-studio-solutions.aspx"
---

A couple of years ago, I wrote a post introducing my system for
[structuring Visual Studio solutions](/blog/jjameson/2007/04/18/structure-visual-studio-solutions).
However, I apparently forgot to post a follow-up providing additional details,
such as configuring assembly versioning and what I like to call "shared assembly
information."

Before I can cover these details, I need to first ensure that you are familiar
with the concept of linking files in Visual Studio solutions, why this is a
powerful feature, and when to use it.

If you ever used Visual SourceSafe (VSS), you likely used its feature for
sharing files across multiple projects. For example, you could check-in a file
at the root of your solution, and then drag-and-drop it into other projects
(i.e. "subfolders") in your solution. Thus, whenever a change was made to the
file (regardless of which particular VSS project the change was made in), the
next time you "got latest" the change would be reflected in all locations. This
was a common way of, for example, having all of your .NET assemblies in the same
Visual Studio solution specify the same assembly version.

Note that Team Foundation Server (TFS) does not provide an equivalent "share
file" feature. Fortunately, however, you no longer need such a feature.

Back in the days of the original Visual Studio .NET and the following version,
Visual Studio .NET 2003, whenever you added an existing file to a project, it
copied the file into the corresponding location in the project.

However, in Visual Studio 2005, the **Add Existing Item** feature provided the
ability to choose to either **Add** the item or **Add As Link** (via the little
down arrow on the button in the dialog box).

In other words, once you upgraded to Visual Studio 2005, it was no longer
necessary to rely on any "sharing" features of your source control system in
order to have multiple projects always reference the latest version of a file.

Note that when you **Add** an item in Visual Studio 2005 or Visual Studio 2008,
the behavior is the same as earlier versions (meaning the file is copied into
the corresponding location within the project). When you choose to **Add As
Link**, however, you simply reference the file in-place. [Don't worry, relative
paths ensure that everyone on your team is free to choose whatever root folder
they wish for their individual workspaces.]

To illustrate this concept, I quickly built out a "demo" solution, as shown
below.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Linked-Files-in-Visual-Studio-Solutions-372x577.JPG"
alt="Linked files in a Visual Studio solution" class="screenshot" height="577"
width="372" caption="Figure 1: Linked files in a Visual Studio solution" >}}

The corresponding folder structure on disk resembles the following:

- Fabrikam

  - Demo

    - Dev _(branch)_

      - Lab1 _(branch)_

        - AssemblyVersionInfo.cs
        - CustomDictionary.xml
        - Fabrikam.Demo.sln
        - Fabrikam.Demo.snk
        - SharedAssemblyInfo.cs

        - AdminConsole
          - AdminConsole.csproj
          - Program.cs
          - Properties
            - AssemblyInfo.cs
        - CoreServices
          - CoreServices.csproj
          - Logging
            - Logger.cs
          - Properties
            - AssemblyInfo.cs

Note that **AssemblyVersionInfo.cs**, **CustomDictionary.xml**,
**Fabrikam.Demo.snk**, and **SharedAssemblyInfo.cs** reside in the same folder
as the Visual Studio solution file and are subsequently "linked into" the two
Visual C# projects. Thus whenever a change is made to one of these files, the
next build of each project will reflect that change.

With this foundation in place, I'll explain some other recommended best
practices over a series of follow-up posts, including:

- [Using custom dictionaries in Visual Studio](/blog/jjameson/2009/04/02/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio)
- [Shared assembly info in Visual Studio projects](/blog/jjameson/2009/04/03/shared-assembly-info-in-visual-studio-projects)
- [Best practices for .NET assembly versioning](/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning)

Stay tuned!
