---
title: Excluding Various SharePoint Items from Search Results on Internet-Facing MOSS Sites
date: 2009-03-05T08:25:00-07:00
excerpt:
  When using Microsoft Office SharePoint Server (MOSS) 2007 on an
  Internet-facing site, you almost certainly don't want to include items like
  announcements, contacts, etc. in search results. Consequently, I recommend
  customers create additional search...
aliases:
  [
    "/blog/jjameson/archive/2009/03/04/excluding-various-sharepoint-items-from-search-results-on-internet-facing-moss-sites.aspx",
    "/blog/jjameson/archive/2009/03/05/excluding-various-sharepoint-items-from-search-results-on-internet-facing-moss-sites.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/05/excluding-various-sharepoint-items-from-search-results-on-internet-facing-moss-sites.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/05/excluding-various-sharepoint-items-from-search-results-on-internet-facing-moss-sites.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

When using Microsoft Office SharePoint Server (MOSS) 2007 on an Internet-facing
site, you almost certainly don't want to include items like announcements,
contacts, etc. in search results.

Consequently, I recommend customers create additional search scopes that
explicitly exclude these items and use these scopes instead (for example, by
explicitly setting the **Scope** property on the Core Results Web Part).

I actually recommend against making any changes to the out-of-the-box (OOTB)
**All Sites** scope so that you can continue to use that scope for internal
users -- as well as for troubleshooting purposes. [This is my general
recommendation for almost all things SharePoint -- or other solutions for that
matter -- meaning that if you can easily customize the technology to suit your
needs without changing the OOTB behavior, then you should certainly use that
approach. Trust me, this makes your support calls go much, much faster.]

Here are the exclude rules that my custom
`SharePointSearchHelper.EnsureScopeIsLimitedToDocumentsAndWebPagesOnly()` method
applies to a search scope intended for Internet use:

- contentclass = STS\_List\_850
- contentclass = STS\_List\_Announcements
- contentclass = STS\_List\_Contacts
- contentclass = STS\_List\_DiscussionBoard
- contentclass = STS\_List\_DocumentLibrary
- contentclass = STS\_List\_Events
- contentclass = STS\_List\_GanttTasks
- contentclass = STS\_List\_GenericList
- contentclass = STS\_List\_IssueTracking
- contentclass = STS\_List\_Links
- contentclass = STS\_List\_PictureLibrary
- contentclass = STS\_List\_Survey
- contentclass = STS\_List\_Tasks
- contentclass = STS\_List\_WebPageLibrary
- contentclass = STS\_List\_XMLForm
- contentclass = STS\_ListItem\_Announcements
- contentclass = STS\_ListItem\_Contacts
- contentclass = STS\_ListItem\_DiscussionBoard
- contentclass = STS\_ListItem\_Events
- contentclass = STS\_ListItem\_GanttTasks
- contentclass = STS\_ListItem\_GenericList
- contentclass = STS\_ListItem\_IssueTracking
- contentclass = STS\_ListItem\_Links
- contentclass = STS\_ListItem\_Survey
- contentclass = STS\_ListItem\_Tasks
- contentclass = STS\_ListItem\_XMLForm

Note that, as described in my
[previous post](/blog/jjameson/2009/03/05/bug-moss-2007-search-scope-with-property-query-rules-only-is-considered-empty),
there's a bug in MOSS 2007 in which a search scope that contains **Property
Query** rules only is not recognized as having any rules at all. Consequently,
you will need to explicitly include the URL of your Web application using a
**Web Address** rule.
