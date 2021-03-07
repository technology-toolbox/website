---
title: "MOSS 2007 Master Page Comparison"
date: 2009-09-19T07:56:00-06:00
excerpt: "This morning I came across an old (June 2007) Excel spreadsheet that I created back when I was working on the Agilent Technologies project. The spreadsheet lists the various placeholder elements in both application.master and default.master for Microsoft..."
aliases: ["/blog/jjameson/archive/2009/09/18/moss-2007-master-page-comparison.aspx", "/blog/jjameson/archive/2009/09/19/moss-2007-master-page-comparison.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/19/moss-2007-master-page-comparison.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/19/moss-2007-master-page-comparison.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

This morning I came across an old (June 2007) Excel spreadsheet that I created
back when I was working on the Agilent Technologies project. The spreadsheet
lists the various placeholder elements in both application.master and
default.master for Microsoft Office SharePoint Server (MOSS) 2007.

I put this together back when I was trying to figure out how to create a single
master page that could be used for:

- Site pages (e.g. /Library/default.aspx)
- System pages (e.g. /Library/Brochures/Forms/AllItems.aspx)
- Application pages (e.g. /Library/\_layouts/viewlsts.aspx)

Rather than make you open a spreadsheet attachment, I'll just copy the contents
into a simple table:

{{< table class="small" caption="MOSS 2007 Master Page Comparison" >}}

| <br>                    Placeholder<br>                 | <br>                    application.master<br>                 | <br>                    default.master<br>                 |
| --- | --- | --- |
|  PlaceHolderPageTitle  |  X  |  X  |
|  PlaceHolderAdditionalPageHead  |  X  |  X  |
|  PlaceHolderGlobalNavigation  |  X  |  X  |
|  PlaceHolderGlobalNavigationSiteMap  |  X  |  X  |
|  PlaceHolderSiteName  |  X  |  X  |
|  PlaceHolderSearchArea  |  X  |  X  |
|  PlaceHolderTopNavBar  |  X  |  X  |
|  PlaceHolderHorizontalNav  |   |  X  |
|  WSSDesignConsole  |  X  |  X  |
|  SPNavigation  |  X  |  X  |
|  PlaceHolderPageImage  |  X  |  X  |
|  PlaceHolderTitleLeftBorder  |  X  |  X  |
|  PlaceHolderTitleAreaClass  |  X  |  X\*  |
|  PlaceHolderTitleBreadcrumb  |  X  |  X  |
|  PlaceHolderPageTitleInTitleArea  |  X  |  X  |
|  PlaceHolderMiniConsole  |  X  |  X  |
|  PlaceHolderTitleRightMargin  |  X  |  X  |
|  PlaceHolderTitleAreaSeparator  |  X  |  X  |
|  PlaceHolderLeftNavBarDataSource  |  X  |  X  |
|  PlaceHolderCalendarNavigator  |  X  |  X  |
|  PlaceHolderLeftNavBarTop  |  X  |  X  |
|  PlaceHolderLeftNavBar  |  X  |  X  |
|  PlaceHolderLeftActions  |  X  |  X  |
|  PlaceHolderNavSpacer  |  X  |  X  |
|  PlaceHolderLeftNavBarBorder  |  X  |  X  |
|  PlaceHolderBodyLeftBorder  |  X  |  X  |
|  MSO\_ContentDiv\*\*  |   |  X  |
|  PlaceHolderBodyAreaClass  |  X  |  X\*  |
|  PlaceHolderPageDescriptionRowAttr  |  X  |   |
|  PlaceHolderPageDescription  |  X  |  X  |
|  PlaceHolderPageDescriptionRowAttr2  |  X  |   |
|  PlaceHolderMain  |  X  |  X  |
|  PlaceHolderBodyRightMargin  |  X  |  X  |
|  PlaceHolderFormDigest  |  X  |  X  |
|  PlaceHolderUtilityContent  |  X  |  X  |

{{< /table >}}

\* - The placeholder is tacked onto the end of the page

\*\* - &lt;PlaceHolder&gt; element (not &lt;asp:ContentPlaceHolder&gt;)

I found the differences between the two master pages very interesting, as well
as the fact that a couple of the placeholders are simply appended to the end of
default.master. It made me wonder if the two master pages were originally
supposed to remain in-sync (in terms of the number and names of placeholders),
but began to diverge as milestones approached and the SharePoint team was
pushing to simply get features working rather than strictly adhering to all of
the original design goals. [C'mon now, we all know that in the world of software
development, sometimes you have to do things that you fully intend to "come back
and fix later" but often end up never getting reworked as other, more important,
issues arise.]

What I learned from this analysis was that by creating a master page with all of
the placeholder elements specified in both application.master and
default.master, I would ensure that my custom master page would work for site
pages, system pages, and application pages.

I'll cover the topic of using a custom master page for application pages in a
[separate post](/blog/jjameson/2009/09/20/overriding-application-master-in-moss-2007).

