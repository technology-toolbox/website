---
title: "Always Include \"Path\" In Search Core Results Web Part"
date: 2009-04-01T07:41:00-06:00
description:
  Here is a bug in Microsoft Office SharePoint Server (MOSS) 2007 that I've
  stumbled across at least twice in the last couple of years...I'm hoping that
  if I take the time to blog about it, I won't forget it again. If you don't
  include Path in the SelectColumns...
aliases:
  [
    "/blog/jjameson/archive/2009/03/31/always-include-path-in-search-core-results-web-part.aspx",
    "/blog/jjameson/archive/2009/04/01/always-include-path-in-search-core-results-web-part.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/04/01/always-include-path-in-search-core-results-web-part.aspx"
---

Here is a bug in Microsoft Office SharePoint Server (MOSS) 2007 that I've
stumbled across at least twice in the last couple of years...I'm hoping that if
I take the time to blog about it, I won't forget it again.

If you don't include **Path** in the **SelectColumns** property of the Search
Core Results Web Part, then you will find that when you click one of the items
in the search results, nothing happens. In other words, the `href` attribute of
the anchor element is not set.

If you look at the default XSLT stylesheet for the Core Results Web Part, you
won't find any mention of "path" -- however, that apparently doesn't mean that
it is safe to remove from the list of selected columns.

If you don't include **Path**, then the **url** element in the search results
XML is not populated and consequently the search result links are rendered
useless.

...and that's no April Fools joke!
