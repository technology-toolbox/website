---
title: Be Careful Using PublishingWeb.GetPagesListName()
date: 2009-05-28T08:52:00-06:00
description:
  "A couple of years ago when we began evaluating Language Packs for Microsoft
  Office SharePoint Server (MOSS) 2007, we discovered that after installing
  certain Language Packs, the \"Pages\" library may be localized -- including
  both the list name as well..."
aliases:
  [
    "/blog/jjameson/archive/2009/05/27/be-careful-using-publishingweb-getpageslistname.aspx",
    "/blog/jjameson/archive/2009/05/28/be-careful-using-publishingweb-getpageslistname.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/05/28/be-careful-using-publishingweb-getpageslistname.aspx"
---

A couple of years ago when we began evaluating Language Packs for Microsoft
Office SharePoint Server (MOSS) 2007, we discovered that after installing
certain Language Packs, the "Pages" library may be localized -- including both
the list name as well as the URL.

For example, after installing the Spanish Language Pack, if you had any code
that referred to something like "/en-US/Products/Pages/default.aspx" then you
would soon find that your code could break when referring to the equivalent page
on a localized site. For example, on a Spanish site, the equivalent URL would be
"/es-ES/Products/_Paginas_/default.aspx" (assuming you chose not to translate
"Products" when creating the Spanish site).

Fortunately, the SharePoint API provides the ` PublishingWeb.GetPagesListName`
method. In the following code sample, note how I use this method to avoid
hard-coding the name of the **Pages** library:

```C#
private static string GetDefaultSearchResultsPageUrl(
    SPWeb searchWeb)
{
    Debug.Assert(searchWeb != null);

    string pageListName = PublishingWeb.GetPagesListName(searchWeb);

    string defaultSearchResultsPageUrl = string.Format(
        CultureInfo.InvariantCulture,
        "{0}/{1}/Results.aspx",
        searchWeb.ServerRelativeUrl,
        pageListName);

    return defaultSearchResultsPageUrl;
}
```

Consequently, this code works as expected, even on localized sites created with
various Language Packs.

The problem -- as we discovered on our project this week -- is that you have to
be careful about how you use the "list name" returned from
`PublishingWeb.GetPagesListName()`.

Last week, another developer on our team sent out an email inquiring about how
to get the **Pages** list in a way that worked even with localized sites. I
quickly responded pointing out the `PublishingWeb.GetPagesListName()` method.

Unfortunately, the developer then added the following code to his feature:

```C#
SPList pages = web.Lists[PublishingWeb.GetPagesListName(web)];
```

This seems natural, after all, since `web.Lists` is simply an `SPListCollection`
and `SPListCollection` provides the following property:

```C#
public SPList this [string strListName] { get; }
```

This appeared to work during his unit testing and so he checked in his changes.
The problem is that this doesn't work in all cases (meaning for all Language
Packs).

If you look at the documentation for the ` PublishingWeb.GetPagesListName`
method, you will see that it states the method returns the "_URL_ name of the
pages list" (emphasis mine) -- which, in most cases, is the same as the "name of
the pages list."

Unfortunately, for certain Language Packs the "URL name" is different from the
list "name." For example, in Chinese the URL name is "Pages" but the list name
is "页面".

The correct way to get the **Pages** list is to use the
[`PublishingWeb.GetPagesListId`](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.publishingweb.getpageslistid.aspx)
method instead:

```C#
SPList pages = web.Lists[PublishingWeb.GetPagesListId(web)];
```

It looks like the `PublishingWeb.GetPagesListName()` method really should have
been called `GetPagesUrlName` instead.
