---
title: "Narrowing Search Results to a Specific Site (e.g. My Blog)"
date: 2010-04-05T05:58:00+08:00
excerpt: "A colleague asked me today if there was a way to search my blog for something specific. 
 In my response, I pointed out that you can narrow your search results from Bing to a specific site. 
 For example, suppose you were looking for a blog post that..."
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "Simplify", "MOSS 2007"]
---

> **Note**
> 
> 
> 		This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/05/narrowing-search-results-to-a-specific-site-e-g-my-blog.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/05/narrowing-search-results-to-a-specific-site-e-g-my-blog.aspx)
> 
> 
> Since
> 		[I no longer work for Microsoft](/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx), I have copied it here in case that 
> 		blog ever goes away.


A colleague asked me today if there was a way to search my blog for something specific.

In my response, I pointed out that you can narrow your search results from Bing to a specific site.

For example, suppose you were looking for a blog post that I wrote about faceted search. All you need to do is specify the appropriate keywords as well as the corresponding property filter:


> faceted search site:blogs.msdn.com/jjameson


If you browse to [www.bing.com](http://www.bing.com/) and then type this into the search box, you will find that it generates a search results URL similar to the following:


> [http://www.bing.com/search?q=faceted+search+site%3Ablogs.msdn.com%2Fjjameson](http://www.bing.com/search?q=faceted+search+site%3Ablogs.msdn.com%2Fjjameson)


It wasn't all that long ago that you could only specify top-level domain names when searching by a specific site. Fortunately, those days are gone and consequently narrowly-focused results are really easy, even when using the major search engines on the Internet.

Note that you can filter results in a similar manner with several other search engines, including Microsoft Office SharePoint Server (MOSS) 2007. In many cases, you specify the URL of the site in the search query terms. However, that's not always the only way to do it.

In MOSS 2007, you can specify a *contextual scope URL* (a.k.a. the "u" query string parameter).

For example, if you browse to the home page for KPMG Global ([http://www.kpmg.com/global/en](http://www.kpmg.com/global/en)), and search for "tax audit", you will find it generates a search results URL similar to the following:


> [http://www.kpmg.com/Global/en/Search/Pages/Results.aspx?k=tax+audit&u=http%3a%2f%2fwww.kpmg.com%2fglobal%2fen](http://www.kpmg.com/Global/en/Search/Pages/Results.aspx?k=tax+audit&amp;u=http%3a%2f%2fwww.kpmg.com%2fglobal%2fen)


While not quite as easy to read due to URL encoding of the parameter, the contextual scope URL in MOSS 2007 makes it relatively simple to implement search functionality like the "Search this site" and "Search all sites" features on the KPMG site.

