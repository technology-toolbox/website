---
title: "\"Inheriting\" the Master Page from the Current Site Context in MOSS 2007"
date: 2009-09-20T09:58:00-06:00
description:
  "In my previous post , I showed how you can override the hard-coded
  \"application.master\" in Microsoft Office SharePoint Server (MOSS) 2007
  application pages (e.g. /Library/_layouts/viewlsts.aspx). Note that for custom
  application pages (i.e. those ASP..."
aliases:
  [
    "/blog/jjameson/archive/2009/09/19/inheriting-the-master-page-from-the-current-site-context-in-moss-2007.aspx",
    "/blog/jjameson/archive/2009/09/20/inheriting-the-master-page-from-the-current-site-context-in-moss-2007.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/09/20/inheriting-the-master-page-from-the-current-site-context-in-moss-2007.aspx"
---

In my
[previous post](/blog/jjameson/2009/09/20/overriding-application-master-in-moss-2007),
I showed how you can override the hard-coded "application.master" in Microsoft
Office SharePoint Server (MOSS) 2007 application pages (e.g.
/Library/_layouts/viewlsts.aspx).

Note that for custom application pages (i.e. those ASP.NET pages that you create
to run under the context of a SharePoint site) you don't need a custom
HttpHandler in order to "inherit" the master page from the current `SPWeb`.

All you need to do is simply set the master page during the PreInit phase of the
ASP.NET page lifecycle.

This is precisely what I developed my custom `SharePointPage` base class for:

```C#
/// <summary>
/// Base class for ASP.NET pages that run under the context of a SharePoint
/// site (e.g. /en-US/Search/Library/_layouts/PublicationSummary.aspx).
/// </summary>
/// <remarks>
/// Inheriting from this base class ensures that the correct master page is
/// used (as specified by the current site context).
/// </remarks>
public class SharePointPage : Page
{
    protected override void OnPreInit(
        EventArgs e)
    {
        base.OnPreInit(e);

        SetMasterPageFromCurrentWeb();
    }

    private void SetMasterPageFromCurrentWeb()
    {
        if (SPContext.Current == null)
        {
            throw new InvalidOperationException(
                "This page must execute within a SharePoint site"
                + " (SPContext.Current is null).");
        }

        string masterPageFile = SPContext.Current.Web.CustomMasterUrl;

        Logger.LogDebug(
            CultureInfo.InvariantCulture,
            "Overriding master page with {0}...",
            masterPageFile);

        this.MasterPageFile = masterPageFile;
    }
}
```

To see a real-world example of this in action, simply browse to
[one of the "publication summary" pages](http://www.chem.agilent.com/en-US/Search/Library/_layouts/Agilent/PublicationSummary.aspx?whid=37419&liid=204)
on the Agilent Technologies [LSCA site](http://www.chem.agilent.com/) (try
searching for **6850**, filtering the search results to **Library**, and then
clicking one of the search results). Note that the `PublicationSummary` page
class inherits from the `SharePointPage` base class.

```C#
public partial class PublicationSummary : SharePointPage,
    IView<PrimaryDocumentData.PrimaryDocumentRow>
{
    ...
}
```

For the sake of this post, ignore the `IView` interface. That is used for
something else entirely (i.e. a simple Model-View-Controller framework) that
perhaps one day I'll get around to covering as well.
