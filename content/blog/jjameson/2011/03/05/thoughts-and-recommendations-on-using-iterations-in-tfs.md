---
title: Thoughts and Recommendations on Using Iterations in TFS
date: 2011-03-05T06:34:00-07:00
description:
  Have you ever lost work items in Team Foundation Server? I know I have. Well,
  let me clarify that...it's not that I actually lost work items due to some bug
  in TFS or failure on the database server. Rather -- and I'm a little
  embarrassed to admit this...
aliases:
  [
    "/blog/jjameson/archive/2011/03/04/thoughts-and-recommendations-on-using-iterations-in-tfs.aspx",
    "/blog/jjameson/archive/2011/03/05/thoughts-and-recommendations-on-using-iterations-in-tfs.aspx",
  ]
categories: ["My System", "Development"]
tags: ["My System", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/03/05/thoughts-and-recommendations-on-using-iterations-in-tfs.aspx"
---

Have you ever lost work items in Team Foundation Server? I know I have.

Well, let me clarify that...it's not that I actually _lost_ work items due to
some bug in TFS or failure on the database server. Rather -- and I'm a little
embarrassed to admit this -- it's just that I somehow _misplaced_ them (which
also occasionally happens with other items as well, such as my wallet). In other
words, I can't seem to quickly find one or more work items that I'm certain I
created previously in a particular TFS project.

The problem -- at least in my case -- is due to the out-of-the-box queries that
you get with the **MSF for Agile Software Development v5.0** process template in
TFS 2010.

To understand why work items occasionally go
[AWOL](http://en.wikipedia.org/wiki/Desertion), let's quickly review the default
queries that are created for a "MSF Agile v5" project:

- Team Queries
  - Iteration 1
    - Active Bugs
    - Active Tasks
    - Bug Triage
    - Completed Tasks
    - Iteration Backlog
    - Open Issues
    - Open Test Cases
    - Open User Stories
    - Resolved Bugs
    - User Stories Delivered
    - User Stories without Test Cases
  - Iteration 2
    - Iteration Backlog
  - Iteration 3
    - Iteration Backlog
  - Troubleshooting
    - Work Items With Summary Values
  - My Bugs
  - My Tasks
  - My Test Cases
  - Product Backlog
  - Products Planning

I'm assuming you're familiar with most, if not all, of these queries. Even if
you're not, it should be fairly obvious that all of the queries under
**Iteration 1** include a filter like:

- **Team Project = @Project**
- **And Iteration Path Under foobar2010\Iteration 1**
- ...

[Assume "foobar2010" is the name of the TFS project.]

Likewise, all of the "My" queries (e.g. **My Bugs**) include a filter like:

- **Team Project = @Project**
- **And Assigned To = @Me**
- ...

Note that the **Product Backlog** and **Product Planning** queries include a
filter like:

- **Team Project = @Project**
- **And Area Path Under @Project**
- **And Work Item Type = User Story**
- ...

Now, with that rather lengthy introduction out of the way, imagine that we use
Microsoft Excel to quickly add a bunch of tasks (or bugs). However, we forget
(or neglect) to set the **Assigned To** and **Iteration Path** properties. What
happens?

Yep, you guessed it...none of those new work items will show up in any of the
out-of-the-box queries.

"Doh!" [Imagine Homer Simpson saying that...it's funnier.]

Now, the truth is this really isn't the fault of TFS -- after all, we were the
ones who didn't set the Iteration Path correctly, thereby ensuring the bugs
and/or tasks appeared as expected in the results of the corresponding "iteration
query" (e.g. **Active Bugs** or **Active Tasks**).

To understand how to avoid this situation, first note that I like to configure
the iterations for a TFS project similar to the following:

- Iteration
  - v1.0
    - M1
    - M2
    - M3
  - v1.1
  - vNext

...or, sometimes, I'll call them "sprints" instead of "milestones" (e.g.
"Sprint-1" rather than "M1"):

- Iteration
  - v1.0
    - Sprint-1
    - Sprint-2
    - Sprint-3
  - v1.1
  - vNext

What you call them really doesn't matter to me. That's just a name. What _is_
important, however, is that any work item that isn't assigned to a "real"
iteration is considered to be invalid (because that means it's essentially
fallen off the radar).

[Yes, customizing the iterations like I've shown above does require a little
extra work to update references to the renamed iterations, but honestly it
doesn't take more than a few minutes.]

To avoid mistakenly "losing" work items, I like to create a new query under the
**Troubleshooting** folder, called **Work Items with Invalid Iteration** and set
the filter to something like:

- **Team Project = @Project**
- **And (**
  - **And Iteration Path = foobar2010**
  - **Or Iteration Path = foobar2010\v1.0**
- **)**

In case it's not immediately obvious, the "And (...)" in the filter above
reflects a **Group Clause** in TFS.

[It's a good idea to periodically check the queries in the **Troubleshooting**
folder to ensure they return zero results.]

Note that **v1.1** and **vNext** are intended to be used as "buckets" for future
functionality (in other words, whenever we identify something that we want to
track, but we don't yet know which iteration it will actually be worked on). At
the point in time where we actually release v1.0 of the solution (and start
working on v1.1 -- and perhaps even v2.0 in parallel), then we would need to add
new iterations and update the **Work Items with Invalid Iteration** query
accordingly.
