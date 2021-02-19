---
title: "Still Crazy (About Typed DataSets) After All These Years"
date: 2010-04-22T07:02:00-06:00
excerpt: "First off, my apologies to Paul Simon regarding the title of this blog post -- but I simply couldn't resist ;-) 
 When architecting and building solutions for customers, I tend to make heavy use of typed DataSets. 
 I believe I used them on my very..."
aliases: ["/blog/jjameson/archive/2010/04/21/still-crazy-about-typed-datasets-after-all-these-years.aspx", "/blog/jjameson/archive/2010/04/22/still-crazy-about-typed-datasets-after-all-these-years.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/22/still-crazy-about-typed-datasets-after-all-these-years.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/22/still-crazy-about-typed-datasets-after-all-these-years.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

First off, my apologies to [Paul Simon](http://en.wikipedia.org/wiki/Still_Crazy_After_All_These_Years) regarding the title of this blog post -- but I simply couldn't         resist ;-)

When architecting and building solutions for customers, I tend to make heavy use         of typed DataSets.

I believe I used them on my very first .NET project almost ten years ago, because         I still clearly remember Mike Pizzo (one of the original architects on the ADO.NET         team) explaining to us how we should think of DataSets as a "scratch pad" for disconnected         data (as opposed to the old DAO, RDO, and ADO objects -- which thankfully are a         thing of the past for most organizations). [Okay, I suppose that, technically, you         could disconnect an ADO Recordset from the underlying database, but that's not how         it was frequently used.]

The tooling for typed DataSets has always been very good (at least in my opinion).         You can create a typed DataSet very quickly using the designer in Visual Studio         and restructure an existing typed DataSet with ease. In my opinion, this is almost         always faster than writing the equivalent C# code directly.

I also love the way typed DataSets can be somewhat complex -- in terms of the business         rules that are encapsulated by them -- yet at the same time very easy to understand         for developers that are not familiar with your solution (e.g. new team members or         developers responsible for maintaining the solution).

For example, consider the typed DataSet shown below.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Typed%20DataSet%20example%20(ScorecardData).png"
alt="Typed DataSet example (ScorecardData)"
height="330"
width="600"
title="Figure 1: Typed DataSet example (ScorecardData)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Typed%20DataSet%20example%20%28ScorecardData%29.png)

What can you tell just by looking at the diagram? What are the business rules that         are enforced simply by the structure of the tables, relationships, and columns?

- First, you can see that **ScorecardData** (the name of the typed DataSet)
  contains information about scorecards for client sites (i.e. locations). Note that
  the term "scorecard" in this scenario refers to a collection of [key performance indicators](http://en.wikipedia.org/wiki/Key_performance_indicator) (KPIs) displayed on a [dashboard](http://en.wikipedia.org/wiki/Dashboards_%28management_information_systems%29).
- Each scorecard is simply a collection of items (essentially the KPIs) -- with each
  item corresponding to a particular client site.
- Each scorecard item may have one or more instances of a KPI status (i.e. the measure
  of the KPI for a particular period). The status -- i.e. "Exceeds", "Meets", or "Does
  Not Meet" (a.k.a. "Green", "Yellow", or "Red") is determined by a corresponding
  set of *thresholds*.

Why about the primary keys on the various tables?

- The primary key on the **Scorecard** table is obvioulsy **ScorecardId**.
- The primary key on the **ScorecardItem** table is **ScorecardItemId** -- although not quite as obvious as the **Scorecard** table, since
  there are multiple key icons shown on that table in the designer. However, you can
  infer this from the relationship between **ScorecardItem** and **KpiStatus** -- and the fact that only the **ScorecardItemId**
  column appears in the **KpiStatus** table. [Wouldn't it be nice if
  the DataSet designer in Visual Studio showed a different icon for the primary key
  from for other unique keys?]
- The primary key on the **ClientSite** table is **ClientSiteId**
  -- which again, can be inferred from the relationship between the **ClientSite**
  and **ScorecardItem** tables.
- The primary key on the **KpiStatus** table is **(ScorecardItemId,
  Period)** -- thus allowing each scorecard item to specify one or more KPI
  status values (for different time periods).

If you were to right-click on the **ScorecardId** column in the **ScorecardItem** table in Visual Studio and then click **Edit key...**,         you would see that there is a unique constraint on (**ScorecardId**,         **ClientSiteId**, **KpiName**). In other words, each scorecard         can only specify one scorecard item for given site and KPI (e.g. we don't want to         allow "Site1" to have two scorecard items that refer to "KeyPerformanceIndicator1").

Also note that since a primary key on a table must be unique, the constraint on         the **KpiStatus** table ensures that a scorecard item (i.e. a KPI)         is only allowed to specify one KPI status for a particular time period. It just         doesn't make sense that "KeyPerformanceIndicator1" could be both "Green" and "Red"         for, say, the "2010 Q1" time period -- it has to be one or other (or "Yellow", I         suppose).

We can also see that there is a unique constraint on the **ClientSiteName** column in the **ClientSite** table. Here's where things get         slightly more complicated. Suppose that we wanted to retrieve data from this DataSet         by filtering on the **ClientSiteName**. In other words, we want to         only show KPI information for a particular site. Seems reasonable, right?

What if two completely different clients each had a site named "Headquarters"? That         could definitely be a problem, because we certainly wouldn't want to show one client's         data to a different client.

However, in order for that to actually occur, we would first have to populate the         DataSet with information from two different clients. Since that wasn't how I intended         this DataSet to be used, I chose to add a unique constraint on the **ClientSiteName**         column. If, at some later point in time, new scenarios were added and I needed to         use this typed DataSet to store data from multiple clients, then I would probably         add a **Client** table and adjust the relationships and constraints         accordingly.

Okay, so where exactly are we -- after all this verbiage?

The point I was hoping to make so far is that -- without looking at a single line         of code or any documentation (which many developers don't like to read anyway) --         we know quite a bit about the solution, including some of the business rules, just         by looking at the typed DataSet.

In my next post, I'll discuss some smart ways we can interact with a typed DataSet         when actually building a solution.

[I know the content of this post is very elementary, but bear with me, I promise         things will get more interesting in subsequent posts ;-) ]

