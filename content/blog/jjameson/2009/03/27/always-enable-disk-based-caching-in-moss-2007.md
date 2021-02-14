---
title: "Always Enable Disk-Based Caching in MOSS 2007"
date: 2009-03-27T02:36:00+08:00
excerpt: "For reasons completely unknown to me, the SharePoint team decided to ship Microsoft Office SharePoint Server (MOSS) 2007 with disk-based caching (a.k.a. blob caching ) disabled. If you are not familiar with disk-based caching, here is a blurb from Microsoft..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/27/always-enable-disk-based-caching-in-moss-2007.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/27/always-enable-disk-based-caching-in-moss-2007.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

For reasons completely unknown to me, the SharePoint team decided to ship Microsoft Office SharePoint Server (MOSS) 2007 with disk-based caching (a.k.a. *blob caching*) disabled. If you are not familiar with disk-based caching, here is a blurb from [Microsoft Office Online](http://office.microsoft.com/en-us/sharepointserver/HA101762841033.aspx):

> Disk-based caching is one way in which you can achieve faster processing of content stored in a Web application database. If your Web application contains large files such as images and multimedia files, enabling disk-based caching improves page delivery time because the cache stores files on the front-end Web server, thus reducing database traffic.

I also highly recommend becoming familiar with [the right way to clear the blob cache](http://msdn.microsoft.com/en-us/library/aa604896.aspx). [Hint: The wrong way to flush the cache is to simply delete the content within the location specified in the `<BlobCache>` configuration element. I can tell you from a personal experience a couple of years ago that this is a very bad idea, indeed -- especially on a shared environment ;-) ]

The problem is that statements like the one above make it sound like the primary benefit of disk-based caching is to avoid repeatedly transferring "large files" between the backend SQL Server and the frontend Web servers. While this is certainly *one* of the benefits, the primary reason why **I always recommend enabling disk-based caching** is that you vastly reduce the number of database roundtrips for each and every page request on your site.

Consider a typical Web page on an Internet site served by MOSS 2007, containing some textual content but also a significant number of "resources" such as images and CSS files. With disk-based caching disabled (i.e. the default), would it surprise you that 100 roundtrips to SQL Server (or more, depending on your content) are required to render a single page?

If you haven't personally seen this with your own eyes, I strongly encourage you to take a few minutes to fire up SQL Server Profiler (preferably on your local VM so you can isolate individual page requests) and start a trace with the following settings:

- **General**
  - Trace name: SharePoint Content *(or whatever you want to call it)*
- **Events Selection**
  - Stored Procedures
    - RPC:Completed
  - TSQL
    - SQL:BatchCompleted
  - Column Filters
    - DatabaseName Like {your content database} *(e.g. "WSS\_Content\_Fabrikam")*
    - NTUserName Like {your app pool service account) *(e.g. "svc-fabrikam-dev")*
    - TextData Not Like exec sp\_reset\_connection

These settings will filter out lots of "noise" -- such as requests by the SharePoint Timer jobs that continually run in the background -- and show you just the SQL traffic necessary to render pages on the site.

With the trace running, browse to the home page of your site, and then go back and clear the trace window. Next, close your browser, restart it, and then browse once again to the home page of your site. Now look at the number of events captured in SQL Server Profiler. Astonishing, isn't it? Also keep in mind that this is for a "warmed up" SharePoint site. If you really want to be in shock, try clearing your trace, resetting your SharePoint application pool, and then browsing to the home page of your site to see the number of SQL requests from a "cold start."

[Note: The reason I instructed you to restart your Web browser rather than simply clicking the **Refresh** button is to avoid forcing the browser to check for updated GIFs, JPEGs, CSS files, etc. For more on this, refer to the [excellent article about HTTP performance and caching](http://msdn.microsoft.com/en-us/library/bb250442%28VS.85%29.aspx) by Eric Lawrence. I highly recommend reading this if you haven't already. It also introduces you to [Fiddler](http://www.fiddlertool.com) -- an essential tool that I mentioned in a [previous post](/blog/jjameson/2008/06/27/fiddler-wpad-slowperformance).]

If you scan through the SQL events in the trace, you will see lots of calls to the `proc_FetchDocForHttpGet` stored procedure, including one for each image (and CSS file) stored in the **Style Library**.

Recall what I said earlier about statements that suggest enabling disk-based caching to avoid transferring "large files" from SQL Server. What is important to realize about the numerous `proc_FetchDocForHttpGet` sproc calls is that SharePoint is not actually returning the image or CSS file on every call. Assuming the resources are ghosted -- er, I mean *uncustomized*-- then the files will actually be served up from the file system on the frontend Web servers. However, SharePoint first has to check whether each file is, in fact, ghosted (uncustomized).

Note that if you look at a Fiddler trace with disk-based caching disabled, you will find that the HTTP requests for images and CSS files actually return a 304 result (i.e. not modified), thus telling the browser that the files have not been changed since they were previously downloaded. In other words, while the Fiddler trace makes it seem like these numerous images and CSS files are not a performance issue, the evidence provided by the SQL Server Profiler trace is cause for alarm (at least in my opinion). Having each page request "hammer" SQL Server like this is certainly going to limit your scalability.

Once you enable disk-based caching, two important things happen:

1. An HTTP header (e.g. `Cache-Control: public, max-age=86400`) is added to the response for each file type specified using the `path` attribute of the `BlobCache` element. (Note that "\_layouts" items are cached differently than, for example, items in the **Style Library**). This explicit `Cache-Control` header greatly reduces the number of HTTP requests for clients that have already downloaded the various resource files (by eliminating the 304's mentioned before).
2. For clients that haven't yet downloaded the various resources, SharePoint does not have to call the `proc_FetchDocForHttpGet` sproc to determine whether each file has been unghosted (customized). [Note that when you enable disk-based caching, a background process polls the database to check for updates -- as observed in SQL Server Profiler.]

I encourage you to repeat the SQL Server Profiler trace after setting `enabled="true"` in the `BlobCache` element of your Web.config files and browsing to the home page of your site. You will see the number of SQL roundtrips is greatly reduced.

So, unless you just have lots of excess capacity on your SQL Server cluster (and therefore don't mind lots of extraneous database roundtrips), I hope you don't consider going live without enabling disk-based caching.

> **Update (2009-03-28)**
>
> It occurred to me this morning that my previous statement about always enabling disk-based caching could be misconstrued. If you don't want to enable disk-based caching on your local VM to expedite your development activities (e.g. to avoid having to repeatedly clear your browser cache to see changes to CSS files), that's perfectly fine. Just make sure you enable disk-based caching on your Test and Production environments.

