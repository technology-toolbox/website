---
title: "Dumping MOSS 2007 Variations - Part 3"
date: 2007-11-02T02:53:00-07:00
excerpt: "In part 1 and part 2 of this series, I talked about some major issues with the variations feature in Microsoft Office SharePoint Server (MOSS) 2007 that caused my current customer to abandon using them on their new Internet site. 
 Here is a brief summary..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/11/02/dumping-moss-2007-variations-part-3.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/11/02/dumping-moss-2007-variations-part-3.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In [part 1](/blog/jjameson/2007/10/30/dumping-moss-2007-variations-part-1) and [part 2](/blog/jjameson/2007/10/31/dumping-moss-2007-variations-part-2) of this series, I talked about some major issues with the variations         feature in Microsoft Office SharePoint Server (MOSS) 2007 that caused my current         customer to abandon using them on their new Internet site.

Here is a brief summary of the three major issues we encountered with variations         prior to deployment:

1. Incompatibility of out-of-the-box (OOTB) content types and variations (refer to
   [part 1](/blog/jjameson/2007/10/30/dumping-moss-2007-variations-part-1) for more details)
2. Page propagation slows down dramatically as the number of pages grows (refer to
   [part 2](/blog/jjameson/2007/10/31/dumping-moss-2007-variations-part-2) for more details)
3. Attempting to create a new variation label after creating a large number of sites
   and pages results in an out-of-memory error on the server

I have not previously covered this last problem in the previous posts. If you manage         to circumvent the first two problems, at some later point in time it is certainly         possible you will need to add a new label to support another language (or, as in         our case, perhaps you discover that you need to recreate a label because someone         -- who shall remain nameless -- accidentally deletes one of the variation sites         by mistake). [If the latter is the case, hopefully you'll be fortunate like we were         and this won't happen on your PROD site.]

If you are using variations, and you happen to notice your OWSTimer.exe process         consuming 1.5 GB of memory (like we did), then problem #3 is likely the culprit.         Fortunately, the variations memory leak that causes this has already been fixed         in a QFE (and, I believe, is scheduled to be included in MOSS 2007 Service Pack         1). Based on the SRX (service request) that I read, it appears this can be obtained         earlier through the normal QFE process. However, we never pursued obtaining the         patch and installing it in our environment.

> **Update (2007-11-28)**
>
> According to a [follow-up](http://blogs.technet.com/stefan_gossner/archive/2007/11/15/some-comments-on-common-variation-problems.aspx) by Stefan Goßner, only part of this bug is fixed in SP1. You'll need a post-SP1 QFE to completely resolve the out-of-memory issue.

As I have mentioned before, the decision to abandon variations was not an easy one         to make and it is not possible to point to one particular problem as the proverbial         "straw."

Ultimately, though, I believe the bulk of the decision fell on performance, or more         accurately, the lack thereof. Take a look at the following graph, which shows the         elapsed time required to propagate each FAQ page from the source **en-US**         site to the four variation sites (**ja-JP**, **ko-KR**,         **zh-CN**, and **zh-TW**).

![Elapsed Time for Variation Page Propagation](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_Variation-Page-Propagation%20(before).jpg "Elapsed Time for Variation Page Propagation")
Figure 1: Elapsed Time for Variation Page Propagation

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_Variation-Page-Propagation%20%28before%29.jpg)

Notice how the time required to propagate each page increases substantially as the         number of pages increases. As I pointed out in [part 2](/blog/jjameson/2007/10/31/dumping-moss-2007-variations-part-2), I believe this is due to the use of the **Relationships List** (stored in the **AllUserData** table) in combination with         the content deployment API. Also notice how the time required to propagate each         page dropped substantially after adding the index described in [part 2](/blog/jjameson/2007/10/31/dumping-moss-2007-variations-part-2). Lastly, notice how long it took to propagate 2074 pages before the         index, compared with the remaining 959 pages after the index was added.

The following graph "zooms in" on the time period after the index was added.

![Elapsed Time for Variation Page Propagation with New AllUserData Index](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_Variation-Page-Propagation%20(after).jpg "Variation Page Propagation with New AllUserData Index")
Figure 2: Elapsed Time for Variation Page Propagation with New AllUserData Index

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_Variation-Page-Propagation%20%28after%29.jpg)

It seems like it took SQL Server a little while to optimize based on the new index,         but it levels out around 40 seconds per page. Compared with the 10-minute propagations         before the new index on **AllUserData**, this seems fantastic. However,         that is only a relative comparison.

The problem is that, as I described in [part 2](/blog/jjameson/2007/10/31/dumping-moss-2007-variations-part-2), we are currently on "v2" of our solution -- which has about 3,700         pages (of which, about 3,200 are FAQs). However, the "v3" solution is going to have         about 20,000 more pages. Consequently, even at an optimistic 40 seconds per variation         page propagation, we were forecasting about 9-1/2 days for migrating the pages from         the legacy system (20,000 pages x 40 seconds/page / 60 seconds/minute / 60 minutes/hour         / 24 hours/day). However, we really don't know the accuracy of that estimate without         additional testing.

The bottom line is that, even with the index I proposed in [part 2](/blog/jjameson/2007/10/31/dumping-moss-2007-variations-part-2), the content deployment (a.k.a. PRIME) API used by the variations         feature to propagate pages incurs signficant overhead.

As a comparison, I believe our last FAQ migration (without variations) completed         in about 4 hours for 16,000 pages (3,200 pages x 5 languages). Note, however, that         in order to achieve this throughput, we had to forego all of the value-add of the         variations feature, such as automatically creating a new FAQ on the **ja-JP**,         **ko-KR**, **zh-CN**, and **zh-TW** sites         (in preparation for translation), when the corresponding FAQ is approved on the         **en-US** site.

Like I said, it was a difficult decision, and a painful one at that.

