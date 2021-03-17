---
title: "Be careful when using the SharePoint PublishingPage.Url property"
date: 2012-02-11T01:17:16-07:00
excerpt:
  "Intellisense isn't helpful when the underlying XML documentation in the code
  is wrong."
aliases: ["/blog/jjameson/archive/2012/02/10/be-careful-when-using-the-sharepoint-publishingpage-url-property.aspx", "/blog/jjameson/archive/2012/02/11/be-careful-when-using-the-sharepoint-publishingpage-url-property.aspx"]
draft: true
categories: ["Development", "SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010"]
---

Intellisense in Visual Studio is only as good as the underlying XML
documentation in the code. Case in point: the
[**PublishingPage.Url**](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.publishingpage.url.aspx)
property in Microsoft Office SharePoint Server 2007 (MOSS 2007) and SharePoint
Server 2010.

Way back in January 2008, a couple of folks pointed out that the
[documentation on MSDN](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.publishingpage.url%28v=office.12%29.aspx)
was wrong. Three years later, you'd think Microsoft would have fixed the
underlying XML documentation in a service pack for MOSS 2007 and certainly would
have corrected the issue in SharePoint 2010. However, you'd be wrong.

For whatever reason, they apparently consider this doc bug too minor to fix. Is
that what we are supposed to think?

I guess a better question is whether or not community feedback added to MSDN is
rigorously reviewed by anyone at Microsoft.

I no longer work at Microsoft, so unfortunately I can't just hop over to
[http://bugcheck](http://bugcheck) to see if anyone on the SharePoint team
actually created a bug to fix this. If they didn't, then shame on them.

As I've said before, no software is perfect. However, software developers should
continuously strive to fix bugs -- even the small, seemingly insignificant ones
like this one. I'm certain that I encountered this issue before, but I got "bit"
by it again this morning while working on a code sample for another post.

Here's a simple repro I whipped up this morning -- with hopes it will help me
remember to only use the **PublishingPage.Url** property when I need the
*site-relative* URL of the page:

```
using System;
using Microsoft.SharePoint;
using Microsoft.SharePoint.Publishing;

namespace PublishingPageUrlDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            Uri siteUrl;

            if (args.Length != 1)
            {
                throw new ArgumentException(
                    "Usage: PublishingPageUrlDemo {site URL}");
            }

            siteUrl = new Uri(args[0]);

            using (SPSite site = new SPSite(siteUrl.AbsoluteUri))
            {
                using (SPWeb web = site.OpenWeb(siteUrl.PathAndQuery))
                {
                    PublishingWeb pubWeb = PublishingWeb.GetPublishingWeb(web);

                    PublishingPage page = pubWeb.GetPublishingPage(
                        pubWeb.PagesListName + "/default.aspx");

                    // According to Intellisense (and the current MSDN
                    // documentation), PublishingPage.Url gets the
                    // server-relative URL, but the following outputs the
                    // site-relative URL ("Pages/default.aspx")
                    Console.WriteLine("page.Url: {0}", page.Url);
                }
            }
        }
    }
}
```
