---
title: "Supportability Concerns with Custom HTTP Modules in SharePoint"
date: 2009-11-06T22:34:00+08:00
excerpt: "I am sure that I'm not the first one to tell you this, but you can't believe everything you read on the Internet these days ;-) 
 Case in point...I've seen a number of sources claim that custom HTTP modules are not supported in Microsoft Office SharePoint..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
> 
>       This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/11/07/supportability-concerns-with-custom-http-modules-in-sharepoint.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/11/07/supportability-concerns-with-custom-http-modules-in-sharepoint.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

I am sure that I'm not the first one to tell you this, but you can't believe
everything you read on the Internet these days ;-)

Case in point...I've seen a number of sources claim that custom HTTP modules
are not supported in Microsoft Office SharePoint Server (MOSS) 2007 and Windows
SharePoint Services (WSS) v3.

These statements make it sound like if you add your own custom processing
to the pipeline of each page request, you will instantly teleport your Production
SharePoint farm into that vast wasteland we generally refer to as "Unsupported"
-- where your support calls to Microsoft will be summarily terminated shortly
after getting your Service Request (SRX) number.

Not so -- at least in this particular case.

Thank goodness, at least for my sake, because I've worked on a number of
MCS (Microsoft Consulting Services) projects where we've delivered custom HTTP
modules. It's always a little scary that something we deliver as part of an
MCS engagement will later be deemed "unsupported."

In general, we are very good about knowing what is -- and is not -- supported,
but there's always a chance that something "creative" we do in order to satisfy
the business requirements for our customers will later be considered a problem
by Microsoft Customer Service and Support (CSS).

There are, in fact, lots of things you can do with SharePoint that will thrust
you into the dreaded "unsupported" category, but a custom HTTP module isn't
one of them.

How can I say this with such conviction? Well, yesterday I came across the
following on MSDN:

> #### Support Details
> 
> A HTTP module assembly can be installed in the Web application's \bin
> directory or in the global assembly cache.
> 
> Because an HTTP module is always called as part of the page processing
> pipeline, a poorly designed or faulty module can have a detrimental effect
> on performance or perceived stability of the environment. Thoroughly test
> each module for performance before deploying it.

If you want to see it for yourself, you can go straight to the source:

<cite>HTTP Module</cite>
[http://msdn.microsoft.com/en-us/library/bb862635.aspx](http://msdn.microsoft.com/en-us/library/bb862635.aspx)

Obviously you want to be very cautious about using custom HTTP modules, but
there are definitely scenarios where this is a good approach.

