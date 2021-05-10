---
title: Excluding Various SharePoint Items from Search Results on Internet-Facing MOSS Sites
date: 2009-03-05T08:25:00-07:00
description:
  When using Microsoft Office SharePoint Server (MOSS) 2007 on an
  Internet-facing site, you almost certainly don't want to include items like
  announcements, contacts, etc. in search results. Consequently, I recommend
  customers create additional search...
aliases:
  [
    "/blog/jjameson/archive/2009/03/04/excluding-various-sharepoint-items-from-search-results-on-internet-facing-moss-sites.aspx",
    "/blog/jjameson/archive/2009/03/05/excluding-various-sharepoint-items-from-search-results-on-internet-facing-moss-sites.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/03/05/excluding-various-sharepoint-items-from-search-results-on-internet-facing-moss-sites.aspx"
---

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

- contentclass = STS_List_850
- contentclass = STS_List_Announcements
- contentclass = STS_List_Contacts
- contentclass = STS_List_DiscussionBoard
- contentclass = STS_List_DocumentLibrary
- contentclass = STS_List_Events
- contentclass = STS_List_GanttTasks
- contentclass = STS_List_GenericList
- contentclass = STS_List_IssueTracking
- contentclass = STS_List_Links
- contentclass = STS_List_PictureLibrary
- contentclass = STS_List_Survey
- contentclass = STS_List_Tasks
- contentclass = STS_List_WebPageLibrary
- contentclass = STS_List_XMLForm
- contentclass = STS_ListItem_Announcements
- contentclass = STS_ListItem_Contacts
- contentclass = STS_ListItem_DiscussionBoard
- contentclass = STS_ListItem_Events
- contentclass = STS_ListItem_GanttTasks
- contentclass = STS_ListItem_GenericList
- contentclass = STS_ListItem_IssueTracking
- contentclass = STS_ListItem_Links
- contentclass = STS_ListItem_Survey
- contentclass = STS_ListItem_Tasks
- contentclass = STS_ListItem_XMLForm

Note that, as described in my
[previous post](/blog/jjameson/2009/03/05/bug-moss-2007-search-scope-with-property-query-rules-only-is-considered-empty),
there's a bug in MOSS 2007 in which a search scope that contains **Property
Query** rules only is not recognized as having any rules at all. Consequently,
you will need to explicitly include the URL of your Web application using a
**Web Address** rule.
