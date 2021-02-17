---
title: "Bug: MOSS 2007 Search Scope with Property Query Rules Only Is Considered Empty"
date: 2009-03-05T01:23:00-07:00
excerpt: "In Microsoft Office SharePoint Server (MOSS) 2007 version 12.0.0.6335 (i.e. the December 2008 CU), there appears to be a bug where a scope that only contains Property Query rules is not recognized as having any rules at all (i.e. empty) and therefore..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/05/bug-moss-2007-search-scope-with-property-query-rules-only-is-considered-empty.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/05/bug-moss-2007-search-scope-with-property-query-rules-only-is-considered-empty.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In Microsoft Office SharePoint Server (MOSS) 2007 version 12.0.0.6335 (i.e. the         December 2008 CU), there appears to be a bug where a scope that only contains **Property Query** rules is not recognized as having any rules at all (i.e.         empty) and therefore is not compiled.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_MOSS%202007%20Search%20Scope%20bug.jpg"
alt="Search scope bug in MOSS 2007 December 2008 CU"
height="424"
width="600"
title="Figure 1: Search scope bug in MOSS 2007 December 2008 CU" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_MOSS%202007%20Search%20Scope%20bug.jpg)

To hack around this bug, add a **Web Address** rule, as shown below.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_MOSS%202007%20Search%20Scope%20bug%20-%20workaround.jpg"
alt="Workaround for search scope bug"
height="425"
width="600"
title="Figure 2: Workaround for search scope bug" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_MOSS%202007%20Search%20Scope%20bug%20-%20workaround.jpg)

     Note that in the screenshot above, **http://foobar** is the Web application     that I want to include content from, while excluding the "Pages" libraries themselves     from the results (i.e. items where **contentclass = STS\_List\_850**).     In other words, I want individual pages within the Pages library to appear in the     search results -- just not the actual **Pages** libraries themselves     (e.g. the **All Items** view of the list).     

Also note that this is not the exhaustive list of rules required to exclude the         various SharePoint items from search results that you most likely don't want to         show to users on an Internet-facing MOSS site. I'll defer that to a [subsequent post](/blog/jjameson/2009/03/05/excluding-various-sharepoint-items-from-search-results-on-internet-facing-moss-sites).

