---
title: "Always Enable Disk-Based Caching in SharePoint Server 2010"
date: 2010-11-16T08:43:00+08:00
excerpt: "In March, 2009, I wrote a post that explains why I always recommend enabling disk-based caching in Microsoft Office SharePoint Server (MOSS) 2007. 
 This morning a Microsoft PFE (Premier Field Engineer) reached out to me after he came across my blog..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:  
>   
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/11/16/always-enable-disk-based-caching-in-sharepoint-server-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/11/16/always-enable-disk-based-caching-in-sharepoint-server-2010.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.


In March, 2009, I wrote a [post](/blog/jjameson/2009/03/27/always-enable-disk-based-caching-in-moss-2007) that explains why **I always recommend enabling disk-based caching** in Microsoft Office SharePoint Server (MOSS) 2007.

This morning a Microsoft PFE (Premier Field Engineer) reached out to me after he came across my blog post while investigating some issues at a customer site. He said that he was at some "government agency" but that's all he would say -- and probably all I want to know ;-)

Anyway, he mentioned that his 10-minute SQL Server Profiler trace showed something like 56,000 calls to the `proc_FetchDocForHttpGet` stored procedure (which I suspect is how he discovered my earlier post, since it appears as the [3rd search result on bing](http://www.bing.com/search?q=proc_fetchdocforhttpget) and the [7th search result on Google](http://www.google.com/#hl=en&amp;q=proc_fetchdocforhttpget)).

We talked for a little bit about the effects of enabling disk-based caching, since the customer's SharePoint administrator was a little reluctant to enable it without some "data" to support it. [Note that there are *some* cons, which is something I really should have covered in a blog post by now. I'll try to get to that soon, I promise. Refer to [my next post](/blog/jjameson/2010/11/16/avoid-issues-with-caching-by-using-quot-theme-versions-quot) for more on this.]

I told the PFE to do some quick analysis on his Profiler trace to see how many of those `proc_FetchDocForHttpGet` calls were for resource files (e.g. cascading style sheets, images, etc.) -- in other words, items ideally suited for caching (as compared to "real" documents, such as PDF or Word docs).

I also told them that I suspected they will see a dramatic drop in the load on their SQL Server after enabling disk-based caching. My guess is that their SQL environment is pretty beefy and therefore able to support the load (circa 1,000 concurrent users on the intranet site) except at peak times -- or perhaps, their load has grown significantly in recent weeks -- which prompted pulling in the PFE to troubleshoot.

This incident got me thinking about whether the story has changed for SharePoint Server 2010. For those of you that have already read more of this post than you would have liked, the short answer is "yes, it's definitely better in the newer version...but no, my recommendation hasn't changed for SharePoint Server 2010." Want to know more? Well, then read on, my friends...

Here is why **I always recommend enabling disk-based caching in SharePoint Server 2010**:

I fired up one of the "vanilla" SharePoint Server 2010 VMs that I built a while ago and started up a SQL Server Profiler trace using the settings detailed in the previously mentioned post.

I then browsed to the home page of one of my sites (based on the out-of-the-box **Publishing Portal** template). As described in my previous post, I then closed the browser, cleared the Profiler trace, and then started Internet Explorer and browsed to the site again.

The good news is that Profiler shows a nominal *five* database roundtrips required for SharePoint Server 2010 to render the home page for the site. Keep in mind that I was using a "vanilla" site with no customizations, so your actual number may vary, depending on your environment.

While five database calls certainly *seems* reasonable for rendering a page request, the bad news is that the last three database calls resemble the following:



    exec proc_FetchDocForHttpGet @DocSiteId='...',@DocDirName=N'...',
        @DocLeafName=N'controls.css', ...
    
    exec proc_FetchDocForHttpGet @DocSiteId='...',@DocDirName=N'...',
        @DocLeafName=N'page-layouts-21.css', ...
    
    exec proc_FetchDocForHttpGet @DocSiteId='...',@DocDirName=N'...',
        @DocLeafName=N'home.jpg', ...



Seriously...what is the likelihood the CSS files or image have changed since the last request for the home page? Zero. [Okay, maybe not absolute 0 -- but probably something like 6.28 x 10<sup>-9</sup>  ;-) ]

After changing the `<BlobCache>` **enabled** attribute from "false" to "true" in my Web.config file, I confirmed that subsequent requests for the home page only require two database roundtrips. Woohoo!

So, why then should we burden SQL Server with these extraneous requests? The answer, of course, is that we shouldn't (at least not in Production and Test environments). [As I noted in my previous post, you certainly don't have to enable BLOB caching in your development environments (I certainly don't).]

However, this leads to an important question: What if my CSS files or images *do* change and I need to ensure that users get the updated versions? Well, as I hinted earlier in this post, that's a topic for a later time.

Don't worry, you won't have to wait very long. [Refer to [my next post](/blog/jjameson/2010/11/16/avoid-issues-with-caching-by-using-quot-theme-versions-quot) for an easy way of ensuring users do not view your site using "stale" CSS files and images as a result of caching.]

Lastly, I should note that MOSS 2007 requires 15 database roundtrips to render the home page for a site based on the Publishing Portal template (yes, this is even after the site is "warmed" up). If you look at the calls to `proc_FetchDocForHttpGet`, you'll find a number of additional CSS files and images.

