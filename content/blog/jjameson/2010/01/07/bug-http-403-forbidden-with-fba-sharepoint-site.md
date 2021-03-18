---
title: "Bug: HTTP 403 (Forbidden) with FBA SharePoint Site"
date: 2010-01-07T07:29:00-07:00
excerpt:
  We recently encountered a bug when trying to access a SharePoint site
  configured with Forms-Based Authentication from Internet Explorer. The root
  site is restricted to authenticated users, whereas the /Public site is
  configured to allow anonymous users...
aliases:
  [
    "/blog/jjameson/archive/2010/01/06/bug-http-403-forbidden-with-fba-sharepoint-site.aspx",
    "/blog/jjameson/archive/2010/01/07/bug-http-403-forbidden-with-fba-sharepoint-site.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/01/07/bug-http-403-forbidden-with-fba-sharepoint-site.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/01/07/bug-http-403-forbidden-with-fba-sharepoint-site.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/01/07/bug-http-403-forbidden-with-fba-sharepoint-site.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

We recently encountered a bug when trying to access a SharePoint site configured
with Forms-Based Authentication from Internet Explorer.

The root site is restricted to authenticated users, whereas the **/Public** site
is configured to allow anonymous users. In other words, when users attempt to
browse to the root site (/), they should automatically be redirected to
/Public/Pages/default.aspx (which contains a login form).

Everything was working great for awhile, but then we discovered that certain
clients would encounter HTTP 403 (Forbidden) errors when attempting to browse to
the root site before logging in. In other words, instead of redirecting to the
/Public site as expected, IE would simply display the HTTP 403 error page
(which, at first glance, actually makes it seem like the site is down).

We observed the site worked just fine from the same clients when using Firefox,
and it only occurred on a couple of laptops when using IE8.

We also noticed that we could workaround the problem by explicitly browsing to
the /Public site instead of relying on the redirect -- hence we "punted" the
issue for a short period, hoping that the issue could only be reproduced on our
Microsoft laptops.

At first we suspected the problem was due to specific IE options, but resetting
the IE configuration didn't resolve the issue.

Next, we suspected a specific version of IE8, since we noticed it happened on
Windows 7 but not on Windows Server 2008.

However, then we discovered that it worked on one laptop but not another -- even
though both laptops were running Windows 7 (x64) and had the exact same version
of Internet Explorer.

Fortunately, Ranjiv Sharma -- one of my colleagues on the project -- discovered
the following blog post:

{{< reference
title="403 Forbidden and Forms Authentication with MOSS. 2009-05-21."
linkHref="http://vettekerry.wordpress.com/2009/05/21/403-forbidden-and-forms-authentication-with-moss/" >}}

It turns out the problem is caused by the Microsoft Office Live Add-in 1.3. Once
you remove the add-in, close all instances of Internet Explorer, and restart IE,
the seamless redirect works as expected.

> **Update (2010-01-15)**
>
> This is a known bug:
>
> **You cannot view a forms-based authentication Windows SharePoint Services 3.0
> site if you have Office Live Update 1.2 for Microsoft Office Live Workspace
> installed
> **
> [http://support.microsoft.com/kb/972535](http://support.microsoft.com/kb/972535)
>
> According to the KB article, it is fixed in the June 2009 Cumulative Update
> (CU). From my experience, it also appears to be fixed by installing the 1.4 of
> the Microsoft Office Live Add-in.
>
> Note that I tried reinstalling the Office Live Add-in on my VM to try to repro
> the error again prior to installing the June 2009 CU. Unfortunately, I
> couldn't find the 1.3 version available for download anymore from
> microsoft.com (the download page is no longer available). When I downloaded
> and installed the 1.4 version, I was unable to repro the error when browsing
> locally from my VM.
>
> I subsequently downloaded the June 2009 CU and proceeded to install it on my
> local VM (since I could still test with the 1.3 version of the Office Live
> Add-in from my laptop). Unfortunately, I could only install the WSS part of
> this CU -- not the MOSS part. (The MOSS patch is giving me the dreaded
> "expected version of the product was not found on the system" error.)
>
> However, even with just the WSS portion of the June 2009 CU, I am no longer
> seeing the error when browsing to my local VM from my laptop.
