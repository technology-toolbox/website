---
title: "Dumping MOSS 2007 Variations - Part 2"
date: 2007-10-31T07:56:00-06:00
excerpt: "In part 1 of this series, I talked about my current customer's decision to abandon the use of the variations feature in Microsoft Office SharePoint Server (MOSS) 2007 after we encountered several major issues prior to deployment. The first issue that..."
aliases: ["/blog/jjameson/archive/2007/10/30/dumping-moss-2007-variations-part-2.aspx", "/blog/jjameson/archive/2007/10/31/dumping-moss-2007-variations-part-2.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/10/31/dumping-moss-2007-variations-part-2.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/10/31/dumping-moss-2007-variations-part-2.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In [part 1](/blog/jjameson/2007/10/30/dumping-moss-2007-variations-part-1) of
this series, I talked about my current customer's decision to abandon the use of
the variations feature in Microsoft Office SharePoint Server (MOSS) 2007 after
we encountered several major issues prior to deployment. The first issue that I
described is the incompatibility of out-of-the-box (OOTB) content types and
variations. Refer to the previous post for a simple set of repro steps to break
the variations feature by changing the content type of pages.

One thing I forgot to mention in the previous post that is related to the issues
with content types and variations:

You may think you are safe from the variation/content type bugs if you don't
change the content type of the default page. However, also be aware that the
variations feature does not enable content types on the **Pages** libraries in
the variation sites. Consequently, if you have any pages that specify a content
type other than the default **Page**, **Article Page**, or **Welcome Page**,
then you are going to break the variations feature if you attempt to propagate
pages before the additional content types have been enabled on the **Pages**
libraries (either manually or through some automated fashion). Oh, and good luck
enabling these additional content types before propagation begins after creating
a new variation label. Feature stapling (i.e. not relying on the OOTB
**Publishing with Workflow** site definition like we did) *might* allow you to
curcumvent this problem. However we did not explore this possibility, I am just
thinking out loud here.

Anyway, on to the second major issue that we encountered...

It is important to note that we are currently on the second major version (i.e.
v2) of the solution for this customer. In v1, we migrated a custom document
management system to MOSS 2007. Consequently, prior to starting our v2 solution,
we had about 120,000 items in our **AllUserData** table and approximately
150,000 items in our **AllDocs** table.

As I mentioned in the previous post, a big part of our v2 solution is migrating
this customer's 3,200 FAQs and 500 other pages from a legacy custom code soluton
to MOSS. Also note that this customer provides this content in English,
Japanese, Korean, Simplified Chinese, and Traditional Chinese. Thus our initial
design included the following variations: **en-US** (the source label),
**ja-JP**, **ko-KR**, **zh-CN**, and **zh-TW**.

After we modified our content migration tools to circumvent the issues with
variations and content types (as described in
[part 1](/blog/jjameson/2007/10/30/dumping-moss-2007-variations-part-1)), we
were able to migrate FAQ pages to the **en-US** site and the variations system
subsequently propagated each page to each variation site (e.g. **ja-JP**). The
problem was that, in our Test environment (TEST), we were only propagating about
one page (to the four variation sites) every minute. Thus our initial
"guess-timate" was around 3,700 minutes (around 62 hours) for the initial
migration. Ugh!

What was really puzzling is that in our Development environment (DEV), we
observed propagation throughput of about 3 pages every minute. Note that DEV is
comprised of a
[bunch of VMs](/blog/jjameson/2007/06/09/moss-development-environment-and-windows-update-bug)
running on a single server. TEST has separate physical servers for the Web
front-end, SSP, and active-passive SQL Server cluster (complete with SAN
storage). However, it is also important to note that we only configured two
variation labels (**en-US** and **ja-JP**) in DEV (we just needed to prove that
our approach worked), whereas TEST had all five variation labels (**en-US**,
**ja-JP**, **ko-KR**, **zh-CN**, and **zh-TW**). It certainly seemed, in my mind
at least, that the "beefier" hardware in TEST should more than compensate for
the additional variation labels.

However, the problem got worse -- meaning the throughput of the propagation to
the variation sites deteriorated rapidly as more and more pages were loaded.

At that point, I sent a message out to one of our internal lists inquiring if
anyone had experienced similar performance problems with variations. An MCS peer
in Europe replied that he had seen 3,000 pages take 7 days to propagate. Ouch.

After investigating the perf problem, I discovered that the fundamental problem
was due to the very lengthy SQL SELECT inside one of the PRIME (a.k.a. the
SharePoint content deployment API) stored procedures (specifically,
**proc\_DeplGetListItemData**). On the Friday we kicked off the FAQ migration,
this sproc was taking 30-40 seconds to complete. By the following Tuesday, it
was taking roughly 100 seconds for each execution. Note that
**proc\_DeplGetListItemData** is just one part of the variation propagation. On
Tuesday morning, each variation page propagation was taking about 10 minutes (to
the four variation sites).

Seven days for 3,000 pages was looking optimistic at that point (most likely
because we had more variation labels than my peer in Europe).

Examining the execution plan for the lengthy SQL statement inside
**proc\_DeplGetListItemData** identified that a Key Lookup was being performed
on the **AllUserData** table. In our environment we were seeing on the order of
2M reads for this one SELECT statement. It is also important to note that users
began complaining about the overall performance of SharePoint in this
environment while the migration was running.

The slow performance was subsequently confirmed by SQL Profiler traces. The
traces identified that INSERTs, UPDATEs, and SELECTs on the **AllUserData**
table (such as **proc\_AddListItem**, **proc\_UpdateDocument**,
**proc\_UpdateListItem**, **proc\_CheckOutDocument**, etc.) would often require
more than 60 seconds to complete -- obviously due to the very high I/O being
performed on the **AllUserData** table.

In parallel with another team member raising the issue to PSS and the Product
Group, I then analyzed the SELECT statement in **proc\_DeplGetListItemData** to
determine an index that could be added to eliminate the Key Lookup and greatly
reduce the time required to execute this sproc.

I tested three different indexes on the **AllUserData** table and found similar
results for all three -- decreasing the execution time from around 140 seconds
to approximately 2.2 seconds. Of the three indexes, the following was deemed to
be the "least expensive" (in terms of overhead with DML operations):

```
CREATE INDEX Idx_AllUserData_Tmp2
ON AllUserData
(
      tp_DirName
      , tp_LeafName
      , tp_SiteId
      , tp_Level
)
```

A couple of things to note about the index:

- The naming convention implies that this is not intended to be final
  manifestation of this fix (you cannot add an index -- or make any other
  changes -- to your SharePoint database unless it has been approved by PSS,
  unless you don't object to falling under the "unsupported" moniker)
- Since **tp\_DirName** specifies a server relative URL, this was chosen as
  the first column in the index -- rather than **tp\_SiteId**.
- In environments where the content database only contains a single site
  collection (such as ours), including **tp\_SiteId** provides essentially no
  value (i.e. it does not influence the selectivity of the index)

Lastly, it is worth noting that even after adding the index to TEST, we were
still experiencing relatively long execution times with other sprocs during the
variation page propagation -- specifically **proc\_DeplAddExportObjectLinks**
and, to a lesser extent, **proc\_DeplCalculateChildrenToExport**. I attempted to
resolve this in a similar fashion by creating a similar index on the **AllDocs**
table. However, the initial index attempted did not improve performance and I
abandoned additional tuning efforts due to schedule and resource constraints.

SQL Profiler traces continued to show some latency in the various PRIME sprocs:

- Over a 3-1/2 hour period, approximately 900 operations took longer than 1
  second (a small fraction compared with before the index was added)
- Of the 900, a small fraction were in the 10-12 second range (e.g.
  **proc\_DeplAddExportObjectLinks**)

Overall, performance drastically improved with the new index on **AllUserData**.
However, even with the index, variations page propagations still took around one
minute to propagate to four variation sites (just prior to adding the index,
page propagations were taking about 10 minutes and gradually taking longer as
more pages were added).

By the way, as of this time of this post, we never did receive a "blessing" from
PSS or the Product Group on my index recommendation. In fact, we are still
waiting for PSS to repro the performance problem (we ended up having to ship
them a copy of our database). It baffles me that PSS doesn't have sample "large"
databases in their labs for reproducing performance problems.

Even though it won't do my current customer any good (since they decided to
abandon variations), we are still pursuing a QFE for the variations propagation
performance problem, hoping that it will potentially help other customers in the
future.

[[Part 3](/blog/jjameson/2007/11/02/dumping-moss-2007-variations-part-3) in this
series is now available.]

