---
title: "Master Page Not Inherited by Document Center Sites"
date: 2007-05-06T01:55:00-07:00
excerpt: "This isn't a recent discovery -- I first encountered this in late Februrary -- but I realized that I had not yet covered this issue in my blog. 
 If you happen to be using the Document Center template in Microsoft Office SharePoint Server (MOSS) 2007..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/05/06/master-page-not-inherited-by-document-center-sites.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/05/06/master-page-not-inherited-by-document-center-sites.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

This isn't a recent discovery -- I first encountered this in late Februrary
-- but I realized that I had not yet covered this issue in my blog.

If you happen to be using the Document Center template in Microsoft Office
SharePoint Server (MOSS) 2007 for a site and you also elect to use a custom
master page (really any master page other than default.master -- including BlueBand.master,
BlackBand.master, etc.) then you will notice that the Document Center site does
not inherit the master page. Sure, you can go to the **Site Master Page
Settings** for the parent site, select a different master page, check
the **Reset all subsites to inherit this Site Master Page setting**
checkbox, and click **OK** as many times as you want, but it is
not going to make any difference. The Document Center site will always look
like it is rendered with default.master.

Evidently the root of the issue is related to the fact that the Document
Center exposes links (e.g. View All Site Content and Site Hierarchy) to \_layouts/viewlsts.aspx
directly to end users (these "system pages" are typically only used by administrators
and contributors). The Document Center has its own master page (and I believe
the underlying pages are hardcoded to use that master page, which is why you
cannot explicitly select a different master page for the Document Center site).

The workaround is to overwrite the default.master in the \_catalogs/masterpage
library of the Document Center site. Fortunately SharePoint Designer makes this
very easy -- although it would be nice to "script" this so that we did not have
to do this manually after rebuilding our site in DEV and TEST. Perhaps I'll
get around to automating this someday.
