---
title: "Avoid Extraneous Database Roundtrips in SharePoint PublishingPageCollection.Item Property"
date: 2010-09-12T00:38:00+08:00
excerpt: "In my previous post , I explained how I analyze database roundtrips using SQL Server Profiler in order to identify potential performance issues. 
 While working on some proof-of-concept code for my current project, I found the PublishingPageCollection..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/09/12/avoid-extraneous-database-roundtrips-in-sharepoint-publishingpagecollection-item-property.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/09/12/avoid-extraneous-database-roundtrips-in-sharepoint-publishingpagecollection-item-property.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.


In [my previous post](/blog/jjameson/2010/09/03/analyzing-database-roundtrips-with-sql-server-profiler), I explained how I analyze database roundtrips using SQL Server Profiler in order to identify potential performance issues.

While working on some proof-of-concept code for my current project, I found the [PublishingPageCollection.Item(String)](http://msdn.microsoft.com/en-us/library/ms543758%28v=office.12%29.aspx) property in Microsoft Office SharePoint Server 2007 can result in a large number of extraneous database roundtrips. If you're not familiar with this property, it is simply used to to retrieve a page by its Web-relative URL.

First, a little more about the scenario...

Imagine that you have a SharePoint site containing numerous Web pages and you need to export that content to create a nicely formatted document suitable for printing (e.g. a PDF). The order of the Web pages can easily be changed by content authors simply by modifying the navigation of the SharePoint site. In other words, on the **Site Navigation Settings** page, the **Show pages **checkbox is checked and the list of items in the **Navigation Editing and Sorting **section can be shifted around as necessary (using the **Move up **and **Move down **links).

When somebody wants to export the Web content to a PDF, we need to write a little bit of code to extract the HTML content of each page, aggregate the content (i.e. concatenate the various sections into a single HTML document), and then run it through an HTML-PDF converter.

For my initial POC, I whipped up a simple console application that starts by getting the list of pages within an **SPWeb** based on the navigation order. Note that the list of pages is simply an instance of `List<string>` where each item is the Web-relative URL of a page within the site (since this must be unique for each page within a site). Then it uses a simple `foreach` loop to append the content of each page to the "aggregate" document (via a generic **[TextWriter](http://msdn.microsoft.com/en-us/library/system.io.textwriter.aspx)** object).

Since I expect to aggregate all of the pages within the site, I use the **[PublishingWeb.GetPublishingPages()](http://msdn.microsoft.com/en-us/library/ms493244%28v=office.12%29.aspx)** method to return all pages for the specified site. Then, in the `foreach` loop, I fetch each particular page from the collection using the indexer (i.e. the **Item** property) specifying the URL of the page.

Here is the original code I started out with:



```
private static void AppendPages(
            SPWeb web,
            TextWriter writer)
        {
            Debug.Assert(web != null);
            Debug.Assert(writer != null);

            List<string> pageUrls = GetSitePagesInNavigationOrder(web);

            PublishingWeb pubWeb = PublishingWeb.GetPublishingWeb(web);

            PublishingPageCollection pages = pubWeb.GetPublishingPages();

            foreach (string pageUrl in pageUrls)
            {
                PublishingPage page = pages[pageUrl];
                AppendPage(writer, page);
            }
        }
```



After getting the basic implementation working, I fired up SQL Server Profiler in order to understand what kind of load this feature would put on SharePoint. In other words, I wanted to know how many database roundtrips were necessary in order to retrieve all of the pages within a site and subsequently concatenate them into a single HTML document.

After configuring a trace in SQL Server Profiler (as described in my previous post), I started the trace and ran my console application. Based on my sample site, 432 database roundtrips were required based on my initial approach. It's important to note that my sample site contains 97 pages, but nevertheless this number of database roundtrips still seemed very high. After all, wasn't the whole point of using the  **[PublishingWeb.GetPublishingPages()](http://msdn.microsoft.com/en-us/library/ms493244%28v=office.12%29.aspx)** method to get all of the pages at once?

Upon closer inspection, I found that each call to the [PublishingPageCollection.Item(String)](http://msdn.microsoft.com/en-us/library/ms543758%28v=office.12%29.aspx) property resulted in several database calls. Specifically, the following statement results in **four **calls to SQL Server:



```
PublishingPage page = pages[pageUrl];
```



Consequently, I refactored the code a little to see if I could significantly reduce the 432 database roundtrips.

I found that by replacing the **PublishingPageCollection **indexer with my own (albeit crude) indexer, I could eliminate roughly 200 database roundtrips (from 432 to 244):



```
private static PublishingPage GetPublishingPage2(
            PublishingPageCollection pages,
            string pageUrl)
        {
            Debug.Assert(pages != null);
            Debug.Assert(string.IsNullOrEmpty(pageUrl) == false);

            foreach (PublishingPage page in pages)
            {
                if (page.Url == pageUrl)
                {
                    return page;
                }
            }

            throw new ArgumentException(
                "The specified page was not found in the collection.",
                "pageUrl");
        }
```



I'll be the first to admit that this code certainly seems a little "wonky." After all, performing a [linear search](http://en.wikipedia.org/wiki/Linear_search) on a collection in order to find a specific item certainly *seems* less efficient. However, I'll trust what I see with my own eyes in SQL Server Profiler over "gut feel" seven days a week and twice on Sunday.

By the way, if 244 database roundtrips still seems excessive to you, I have to say I agree. Keep in mind that the SharePoint API does a fair amount of "lazy loading" -- often in places you might not expect, such as accessing a simple property.

For example, accessing the **[PublishingPage.Layout](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.publishingpage.layout.aspx)** property, as shown below, actually results in two database roundtrips:



```
const string expectedPageLayout = "PageFromDocLayout.aspx";

            if (page.Layout.Name != expectedPageLayout)
            {
```



```
...
            }
```



Personally, I'm not a big fan of implicit lazy loading like this in public APIs since it makes it all too easy for developers (like me) to write inefficient code. At the very least, I'd much rather see anything that performs any substantial amount of work (e.g. making a database call) implemented as a method instead of a property.

If I eliminate the code that validates the page layout matches the expected value, the number of database roundtrips is reduced to 53.

Here is the complete code sample from my proof-of-concept (in case you want to walk through it on your own). You simply need to create a new C# console application in Visual Studio, add a few references (**Microsoft.SharePoint**, **Microsoft.SharePoint.Publishing**, and** Microsoft.SharePoint.Security **), and then call **PublishingDemo.Execute **in the main class (after substituting the URLs for your environment, of course).



```
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Globalization;
using System.IO;
using System.Text;

using Microsoft.SharePoint;
using Microsoft.SharePoint.Navigation;
using Microsoft.SharePoint.Publishing;

namespace MyConsoleApplication
{
    static class PublishingDemo
    {
        static Guid publishingPageContentFieldId =
            new Guid("{f55c4d88-1f2e-4ad9-aaa8-819af4ee7ee8}");

        public static void Execute()
        {
            const string siteUrl = "http://foobar";
            const string webUrl = "/Branch-Management/Post-Orders/Acme";
            const string documentTitle = "Post Orders";

            StringBuilder buffer = new StringBuilder();

            using (StringWriter writer = new StringWriter(
                buffer,
                CultureInfo.InvariantCulture))
            {
                using (SPSite site = new SPSite(siteUrl))
                {
                    using (SPWeb web = site.OpenWeb(webUrl))
                    {
                        AggregateSitePages(web, writer, documentTitle);
                    }
                }
            }

            string aggregatedContent = buffer.ToString();

            Console.WriteLine(aggregatedContent);
        }

        private static void AggregateSitePages(
            SPWeb web,
            TextWriter writer,
            string documentTitle)
        {
            Debug.Assert(web != null);
            Debug.Assert(writer != null);

            writer.WriteLine(
@"<!DOCTYPE html PUBLIC '-//W3C//DTD HTML 4.01 Transitional//EN'
    'http://www.w3.org/TR/html4/loose.dtd'>");

            writer.Write(
@"<html>
    <head>
        <meta content='en-us' http-equiv='Content-Language' />
        <meta content='text/html; charset=utf-8' http-equiv='Content-Type' />
        <title>");

            writer.Write(documentTitle);
            writer.WriteLine(
@"</title>
    </head>
    <body>");

            AppendPages(web, writer);

            writer.WriteLine(
@"    </body>
</html>");

        }

        private static void AppendPage(
            TextWriter writer,
            PublishingPage page)
        {
            Debug.Assert(writer != null);
            Debug.Assert(page != null);

            const string expectedPageLayout = "PageFromDocLayout.aspx";

            if (page.Layout.Name != expectedPageLayout)
            {
                string message = string.Format(
                    CultureInfo.InvariantCulture,
                    "The page ({0}/{1}) does not have the expected layout"
                        + " ({2}).",
                    page.PublishingWeb.Url,
                    page.Url,
                    expectedPageLayout);

                throw new InvalidOperationException(message);
            }

            string pageContent = (string)page.ListItem[
                publishingPageContentFieldId];

            writer.Write("<h1>");
            writer.Write(page.Title);
            writer.WriteLine("</h1>");

            writer.WriteLine(pageContent);
        }

        private static void AppendPages(
            SPWeb web,
            TextWriter writer)
        {
            Debug.Assert(web != null);
            Debug.Assert(writer != null);

            List<string> pageUrls = GetSitePagesInNavigationOrder(web);

            PublishingWeb pubWeb = PublishingWeb.GetPublishingWeb(web);

            PublishingPageCollection pages = pubWeb.GetPublishingPages();

            foreach (string pageUrl in pageUrls)
            {
                //PublishingPage page = pages[pageUrl];
                PublishingPage page = GetPublishingPage(pages, pageUrl);
                //PublishingPage page = GetPublishingPage2(pages, pageUrl);

                AppendPage(writer, page);
            }
        }

        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Performance",
            "CA1811:AvoidUncalledPrivateCode")]
        private static PublishingPage GetPublishingPage(
            PublishingPageCollection pages,
            string pageUrl)
        {
            Debug.Assert(pages != null);
            Debug.Assert(string.IsNullOrEmpty(pageUrl) == false);

            PublishingPage page = pages[pageUrl];

            if (page == null)
            {
                throw new ArgumentException(
                    "The specified page was not found in the collection.",
                    "pageUrl");
            }

            return page;
        }

        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Performance",
            "CA1811:AvoidUncalledPrivateCode")]
        private static PublishingPage GetPublishingPage2(
            PublishingPageCollection pages,
            string pageUrl)
        {
            Debug.Assert(pages != null);
            Debug.Assert(string.IsNullOrEmpty(pageUrl) == false);

            foreach (PublishingPage page in pages)
            {
                if (page.Url == pageUrl)
                {
                    return page;
                }
            }

            throw new ArgumentException(
                "The specified page was not found in the collection.",
                "pageUrl");
        }

        private static List<string> GetSitePagesInNavigationOrder(
            SPWeb web)
        {
            Debug.Assert(web != null);

            List<string> pageUrls = new List<string>();

            foreach (SPNavigationNode node in web.Navigation.QuickLaunch)
            {
                if (node.Url.StartsWith(
                    web.ServerRelativeUrl,
                    StringComparison.OrdinalIgnoreCase) == false)
                {
                    string message = string.Format(
                        CultureInfo.InvariantCulture,
                        "Invalid navigation node ({0}). Navigation for a"
                            + " Post Orders site can only contain pages within"
                            + " the site.",
                        node.Title);

                    throw new InvalidOperationException(message);
                }

                string pageUrl = node.Url.Substring(
                    web.ServerRelativeUrl.Length + 1);

                pageUrls.Add(pageUrl);
            }

            return pageUrls;
        }
    }
}
```

