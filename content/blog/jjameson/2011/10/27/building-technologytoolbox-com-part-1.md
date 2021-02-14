---
title: "Why was Subtext selected for TechnologyToolbox.com? (a.k.a. Building TechnologyToolbox.com, part 1)"
date: 2011-10-27T02:53:14+08:00
excerpt: "In my previous post, I mentioned how my new blog is currently powered by Subtext -- or rather my own (slightly modified) version of Subtext 2.5. I also noted that Subtext wasn't my first choice when selecting a blogging solution..."
draft: true
categories: ["Development"]
tags: ["Subtext", "Web Development"]
---

In my previous post, I mentioned how my new blog is currently powered by [Subtext](http://subtextproject.com) -- or rather my own (slightly modified)  version of Subtext 2.5. I also noted that Subtext wasn't my first choice when selecting  a blogging solution for the new TechnologyToolbox.com site.

### BlogEngine.NET or dasBlog?

Actually, I started out using [BlogEngine.NET](http://www.dotnetblogengine.net/).  Initially, I looked at both [dasBlog](http://dasblog.info/) and BlogEngine.NET.  dasBlog made the short list because I've seen it in use on [Scott Hanselman's blog](http://www.hanselman.com/blog/) for a number  of years and he seems quite happy with the performance, stability, and feature set.  He even contributes an occasional bug fix for dasBlog every now and then (e.g. changesets [55621](http://dasblog.codeplex.com/SourceControl/changeset/changes/55621)  and [50527](http://dasblog.codeplex.com/SourceControl/changeset/changes/50527)) which gave me a "warm and fuzzy" feeling during the due diligence process  of selecting a blogging solution.

The problem with dasBlog is that it appears to be "dead" from a development perspective  -- meaning nobody seems to be actively working on it. If you look at the main dasBlog  site ([http://dasblog.info](http://dasblog.info/)) you will see that  it still refers to the March 16th 2009 version as the latest release -- and yet  when you look at the [corresponding CodePlex
project](http://dasblog.codeplex.com), you can see dozens of bug fixes and enhancements have been checked  in since that time. Therefore I turned my attention from dasBlog and started focusing  on BlogEngine.NET instead.

I really liked what I saw in BlogEngine.NET. The solution is very "polished"  (unlike a number of other open source blogging solutions out there), appears to  be very extensible, and -- unlike dasBlog -- is actively being worked on by the  development team. It supports all of the core features that I considered "must haves"  for the TechnologyToolbox.com blog, and it was very easy to install and configure.  It also supports BlogML for importing posts (which was originally a "nice-to-have"  on my list -- although, in retrospect, I would now consider this a "must have" instead).

Consequently, I decided to go with BlogEngine.NET and proceeded to create a utility  to migrate my blog content from [my old
MSDN blog](http://blogs.msdn.com/b/jjameson/). That's when I discovered a showstopper...

After migrating the roughly 300 posts from my MSDN blog (running on the Telligent  Community platform) into my new BlogEngine.NET-based blog, I noticed some significant  performance issues.

Note that the default configuration of BlogEngine.NET does not require a SQL  Server database. Instead blog posts are stored as individual XML files and accessed  through a provider that encapsulates the underlying storage mechanism.

Suspecting the XML storage implementation to be the source of the performance  problem, I spent a few minutes creating a SQL Server database and switching from  the **XmlBlogProvider** to the **[DbBlogProvider](http://blogengine.codeplex.com/wikipage?title=SQLServerBlogProvider)**   -- and subsequently migrated my historical content into the database. I fully expected  this to resolve the scalability issue I discovered shortly before.

Unfortunately it did not.

After spending about a half hour walking through BlogEngine.NET in the debugger,  I discovered that the source of the problem was not in the underlying blog provider  (i.e. **XmlBlogProvider** or **DbBlogProvider**), but  rather in the service layer "above" the provider.

It turns out that BlogEngine.NET caches all of your blog posts in memory -- regardless  of whether you choose to store them in XML files or in a SQL Server database. And  it's not just the post summary information that is cached in memory. Rather it's  the *entire content of each and every post*.

Specifically, in BlogEngine.NET all of the post content is stored in a static  list -- which makes it even worse than if this same approach were implemented using  the ASP.NET caching infrastructure. If the ASP.NET cache were used, then at least  the underlying platform could do its job of effectively managing the memory usage  of the application (assuming the full content of each post was cached individually).

When I discovered this, I did some quick searching on the BlogEngine.NET forums  and found several other people complaining about the same issue. I also found references  to BlogEngine.NET deployments with many more blog posts than I currently have, which  made me wonder how they were able to cope with what I consider to be a fundamental  flaw in the architecture of BlogEngine.NET.

The reason, it turns out, depends entirely on what your deployment architecture  looks like. In other words, are you running BlogEngine.NET on a dedicated server  with multiple gigabytes of RAM? If that's the case, then the approach taken by the  BlogEngine.NET development team isn't really an "architectural flaw" at all -- rather  they are simply taking advantage of the "memory is cheap these days" mantra and  it will take *a lot* of blog posts to consume even 1GB of memory.

However what if, like me, you are running in a shared hosting environment? In  that case, the "memory is cheap" excuse no longer holds true, because the hosting  provider (e.g. WinHost) needs to run many different websites on a single server.  If your application has a large memory footprint, then it is not a good candidate  for shared hosting. [Note that I never saw anything from WinHost regarding how much  memory consumption would be considered a problem. Rather my gut told me that if  I continued with BlogEngine.NET then eventually I would hear from the WinHost support  folks telling me I either need to consume less memory or move to a dedicated hosting  plan instead.]

Thus began my search for another blogging solution.

After some more looking around, I discovered a few other potential candidates:

- [Oxite](http://oxite.codeplex.com/)
- [Orchard](http://orchardproject.net/)
- [Umbraco](http://umbraco.com/)
- [Subtext](http://subtextproject.com)

### Oxite

Note that I didn't spend any significant time looking at Oxite, since CodePlex  clearly states it is no longer being worked on. However, it did lead me to the Orchard  CMS project which looked very promising...at first.

### Orchard CMS

It's worth emphasizing that Orchard is a Content Management System -- not just  a blog engine. In other words, while it includes blogging functionality "out-of-the-box",  it also provides the core infrastructure for many other types of website content.

When I discovered that Orchard is powering the blog for the renowned [John Papas](http://johnpapa.net/), I thought surely this would be the  just the cure for my BlogEngine.NET woes. Consequently, I downloaded Orchard, installed  it on one of my development VMs, and proceeded to setup a test blog with some sample  content from my old MSDN blog.

After my experience with BlogEngine.NET, you can imagine that I was feeling a  little "gun shy" about free, open source applications for blogging. As such I quickly  started looking at the underlying architecture of Orchard, rather than how it appeared  on the surface from the administrator and blog author perspectives.

Like BlogEngine.NET, Orchard appears very "polished" and was a breeze to setup.  Orchard is also very extensible.

Unfortunately, I quickly discovered a couple of issues with Orchard that left  me feeling a little queasy.

The first is what appears to be a rather obvious bug that is described in a question  posted on Stack Overflow:

<cite>How can I set the permissions for blogs in Orchard CMS?</cite>
[http://stackoverflow.com/questions/6941720/how-can-i-set-the-permissions-for-blogs-in-orchard-cms](http://stackoverflow.com/questions/6941720/how-can-i-set-the-permissions-for-blogs-in-orchard-cms)

The second issue is related to the fundamental architecture of Orchard. When  I used SQL Server Profiler to inspect the database calls for one of my sample blog  pages (as described in [one of my previous posts](/blog/jjameson/2010/09/03/analyzing-database-roundtrips-with-sql-server-profiler)), I observed over 40 separate round-trips to the database  just to render a *single* page. [Note that I am going off memory here from  a couple of months ago, so I could be wrong about the number -- but I seem to recall  something in the neighborhood of 42-44 round-trips to SQL Server.]

In my opinion, the Orchard team has made a substantial tradeoff in optimization  (i.e. minimizing the number of database calls required to render a page) in order  to achieve a highly componentized architecture and what they call "UI composition"  (e.g. content item → content type → content part). While I admire the  ultimate goals and flexibility in the Orchard architecture, it seems a little too  "ivory tower" to me and I wonder how Orchard scales in real world deployments.

By comparison, SharePoint typically requires only one or two database round-trips  to render a page (once the site is warmed up, of course). Note that in SharePoint-Land,  a single round-trip to SQL Server often includes multiple stored procedure calls  or T-SQL statements batched together. The key point to understand is that SharePoint  tries to fetch everything it needs from the database to render a single page using  a very small number of round-trips to SQL Server, because the SharePoint team --  like me -- believes the key to scaling massively is to minimize your "off-the-box"  calls.

Based on what I saw in my cursory examination of the Orchard architecture, it  seems like Orchard would require substantial rework in order to batch multiple calls  to the database into significantly fewer round-trips to SQL Server.

Consequently, I moved on to looking at another CMS solution that includes blogging  functionality, namely Umbraco.

### Umbraco

Umbraco is relatively easy to install and configure, but I discovered some issues  when you want to run it in isolation within your site. For example, as I mentioned  in my previous post, I was looking for a blogging solution to serve content from  the **/blog** folder of TechnologyToolbox.com while other areas of  the site are served from a different ASP.NET application.

In this configuration, I encountered an error similar to the one described in  the following issue:

<cite>Error after installing blog </cite>
[http://blog4umbraco.codeplex.com/workitem/6895](http://blog4umbraco.codeplex.com/workitem/6895)

Fortunately, a little digging around led to the corresponding fix:

<cite>xslt assumes Blog is in root </cite>
[http://blog4umbraco.codeplex.com/workitem/6783](http://blog4umbraco.codeplex.com/workitem/6783)

However, the more I looked at Umbraco, the more I felt like it was too complex  for what I was looking for. The more I dug into Umbraco, the more I thought about  how much time I would need to invest in learning yet another CMS solution. Since  SharePoint is one of my core competencies, I honestly don't see myself recommending  Umbraco over SharePoint for the size of companies that I tend to work with.

Right about the time I started looking into customizing the "master page" in  Umbraco, I decided to once again pull the plug.

At this point, I was getting quite frustrated in my search for a replacement  to my original choice of BlogEngine.NET. In fact, I even thought about writing my  own blogging solution (albeit with minimal functionality).

Fortunately, that's when I discovered Subtext.

### Subtext

> If you've ever caught yourself throwing your hands in the air and declaring
> that you're going to write your own blogging engine, then Subtext is for
> you.
>
> - [http://subtextproject.com](http://subtextproject.com)

When I read these words, I got goose bumps. Okay...not really...but they certainly  did resonate with me based on my recent experience.

Fortunately, Subtext was a breeze to install and I don't recall hitting any snags  right away (unlike my experience with Orchard and Umbraco). More importantly, however,  after looking at the underlying architecture of Subtext, and "sniffing" the database  traffic using SQL Server Profiler, I had a much better feeling about it than I did  with any of the other blogging solutions I considered using.

Subtext does a fair amount of caching, but it does so in a way that I consider  to be much "smarter" than BlogEngine.NET -- and the fact that it actually does caching  seems to make it a more logical choice than Orchard -- at least at this point in  time. [That's not meant to bash the Orchard development team. I sincerely hope that  in the near future, they turn their attention to optimizing performance and scalability.]

Subtext is definitely not as "polished" as BlogEngine.NET or Orchard from a UI  perspective (and to be honest, you can tell that [Phil](http://haacked.com/)  doesn't place a great deal of value on the look-and-feel of the blog admin pages).  Nevertheless, the core functionality appears to be very solid.

Are there bugs in Subtext? Sure...there are bugs in *every*piece of software  out there.

Did I encounter any issues with Subtext while implementing it on TechnologyToolbox.com?  Well, yes, as a matter of fact I did.

Fortunately, though, it didn't take very long to get up-to-speed on the inner  workings of Subtext, and I was able to make the necessary tweaks that I needed along  the way.

I'll explain more about those "tweaks" in subsequent posts.

