---
title: My Initial Thoughts on Microsoft Visual Studio Scrum 1.0 (TFS 2010 Process Template)
date: 2010-12-02T11:26:00-07:00
excerpt:
  "I've been using the new Scrum template for Team Foundation Server 2010 for a
  little over three weeks now -- not on a real project, admittedly, but rather
  on a sample project that I've been working on. [On the customer project that
  I've been working on..."
aliases:
  [
    "/blog/jjameson/archive/2010/12/01/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template.aspx",
    "/blog/jjameson/archive/2010/12/02/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template.aspx",
  ]
draft: true
categories: ["Development"]
tags: ["TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/12/02/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/12/02/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/12/02/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

I've been using the
[new Scrum template](http://visualstudiogallery.msdn.microsoft.com/en-us/59ac03e3-df99-4776-be39-1917cbfc5d8e)
for Team Foundation Server 2010 for a little over three weeks now -- not on a
*real* project, admittedly, but rather on a sample project that I've been
working on. [On the customer project that I've been working on for over a year
now, we're "stuck" on TFS 2008 until the extranet TFS instance hosted by
Microsoft is upgraded to the new version. Of course, even after TFS is upgraded,
it's unlikely we would create a new project just to use the new process
template, but anyway...]

One of the first things that I noticed about the Scrum template is the fact that
the "Description" field isn't restricted to plain text -- unlike the Description
field in the MSF for Agile Software Development v5.0 process template. As the
old saying goes, sometimes it's the little things...

Honestly, I was rather disappointed to find that after upgrading to TFS 2010,
the new MSF Agile process template does not allow HTML/formatting in the
Description field. I recall reading on someone's blog how this was something the
Visual Studio folks debated at great length, but eventually punted and kept the
field as plain text. Frankly, I've been known to put content into the History
field when using TFS 2008 (even though I would have preferred to put it in the
Description field) just so I could incorporate some important formatting. [I
often copy/paste back and forth between work items and email, and having to
reformat content (for example, repro steps) is not something I really enjoy
spending time on. It was a huge relief to see the new Repro Steps field in TFS
2010 that allows HTML formatting.]

Of course, enabling HTML/formatting in the Description field doesn't come
without penalty -- meaning you won't be able to specify the **Description HTML**
field when adding or updating work items using Microsoft Excel. My guess is that
the integration with Microsoft Excel and Microsoft Project (which only support
plain text fields) eventually trumped the desire to change the field in the MSF
Agile template to allow HTML formatting. [Personally, I would have preferred it
if the **Description HTML** field in Excel wasn't marked read-only and instead
showed the actual HTML markup. It's not like I expect Excel and Project to
provide a full-blown WYSIWYG experience for an HTML field (I am a reasonable
person, after all). However, there's probably an equal (or greater) number of
people out there who would balk at the presence of HTML tags when viewing work
items in Excel. Oh well, it is what it is (at least for now).]

The second thing that I noticed about the Scrum template is that it creates 24
work items by default, but these are all of type Sprint (four releases with six
sprints each). Personally, I liked the older templates (from TFS 2005 and 2008)
where the default work items actually provided some meaningful guidance for
those who are new to TFS. However, I suppose that most people working with TFS
these days probably keep a spreadsheet with the default work items they like to
start out with anyway, and since creating work items from Excel is so easy that
it takes almost no effort whatsoever, this really isn't an issue (although not
being able to import the **Description HTML** field does make this rather
tedious with a Scrum project).

Since I'd much rather use iterations named something like "v1.0\Sprint 1" rather
than the default "Release 1\Sprint 1", I had to update the out-of-the-box work
item queries after renaming the iterations in order to get them working again.

The next thing I discovered about the Scrum template is that it doesn't include
any Excel workbooks. If you've used the Excel workbooks from the MSF Agile v5.0
template (e.g. Product Planning and Iteration Backlog) then, like me, you're
probably going to be disappointed to find there are none included in the Scrum
template. You can certainly create some similar workborks on your own, but I
think it would take considerable work to give you equivalent functionality.

Lastly, I found that while the reports included with the Scrum template include
the most popular (e.g. Sprint Burndown, Release Burndown, and Velocity), the
template doesn't include a number of reports that you get with the MSF Agile
template that are still very much applicable to Scrum (e.g. Planned vs.
Unplanned Work).

While I certainly like many aspects of the new Scrum template, the important
question is: Would I recommend it (over MSF Agile v5.0) to customers when
starting a new project (or when starting work on a new version)? Honestly, I'm
not very particular when it comes to what you call something -- as long as
everyone on the team uses consistent terminology. If you can get everyone to
agree to use "iterations" rather than "sprints", and "user stories" instead of
"product backlog items", then -- at least in my mind -- there's no reason you
can't follow the Scrum principles while using the MSF Agile template (which
includes much more out-of-the-box "goodness" than the Scrum template).

If, however, you'd rather forego the MSF Agile workbooks, Excel reports, and
dashoards in order to align the project terminology with Scrum, then by all
means, go right ahead.

In case you are wondering what tasks I recommend starting out with on a new
Scrum project, I'll include them here. Note that many of these tasks are from
the original MSF Agile template and don't take more than a few minutes to
complete. However, I still like to include them to serve as a "checklist" on new
projects.

{{< table class="small" caption="Initial Tasks for a Scrum Project" >}}

| Title | Description |
| --- | --- |
| Setup: Configure permissions for TFS project | Add team members to one of the four security groups: Builders, Contributors, Project Administrators, or Readers. To configure security, in the **Team Explorer** window, right-click the team project, point to **Team Project Settings** and then click **Group Membership**. |
| Setup: Configure permissions on team project portal | Add team members to one of the five permission levels: Full Control, Design, Contribute, Read, or View Only. To configure security on the SharePoint team site, open the project portal, click **Site Actions**, and then click **Site Permissions**. |
| Setup: Create project structure | Create the project structure that captures what areas the development team will be working in. To set project structure, in the **Team Explorer** window, right-click the team project, point to **Team Project Settings** and then click **Areas and Iterations**. |
| Setup: Migrate work items | If you are bringing an existing project into VSTS, migrate work items such as bugs and tasks from the previous work item repository. You should complete migration of work items before team members are granted access to the team project. |
| Setup: Migrate source code | If you are bringing an existing project into VSTS, migrate the source code from the previous source code repository. You should complete migration of source code before team members are granted access to the team project. |
| Setup: Configure check-in policies | Setup the business rules or policies that govern source code check-ins. For more information, see **Check-in Policies and Notes** in the Visual Studio help. |
| Setup: Create initial source tree and Visual Studio solution | Create the initial source tree, including the Main branch and additional Dev branches (as necessary). |
| Setup: Add custom dictionary to Visual Studio solution | For more information on this task, refer to the following blog post:<br><br>**CA1704 Code Analysis Warning and Using Custom Dictionaries in Visual Studio**<br>[http://blogs.msdn.com/b/jjameson/archive/2009/04/02/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio.aspx](/blog/jjameson/2009/04/02/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio) |
| Setup: Generate strong name key and configure assembly signing | Use the strong name utility (sn.exe) to create a new strong name key and configure all projects in the solution to sign their respective assemblies using this key. |
| Setup: Add "SharedAssemblyInfo" and "AssemblyVersionInfo" files to Visual Studio solution | For more information on this task, refer to the following blog post:<br><br>**Shared Assembly Info in Visual Studio Projects**<br>[http://blogs.msdn.com/b/jjameson/archive/2009/04/03/shared-assembly-info-in-visual-studio-projects.aspx](/blog/jjameson/2009/04/03/shared-assembly-info-in-visual-studio-projects) |
| Setup: Configure automated build | Configure an automated build to run on a periodic basis (typically daily). For more information, see **Administering Team Foundation Build** in the Visual Studio help. |
| Setup: Send email to users with TFS project details | Send an e-mail to team members that provides information about which Team Foundation Server they should connect to, and which team project they should use so they can get started working on the team project. |
| Brainstorm and prioritize Product Backlog | The product backlog is a high-level list that is maintained throughout the entire project. It aggregates backlog items: broad descriptions of all potential features, prioritized as an absolute ordering by business value. It is therefore the "What" that will be built, sorted by importance. [[http://en.wikipedia.org/wiki/Scrum\_(development)](http://en.wikipedia.org/wiki/Scrum_%28development%29)] |
| Define the Sprint Backlog (v1.0\Sprint 1) | In the "Sprint Planning Meeting" with the entire team, select what work is to be done in the sprint and prepare the Sprint Backlog that details the time it will take to do the work. |

{{< /table >}}
