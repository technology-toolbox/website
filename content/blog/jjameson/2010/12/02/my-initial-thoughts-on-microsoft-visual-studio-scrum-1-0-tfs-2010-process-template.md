---
title: My Initial Thoughts on Microsoft Visual Studio Scrum 1.0 (TFS 2010 Process Template)
date: 2010-12-02T11:26:00-07:00
description:
  "I've been using the new Scrum template for Team Foundation Server 2010 for a
  little over three weeks now -- not on a real project, admittedly, but rather
  on a sample project that I've been working on. [On the customer project that
  I've been working on..."
aliases:
  [
    "/blog/jjameson/archive/2010/12/01/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template.aspx",
    "/blog/jjameson/archive/2010/12/02/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template.aspx",
  ]
categories: ["Development"]
tags: ["TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/12/02/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template.aspx"
---

I've been using the
[new Scrum template](http://visualstudiogallery.msdn.microsoft.com/en-us/59ac03e3-df99-4776-be39-1917cbfc5d8e)
for Team Foundation Server 2010 for a little over three weeks now -- not on a
_real_ project, admittedly, but rather on a sample project that I've been
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

<div class="d-md-none">
  <a href='{{< relref "resources/table-1-popout" >}}' target="_blank">Table 1 - Initial Tasks for a Scrum Project</a>
  <i class="bi bi-arrow-up-right-square"></i>
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-md-block">
  {{< include-html "resources/table-1.html" >}}
</div>
