---
title: "Faceted Search in MOSS 2007 and the MSSDocProps Issue"
date: 2009-09-18T02:34:00+08:00
excerpt: "Many customers deploying Microsoft Office SharePoint Server (MOSS) 2007 often have a requirement to provide some kind of \"faceted search\" feature that allows users to quickly and easily narrow their search results. Before I ever knew the commonly accepted..."
draft: true
categories: ["SharePoint", "Infrastructure"]
tags: ["MOSS 2007", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/18/faceted-search-in-moss-2007-and-the-mssdocprops-issue.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/18/faceted-search-in-moss-2007-and-the-mssdocprops-issue.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

Many customers deploying Microsoft Office SharePoint Server (MOSS) 2007 often
have a requirement to provide some kind of "faceted search" feature that allows
users to quickly and easily narrow their search results. Before I ever knew
the commonly accepted name for this feature, I had already started developing
a custom "Narrow Search Results Web Part" for the
[Agilent Technologies site](http://www.chem.agilent.com/) that I
have mentioned a couple of times in previous posts. [You can
[see this Web Part in action](http://www.chem.agilent.com/en-US/Search/Pages/default.aspx?k=purification&a=%20scope:%22English%20%28U.S.%29%20Content%22+MainCat:%22Products+%26+Services%22) by searching for something like "purification"
and then clicking one of the categories under the **Narrow Your Results**
heading, such as **Products & Services** -- which naturally
filters your search results to items matching the selected category. You can
also "drill down" to further refine your results.]

The custom Narrow Search Results Web Part simply performs a search (using
the search criteria specified in the query string), tells SharePoint to return
a maximum of <var>N</var> results (where <var>N</var> is a configurable parameter on the Web Part),
and subsequently performs a "group by" operation on the search results.

Shortly after completing a proof-of-concept of this feature for Agilent,
while at the Microsoft TechReady conference in Seattle I was speaking with one
of the Program Managers for MOSS Search and mentioned how I had wished that
this kind of feature came out-of-the-box with MOSS 2007. With the ability to
specify *[property filters](http://msdn.microsoft.com/en-us/library/ms582745.aspx)*in addition to simple keywords, it certainly didn't
take me long to create that initial POC (if memory serves, it was less than
a day) but I obviously would have preferred to avoid custom code if at all possible.
That's when he told me that what I was referring to was called "faceted search"
and that it was also one of the most requested features in MOSS Search. He also
told me to go take a look at the
[Faceted Search project on CodePlex](http://facetedsearch.codeplex.com/).

Well, long story short, I did take a look at Leo's Faceted Search -- actually
doing some demos of it for Agilent -- but unfortunately found it didn't provide
the "drill down" user experience that Agilent was looking for. Hence we ended
up sticking with the Narrow Search Results Web Part (and obviously spending
a lot more development time turning the original POC code into a robust feature
suitable for production use). [Note that on the Agilent search results page,
you can drill down from MainCat (Main Category) to MidCat (Middle Category),
and then finally from MidCat down to SubCat (Subcategory). At the time we implemented
this, Leo's Faceted Search did not support the concept of dependent facets (i.e.
hiding one or more facets until another facet is selected).]

The biggest problem that we discovered is that using property filters in
MOSS 2007 comes at a fairly steep price. This is what I commonly refer to as
the "MSSDocProps issue."

Allow me to explain...

Rather than rewriting stuff I've already said before, I'll simply take an
excerpt from an email I originally sent back on 2008-08-20:

> I believe the primary reason why the load on the Search database is extremely
> high is due to the changes in MOSS 2007 with regards to the "property store"
> (i.e. the database used to perform property searches, as opposed to "full
> text searching" against the content index). In SPS 2003 the property store
> was implemented as a local "Jet" database on each query server and thus
> the load incurred when performing property searches was distributed amongst
> the various query servers in the farm. However, in MOSS 2007, a couple of
> key changes were made. First, the property store was moved to SQL Server
> (i.e. the Search database) and therefore the load is now delegated from
> the query servers back to a single database server. Second, the manner in
> which MOSS 2007 filters its results using the property store uses some,
> shall we say, "interesting" queries (something along the lines of "SELECT
> TOP 2000 ... FROM MSSDocProps ..." in which the "2000" number varies depending
> on how many search results are requested). Judging from [SRX for another
> customer], it appears that our "official response" to this problem is to
> offload the Search database to another SQL Server farm.

Here's a more lengthy explanation from an email I sent a couple of months
later (when Agilent was still experiencing problems on their site):

> Fact: MOSS 2007 is more resource intensive than SPS 2003. I believe with
> the release of the documentation for SP1, this was formally acknowledged
> by the SharePoint team.
>
> Fact: The Frontier physical architecture was originally planned for SPS
> 2003. In hindsight, this should have been scaled accordingly when the decision
> was made to go with MOSS 2007 instead of SPS 2003 (and MCMS 2002). However,
> there were obviously no numbers in the beginning, and very limited experience/knowledge
> on highly scalable MOSS environments. As the old saying goes, hindsight
> is...blah...blah...blah
>
> Fact: In SPS 2003, the "property store" (which contains the metadata
> for items in the content index) was implemented as a local database on each
> query server. In other words, the data -- and processing -- of properties
> for search results was distributed (across each query server in the farm).
>
> Fact: In MOSS 2007, the "property store" was moved to SQL Server. Consequently,
> the data (and a good portion of the processing load) that was previously
> distributed across the query servers is now on the backend SQL Server.
>
> Fact: In the current implementation, SharePoint Search fetches a large
> number of results from the MSSDocProps table in order to filter search results
> (if you don't like my definition of "large", feel free to ask any SQL DBA
> if they like to see queries like "SELECT TOP 1820 ..."). These specific queries
> have often been reported as problematic for numerous customers [...]
>
> Fact: Issues with "property queries" have been acknowledged [...] by
> the SharePoint support organization as well as the product team. In some
> cases, product functionality has actually been removed in order to help
> mitigate these issues (for example,
> [KB 950437](http://support.microsoft.com/kb/950437/) -- which
> removes the **Contains**/**Does not contain**
> operators from the Advanced Search Box). In other cases, Microsoft has formally
> recommended to customers to offload the SQL portion of SharePoint Search
> to a different database server (which, in any enterprise implementation,
> means a new SQL Server cluster).
>
> [(Additional ranting intentionally removed)]

The crux of the issue is that if you frequently specify property filters
in MOSS 2007 Search (which typically means you've provided some kind of faceted
search -- since users don't typically use Advanced Search or explicitly type
property filters in the search box) then you might very well be seeing performance
issues on your site. This of course depends on the details of your particular
environment. For example: How high is the load on your farm? How "beefie" is
your backend SQL Server cluster? How many "spindles" are you using on your SAN
and what RAID configuration are you using? Are these spindles dedicated or are
they shared?

It's these last two questions that I want to address in this post.

My formal recommendation for any large-scale MOSS 2007 deployment using faceted
search (i.e. think public-facing Internet sites, like Agilent, or even large
intranet sites with many thousands of users) is to "isolate" the SSP Search
database (i.e. the "property store") so that when disk queue lengths inevitably
start to grow due to queries like "SELECT TOP 1820 ..." , at least you minimize
the risk of bringing down your farm.

Note that the proposed solution of spinning up a new SQL Server cluster in
order to offload Search is really only something that should be considered in
the most extreme of cases. I seriously doubt there are many SharePoint deployments
out there that require this level of remediation.

Provided you dedicate a sufficient number of RAID 10 spindles to the SSP
Search database, you should be fine. However, that certainly doesn't mean you
shouldn't be monitoring your backend SQL node(s) and corresponding SAN storage
for bottlenecks.

