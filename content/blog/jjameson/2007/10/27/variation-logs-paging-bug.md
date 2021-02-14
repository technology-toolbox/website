---
title: "Variation Logs Paging Bug"
date: 2007-10-27T02:01:00+08:00
excerpt: "It's embarrassing how my blog posts rapidly died off after this past June. However it's even more embarrassing to disclose the paging bug when viewing the Variation Logs page in Microsoft Office SharePoint Server (MOSS) 2007. 
 [By the way, my intent..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/10/27/variation-logs-paging-bug.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/10/27/variation-logs-paging-bug.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

It's embarrassing how my blog posts rapidly died off after this past June. However  it's even more embarrassing to disclose the paging bug when viewing the **Variation Logs** page in Microsoft Office SharePoint Server (MOSS) 2007.

[By the way, my intent is to jump back on the blogging bandwagon over the next  couple of weeks. I will share some important discoveries our team has made over  the last several months while building the second version of our solution based  on MOSS 2007 for a large enterprise customer.]

I take no credit for discovering the paging bug. Actually a fellow Microsoft  consultant, Jon Quist, pointed it out to me over a month ago while we were investigating  numerous issues with the MOSS variations feature.

To view the variation logs and observe the paging bug:

1. Browse to the home page of the top level site, click **Site Actions**,
   point to **Site Settings**, and then click **Modify All Site
   Settings**.
2. On the **Site Settings** page, in the **Site Collection
   Administration** section, click **Variation logs**.
3. On the **Variation Logs** page, observe that the results are
   sorted by **Time Started (GMT)** in descending order and that only
   the first 20 log entries are shown.
4. Click the right arrow in the paging control (i.e. "**1 - 20 &gt;**")
   to view the next page of results.
5. Notice that the results now appear to be sorted by **Time Started
   (GMT)** *ascending*. In other words, you have "jumped" from the
   viewing the newest log entries to viewing the oldest log entries.

This makes it impossible to effectively view recent items when you have more  than a few dozen log entries.

Ending this post here really wouldn't be helpful at all -- I'd simply be pointing  out a rather embarrassing bug that suggests the variations test cases need some  work. So what can you do before you get a Service Pack or QFE (i.e. hotfix) for  this bug?

The short answer is query the SharePoint database directly. [Remember it's okay  to query SharePoint databases all you want (realizing that this isn't technically  supported, because Microsoft may change the schema over time) -- just don't modify  your database directly unless you are running a script provided by PSS or you wouldn't  object to rebuilding your SharePoint environment if you encounter a problem after  modifying the database.]

The long answer is derived by using SQL Server Profiler to capture the SELECT  statement used to render the **Variation Logs** page. The query looks  something like this:

```
SELECT ...
FROM
    UserData
    ...
WHERE
    (UserData.tp_IsCurrent = 1)
    AND ...
    AND
    (
        (UserData.[nvarchar8] LIKE '%Variation%')
        AND ...
        AND (t1.DirName='Long Running Operation Status')
    )
ORDER BY
```

The first time you browse to the **Variation Logs** page, the ORDER  BY clause specifies to sort first by **tp\_Created** descending and  then by **tp\_ID** ascending. However, when you page to the next set  of results, the ORDER BY clause only specifies **tp\_ID** -- thus displaying  the oldest log entries first.

Note that the previous SELECT statement is substantially shorter than the actual  statement executed (large portions of the statement have been replaced by ellipses).  The key thing to note is that the log entries -- or, more generally, the status  messages for all long running operations -- are stored in the **AllUserData** table (**UserData** is a view on the **AllUserData** table). If you are not familiar with the **AllUserData** table,  it is almost always the largest table in the SharePoint content database -- in terms  of number of rows -- and contains all items in the various SharePoint lists. In  other words, **Long Running Operation Status** is just another SharePoint  list. Talk about eating your own dogfood!

Thus, to circumvent the paging bug in the **Variation Logs** page,  use a query similiar to the following:

```
SELECT
    UserData.[nvarchar1] AS 'Log Entry'
    ,UserData.[tp_Created] AS 'Time Started (GMT)'
    ,UserData.[tp_Modified] AS 'Time Last Updated (GMT)'
    ,UserData.[ntext5] AS 'Messages'
FROM
    UserData
WHERE
    tp_ListId = '{GUID}'
    AND nvarchar8 LIKE '%Variation%'
    AND tp_DirName = 'Long Running Operation Status'
```

where {GUID} is the unique identifier for the **Long Running Operation
Status** list, which can be obtained via the following query:

```
SELECT tp_ID
FROM AllLists
WHERE tp_Title = 'Long Running Operation Status'
```

Note that we can actually get a significantly better query plan if we drop the  superfluous **tp\_DirName** from the WHERE clause:

```
SELECT
    UserData.[nvarchar1] AS 'Log Entry'
    ,UserData.[tp_Created] AS 'Time Started (GMT)'
    ,UserData.[tp_Modified] AS 'Time Last Updated (GMT)'
    ,UserData.[ntext5] AS 'Messages'
FROM
    UserData
WHERE
    tp_ListId = '{GUID}'
    AND nvarchar8 LIKE '%Variation%'
```

I hope you find this first blog post in a long while to be helpful when working  with MOSS 2007.

