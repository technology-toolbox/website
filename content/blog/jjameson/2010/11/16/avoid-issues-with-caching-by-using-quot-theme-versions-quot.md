---
title: "Avoid Issues with Caching by Using \"Theme Versions\""
date: 2010-11-16T09:45:00+08:00
excerpt: "In a previous post discussing Web standards design, I mentioned how I like to use \"Theme\" folders to organize CSS files and related images that define a specific look-and-feel for a site. 
 For example, suppose we are tasked with building the Internet..."
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Web Development", "SharePoint 2010"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/11/16/avoid-issues-with-caching-by-using-quot-theme-versions-quot.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/11/16/avoid-issues-with-caching-by-using-quot-theme-versions-quot.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In a previous [post](/blog/jjameson/2010/01/30/web-standards-design-with-moss-2007-part-1) discussing Web standards design, I mentioned how I like to use "Theme" folders to organize CSS files and related images that define a specific look-and-feel for a site.

For example, suppose we are tasked with building the Internet site for Fabrikam Technologies and we are leveraging SharePoint Server (2007 or 2010 doesn't matter -- the concepts I'll cover in this post apply to both). Note that for SharePoint solutions, we want to store CSS files (and, ideally, any images referenced in the CSS files) under the **Style Library** for the site collection.

Suppose we choose to use the concept of a "Theme" folder as well as the [960 Grid System](http://960.gs) (as described in my previous post). Consequently, we end up with something like this:

- Style Library
  - Fabrikam
    - Themes
      - Theme1
        - 960.css
        - Fabrikam-Basic.css
        - Fabrikam-Main.css

As mentioned in my previous post, if we want to completely overhaul the look-and-feel of the Fabrikam site, all we need to do is create a new "Theme2" folder and add the new CSS files and images.

However, what if instead of completely overhauling the CSS for the site, we need to simply make a change to an existing CSS rule or add some new rules?

The answer is pretty simple...you just update the CSS file, rebuild your WSP that deploys your CSS, and then run "stsadm.exe -o upgradesolution ..." to deploy the updated file(s). Well, almost...

What if we previously enabled disk-based caching for the Web application? (And, of course, [you always want to enable disk-based caching in SharePoint](/blog/jjameson/2010/11/16/always-enable-disk-based-caching-in-sharepoint-server-2010), right?!)

In that case, things aren't quite so easy.

First, when you enable disk-based caching, SharePoint will (thankfully) start adding a `Cache-Control` header in the HTTP response for each file type specified using the `path` attribute of the `<BlobCache>` element. Consequently, browsers (and perhaps one or more proxy servers along the way) will cache local copies of the files.

Second, the BlobCache folder on the file system will have an obsolete version of the CSS file. Sure, you can clear the blob cache, but that process is painful -- er, I mean, *tedious*-- especially if you have a number of front-end Web servers in your farm.

To avoid these issues with caching, I simply increment the "version" of the Theme folder whenever a new CSS file needs to be deployed to Production. The "version" that I'm referring to is really just part of the folder name.

For example, suppose we've previously deployed the folders and files shown above to Production and now we need to add some new CSS rules to Fabrikam-Main.css. To ensure the "stale" CSS file is no longer retrieved from cache (either the local browser cache, a proxy server cache, or the SharePoint BLOB cache), we simply need to rename the **Theme1** folder to **Theme1.1**:

- Style Library
  - Fabrikam
    - Themes
      - Theme1.1
        - 960.css
        - Fabrikam-Basic.css
        - Fabrikam-Main.css

This rename is done in the source control system (not directly in the Style Library within SharePoint) and includes updating the custom Fabrikam master page(s) to refer to the **Theme1.1** folder. When the new WSP is deployed, the Theme1.1 folder is created in the Style Library, the master page is updated, and users subsequently see the site rendered with the new CSS rules.

Note that this rename is done at most once per Sprint (i.e. release to Production) -- it certainly isn't done each and every time a developer touches one of the CSS files.

It is also worth noting that renaming the Theme folder will cause *all* files within the folder to be downloaded by clients the next time they connect to the site. Consequently, a much better alternative to the structure shown above is:

- Style Library
  - Fabrikam
    - Themes
      - 960.css
      - Theme1.1
        - Fabrikam-Basic.css
        - Fabrikam-Main.css

By moving the 960.css file up one level, we can avoid forcing users to download the same file each time the Theme folder is incremented.

This, of course, assumes that the CSS file provided by the 960 Grid System -- unlike Fabrikam-Main.css -- is fairly "baked" (meaning it doesn't change). Even if it did, you could certainly rename just that one file to something like 960\_v2.css (and update the corresponding references to the file, of course).

Obviously, you could alternatively choose to rename each individual CSS file once per release -- in which case, you are essentially doing manually what the [**SPUtility.MakeBrowserCacheSafeLayoutsUrl**](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.utilities.sputility.makebrowsercachesafelayoutsurl.aspx) method does for you -- but honestly, incrementing the entire "Theme" version seems like a very reasonable compromise.

When I first started looking into this issue, I thought about writing some code that essentially performed the same function as the **MakeBrowserCacheSafeLayoutsUrl** method -- except on files stored in the Style Library, instead of the layouts folder on the file system. However, I decided the "Theme version" approach was much simpler and sufficient to solve the fundamental problem with caching.

The only caveat is that this leads to obsolete folders in your Style Library, but you can easily clean these up if it really bothers you to store a few megabytes of files in your database that will never be used again -- or, better yet, clean these up by activating a custom "upgrade feature" on your site. The concept of an "upgrade feature" is something I started using recently and it is proving to be very powerful -- especially when you deploy to Production every 6 weeks or so. Perhaps I'll talk more about that in a separate post.

