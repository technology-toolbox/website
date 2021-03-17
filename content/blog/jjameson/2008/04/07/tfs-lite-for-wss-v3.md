---
title: "\"TFS Lite\" for WSS v3"
date: 2008-04-07T11:08:00-06:00
excerpt:
  "In my previous post , I introduced my \"TFS Lite\" SharePoint site template
  that I've been using for years as a simple scenario/task/bug/risk/milestone
  tracking \"application\" with various projects and customers. In today's post,
  I'll discuss some of the..."
aliases:
  [
    "/blog/jjameson/archive/2008/04/06/tfs-lite-for-wss-v3.aspx",
    "/blog/jjameson/archive/2008/04/07/tfs-lite-for-wss-v3.aspx",
  ]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Core Development", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/04/07/tfs-lite-for-wss-v3.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/04/07/tfs-lite-for-wss-v3.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In my [previous post](/blog/jjameson/2008/04/01/tfs-lite-for-wss-v2), I
introduced my "TFS Lite" SharePoint site template that I've been using for years
as a simple scenario/task/bug/risk/milestone tracking "application" with various
projects and customers. In today's post, I'll discuss some of the important
changes in Windows SharePoint Services (WSS) v3 that impacted TFS Lite, such as
the fact that *all* SharePoint lists now support versioning, not just the
**Issues** list, as was the case in WSS v2 and SharePoint Portal Server (SPS)
2003.

As I noted in the earlier post, the Work Items list at the heart of TFS Lite was
originally based on the Issues list in order to maintain a history as items are
modified. When you are dealing with sensitive items like bugs and risks, a
simple snapshot of the latest item simply won't do. You need traceability to
determine who said what and when. Granted, there are times when I wish I could
"undo" or "rollback" a change to a list item -- for example, when I enter
comments on a bug like "Fixed in 3.0.220.0" but neglect to change the **Status**
to **Resolved**. However, dealing with goofs like that just comes with the
territory when you need a detailed audit history of every change to an item.

When working with the previous versions of WSS and SPS, the various views used
to render snapshots of the Work Items list specified **Current is equal to Yes**
in order to show only the latest version of each work item. Since any list in
WSS v3 can now inherently support versioning, it is no longer necessary to base
the Work Items list on the Issues list, nor is it necessary to include the
**Current is equal to Yes** criteria in each view.

The other significant change in WSS v3 that impacted TFS Lite is the way related
items are entered. In WSS v2 and SPS 2003, the Issues list allowed you to simply
type in the ID for a related issue to associate two or more items together.
Furthermore, in WSS v2 when you added, for example, issue 1 as a related item to
issue 2, then when you viewed issue 1, you would see issue 2 as a related item,
and vice-versa. The net result in WSS v2 was that if you related issue 1 to
issue 2 and then subsequently related issue 2 to issue 3, then issues 1, 2, and
3 were all interrelated. This is not the case in WSS v3.

In WSS v3, however, when you add issue 1 as a related item to issue 2, the
related item only shows up on issue 2 (in other words, issue 1 does not show
that it is related to issue 2). Also note that the user interface changed from a
simple textbox in WSS v2 (in which you simply type the ID of the related issue)
to a pair of lists in WSS v3 (in which you select one or more items in the list
on the left and then click **Add &gt;** to associate them to the current item).
These changes can be considered good or bad -- depending on your perspective.

If you like the fact that you can now associate multiple issues at a single time
(and also view the title of each item when selecting related issues), then this
is great. However, if you liked the "bidirectional association" in WSS v2 or if
you have many items in your list (say, around 2,500) then:

1. The user experience of having to select from a list containing thousands of
   items leaves much to be desired.
2. There is definitely a performance impact when you create or edit an item
   (since SharePoint has to fetch all items in the list to allow you to possibly
   associate any one of them to the current item).

With these WSS v3 changes in mind, let's "upgrade" the TFS Lite site template
accordingly...

Here is a step-by-step guide for creating the TFS Lite site template for WSS v3:

1. Create a new site collection using the **Team Site** template.

2. Optionally delete the **Tasks** list that is automatically created as part of
   the Team Site template (since a "task" is simply a work item where Category =
   "Task").

3. Create a new list called **WorkItems** based on the **Tasks** list and select
   **Yes** for the **Send e-mail when ownership is assigned?** option.

4. Rename the **WorkItems** list to **Work Items**. (I prefer to avoid spaces
   when creating lists to avoid "garbage" in the URLs as a result of URL
   encoding.)

5. Enable versioning on the **Work Items** list to create a version each time
   someone edits an item in this list.

6. Add the columns specified in the following table:
   
   {{< table class="small" >}}
   
   | Column Name | Description | Type | Required | Additional Information | Default | Add to Default View |
   | --- | --- | --- | --- | --- | --- | --- |
   | Category |  | Choice | Yes | Choices:<ul><li>Bug</li><li>Quality of Service Requirement</li><li>Risk</li><li>Scenario</li><li>Task</li></ul> | Bug | Yes |
   | Area |  | Choice | Yes | Choices:<ul><li>Project</li><li>Project\Area1</li><li>Project\Area2</li><li>Project\Area3</li></ul> |  | Yes |
   | Iteration |  | Choice | Yes | Choices:<ul><li>v1.0</li><li>v1.0\M0</li><li>v1.0\M1</li><li>v1.0\M2</li><li>v1.0\RC1</li><li>v1.0\RC2</li><li>v1.0\RTM</li><li>v1.1</li><li>v2.0</li><li>v2.0\M0</li><li>v2.0\M1</li><li>v2.0\M2</li><li>v2.0\RC1</li><li>v2.0\RC2</li><li>v2.0\RTM</li><li>vNext</li></ul> | v1.0 <sup>*</sup> | Yes |
   | Triage |  | Choice | Yes | Choices:<ul><li>1 - Needs Review</li><li>2 - Investigate</li><li>3 - Reviewed</li><li>4 - Recommend Approve</li><li>5 - Recommend Reject</li><li>6 - Approved</li><li>7 - Not Approved</li></ul> | 1 - Needs Review | Yes |
   | Priority <sup>**</sup> |  | Choice | Yes | Choices:<ul><li>1 - High</li><li>2 - Normal</li><li>3 - Low</li></ul> | 2 - Normal | Yes |
   | Severity |  | Choice | Yes | Choices:<ul><li>1 - Critical</li><li>2 - Important</li><li>3 - Nuisance</li><li>4 - Useful</li></ul> | 4 - Useful | Yes |
   | Status <sup>**</sup> |  | Choice | Yes | Choices:<ul><li>Active</li><li>Resolved</li><li>Closed</li></ul> | Active | Yes |
   | Comments |  | Multiple lines of text | Yes | Append Changes to Existing Text: Yes |  | No |
   | Exit Criteria | Indicates if the work item is a key deliverable for the iteration or milestone. If the field is set to Yes, the work item displays in the project summary view. | Yes/No (check box) | No |  | No | No |
   | Blocked | Indicates if the work item is blocked in some manner. If the field is set to Yes, the work item displays on the project summary view. | Yes/No (check box) | No |  |  | Yes |
   | KPI | Key Performance Indicator representing the status of a key deliverable or milestone. | Choice | Yes | Choices:<ul><li>Green</li><li>Yellow</li><li>Red</li><li>Complete</li></ul> |  | No |
   | CreatedFilter | Calculated field used to filter recently created items (for example, to show items submitted during the last week). | Calculated (calculation based on other columns) |  | Formula:<br>=Created+7<br><br>The data type returned from this formula is: Date and Time<br><br>Date and Time Format: Date Only |  | No |
   | ModifiedFilter | Calculated field used to filter recently modified items (for example, to show accomplishments over the last week). | Calculated (calculation based on other columns) |  | Formula:<br>=Modified+7<br><br>The data type returned from this formula is: Date and Time<br><br>Date and Time Format: Date Only |  | No |
   
   {{< /table >}}
   
   <small>* You should actively change the default value for this column based
   on the target iteration for completing work items.</small>\
   <small>** Modify the out-of-the-box column accordingly.</small>

7. Configure the following views:
   
   {{< table class="small" >}}
   
   | View | Columns | Sort | Filter | Group By |
   | --- | --- | --- | --- | --- |
   | All Work Items | <ul><li>ID</li><li>Title</li><li>Category</li><li>Status</li><li>Priority</li><li>Severity</li><li>Area</li><li>Blocked</li><li>Assigned To</li></ul> | ID |  |  |
   | My Work Items<br>(default view) | <ul><li>ID</li><li>Title</li><li>Category</li><li>Status</li><li>Priority</li><li>Severity</li><li>Area</li><li>Blocked</li></ul> | Severity | Status is not equal to Closed<br>And<br>Assigned To is equal to [Me] | Priority |
   | Active Bugs | <ul><li>ID</li><li>Title</li><li>Priority</li><li>Severity</li><li>Area</li><li>Assigned To</li></ul> | Severity | Status is equal to Active<br>And<br>Category is equal to Bug | Priority |
   | Resolved Bugs | <ul><li>ID</li><li>Title</li><li>Priority</li><li>Severity</li><li>Area</li><li>Assigned To</li></ul> | Severity | Status is equal to Resolved<br>And<br>Category is equal to Bug | Priority |
   | Open Issues | <ul><li>ID</li><li>Title</li><li>Category</li><li>Status</li><li>Priority</li><li>Severity</li><li>Area</li><li>Assigned To</li></ul> | Severity | Status is not equal to Closed<br>And<br>Blocked is equal to Yes | <ol><li>Category</li><li>Priority</li></ol> |
   | Open Scenarios | <ul><li>ID</li><li>Title</li><li>Category</li><li>Status</li><li>Priority</li><li>Severity</li><li>Area</li><li>Assigned To</li></ul> | Severity | Status is not equal to Closed<br>And<br>Category is equal to Scenario | Priority |
   | Open Tasks | <ul><li>ID</li><li>Title</li><li>Category</li><li>Status</li><li>Priority</li><li>Severity</li><li>Area</li><li>Assigned To</li></ul> | Severity | Status is not equal to Closed<br>And<br>Category is equal to Task | Priority |
   | Open Work Items | <ul><li>ID</li><li>Title</li><li>Category</li><li>Status</li><li>Priority</li><li>Severity</li><li>Area</li><li>Blocked</li><li>Assigned To</li></ul> | Severity | Status is not equal to Closed | <ol><li>Category</li><li>Priority</li></ol> |
   | Project Checklist | <ul><li>ID</li><li>Title</li><li>Category</li><li>Status</li><li>Priority</li><li>Severity</li><li>Area</li><li>Assigned To</li><li>KPI</li></ul> | <ol><li>Priority</li><li>Severity</li></ol> | Status is not equal to Closed<br>And<br>Exit Criteria is equal to Yes | <ol><li>Iteration</li><li>Category</li></ol> |
   
   {{< /table >}}

8. Create a new document library named **Pages** and select **Web Part page** as
   the document template.

9. In the **Pages** library, create a new page called **ProjectSummary.aspx**
   using the **Header, Footer, 3 Columns** layout.

10. Create and configure the various project summary Web Parts based on the underlying Work Items list:
    
    {{< table class="small" >}}
    
    | Web Part | Columns | Sort | Filter | Group By | Item Limit |
    | --- | --- | --- | --- | --- | --- |
    | Project Summary | <ul><li>KPI</li><li>Title</li></ul> | ID | Iteration is equal to v1.0\M0 <sup>*</sup> |  | 100 |
    | Top 10 Issues | <ul><li>Title</li><li>Assigned To</li></ul> | <ol><li>Priority</li><li>Severity</li></ol> | Blocked is equal to Yes |  | 10 |
    | Accomplishments (Last 7 Days) | Title | Modified<br>(descending) | Status is equal to Closed<br>And<br>ModifiedFilter is greater than [Today] | Category<br>(Expanded) | 20<br>(Display items in batches of the specified size.) |
    | Priorities/Milestones | Title | ID | Status is not equal to Closed<br>And<br>Exit Criteria is equal to Yes | Iteration<br>(Expanded) | 100<br>(Display items in batches of the specified size.) |
    
    {{< /table >}}
    
    <small>* Change the filter as necessary to match the current
    iteration.</small>

11. (Optional) Modify the **Project Summary** Web Part to display an image
    corresponding to the designated KPI value (using SharePoint Designer and a
    tiny bit of XSLT as described in my
    [previous post](/blog/jjameson/2008/04/01/tfs-lite-for-wss-v2)).

12. In the **Links** list, add a link to the **Project Summary** page.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/TFS-Lite-WSS-v3-600x363.jpg"
alt="Project Summary \"dashboard\"" class="screenshot" height="363" width="600"
title="Figure 1: Project Summary \"dashboard\"" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/TFS-Lite-WSS-v3-991x599.jpg)

You might be wondering why I chose to base the Work Items list on Tasks instead
of just creating a custom list from scratch. Aside from the fact that the Tasks
list saves me from having to create a few columns, the real reason is to
automatically e-mail team members when a work item is assigned to them. If you
simply create a custom list and add an **Assigned To** column, you'll find that
the option to **Send e-mail when ownership is assigned?** is conspicuously
absent from the **Advanced Settings** page for the list.

You certainly don't have to implement the list this way. Alternatively, you
could choose to use alerts instead and have people "subscribe" to changes in the
**My Work Items** view. However, I chose a "push" model instead of a "pull"
model to ensure people are always alerted right away when they are responsible
for working on something.

You also might be wondering how work items can be related to each other since I
chose not to base the new Work Items list on the Issues list. Given the change
in behavior in the WSS v3 Issues list for related items, I recommend simply
adding a section to the **Description** field name named **Related Items** and
then insert hyperlinks to other items. The primary reason for this is to avoid
the performance impact as the number of work items grows large.

> **Important**
>
> There appears to be a bug in WSS v3 (and MOSS 2007) where the Web Parts on the
> Project Summary page are not configured with the correct views after creating
> a new site from the attached site template. Consequently you will need to
> spend a few minutes reconfiguring the Web Parts using the settings specified
> in step 10 above. The Web Parts are created, but the columns, sort, filter,
> and group by settings revert to the default values. Fortunately it only takes
> a few minutes to workaround this bug.

> **Update (2008-04-08)**
>
> The issue noted below by Dragan has been corrected. The fix is described in
> [a subsequent post](/blog/jjameson/2008/04/08/creating-a-site-template-in-moss-2007-that-works-in-wss-v3).
> However, as I originally suspected might be the case, the KPI images that I
> use in the project dashboard view are only available in MOSS 2007.
> Consequently you will need to substitute your own images (or simply revert to
> displaying the status as text).
